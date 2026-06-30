# Issue Tracker State: Atom vs useState

## What we're trying to do

Replace six `useState` calls (`backlog`, `todo`, `inProgress`, `inReview`, `completed`, `canceled`) plus a manual `distribute()` filter function and an `allIssuesRef` hack with a reactive state system that:

1. Reads `IssueEntry[]` from IndexedDB via `readAllIssues` (an Effect)
2. Re-reads automatically when the underlying `EventJournal` changes (local writes, remote sync, cross-tab broadcast)
3. Derives the six status buckets without manually re-filtering and re-setting six pieces of state every time

The `effect/unstable/reactivity/Atom` system (used via `@effect/atom-react`'s `useAtomValue`) looked like the right tool: computed atoms that auto-derive from a base atom, similar to Jotai/Recoil selectors.

## Why it's not working

Three separate problems stacked on top of each other:

**1. Registry mismatch.** Atoms are only reactive *within the registry instance that mounted them*. Early attempts created `const registry = AtomRegistry.make()` at module scope — a different instance than the one `RegistryProvider` creates and that `useAtomValue` actually subscribes to via `RegistryContext`. Writes to the wrong registry are invisible to components.

**2. Stale closures in the journal fiber.** The `EventJournal` subscription is a long-running `Effect.runFork` fiber started once (singleton guard: `if (journalFiber) return`). It captures whatever `invalidate` closure existed at the moment it started. Even after fixing the registry, the fiber doesn't pick up a fresh `invalidate` on re-render.

**3. The core mismatch — Effect atoms don't auto-rerun.** This is the actual blocker. Per the official `atom-react` test suite (`atom-react/test/*.test.tsx`):

   - `Atom.make(plainValue)` + `registry.set(atom, newValue)` → reactive, confirmed by tests
   - `Atom.make((get) => get(other) * 2)` (sync computed) → reactive, confirmed by tests
   - `Atom.make(someEffect)` → runs the Effect **once**, settles into an `AsyncResult`, and **does not re-run** just because a "tick" dependency you `get()` inside it changes

   Our `issuesAtom` was `Atom.make((get) => { get(tick); return readAllIssues.pipe(...) })` — a computed function returning an Effect. Bumping `tick` does cause the computed function to re-evaluate, but the registry's caching behavior for Effect-returning atoms means it doesn't necessarily restart the `readAllIssues` run. There's no test in the suite confirming this pattern works, and in practice it didn't.

   The pattern that *is* proven to work for streaming async data is `Atom.runtime(layer).atom(someStream, { initialValue })` — wrapping the journal subscription itself as a `Stream` and letting the atom runtime own the fiber. This is architecturally different from "poll a tick and re-fetch," and is a bigger rewrite than we'd scoped.

## Can we use useState instead?

Yes — and given the above, it's the more pragmatic choice. The original code you had (single `distribute()` function + six `useState` calls + `allIssuesRef`) already works correctly. It is not broken; it's just more boilerplate than the atom version promised.

**Recommendation:** revert to `useState`, but trim the boilerplate two ways without touching the reactivity model:

1. **Replace `allIssuesRef` pattern** — instead of a manually-synced ref, store all issues in one `useState<IssueEntry[]>` and derive the six buckets with `useMemo` on every render instead of six separate `setX` calls:

```ts
const [issues, setIssues] = useState<IssueEntry[]>([]);

const backlog    = useMemo(() => issues.filter(i => i.status === "backlog"), [issues]);
const todo       = useMemo(() => issues.filter(i => i.status === "todo"), [issues]);
const inProgress = useMemo(() => issues.filter(i => i.status === "in-progress"), [issues]);
const inReview   = useMemo(() => issues.filter(i => i.status === "in-review"), [issues]);
const completed  = useMemo(() => issues.filter(i => i.status === "completed"), [issues]);
const canceled   = useMemo(() => issues.filter(i => i.status === "canceled"), [issues]);
```

This drops `distribute()` entirely — `setIssues(newArray)` is the only state write, and the six buckets recompute automatically via `useMemo`. `allIssuesRef` also disappears since `issues` is already the live array.

2. **Keep the journal fiber, BroadcastChannel, and polling logic exactly as in your original code** — that part was never broken, it just needs `load()` to call `setIssues(...)` instead of `distribute(...)`.

This gets you 90% of what the atom rewrite promised (no manual array-filter-six-times boilerplate) with zero new dependencies, zero registry/closure pitfalls, and a pattern that's been working in your codebase already.

## If you still want atoms later

Worth revisiting once `@effect/atom-react` is out of beta, using the `Atom.runtime(layer).atom(Stream)` pattern shown by the working `todosAtom` example (different project), not the tick-based computed-Effect-atom approach we tried here.