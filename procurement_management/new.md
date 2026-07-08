---
title: Spatial Runtime Engineering Handoff
version: July 2026
status: Foundation Approved
audience: Engineering Team
owner: Platform Engineering
---

# Spatial Runtime Engineering Handoff

## Executive Summary

Over the past several months, Guided Spaces has evolved from an animated navigation framework into the foundation of a reusable platform.

Originally our objective was to build page transitions and camera-based navigation.

Through multiple architectural iterations we discovered that the underlying problem is much larger.

We are not building navigation.

We are building a **Spatial Runtime**.

The Spatial Runtime will become the common foundation for every DeepEcom application including:

Procurement Automation
Sampatti Automation
Order Processing
DeepTrack
Future business applications

Applications should no longer implement navigation, motion, workflow orchestration or architectural policies independently.

Those become responsibilities of the runtime.

---

# Vision

Spatial Runtime is a Capability-Oriented Spatial Operating System whose primary UI primitive is the Actor.

Navigation, motion, workflow, layout, contracts and reusable capabilities become platform concerns instead of application concerns.

Applications contribute business logic.

The runtime provides the operating system.

---

# Why We Changed Direction

Initially our architecture looked like this.
Pages
    ↓
Navigation
    ↓
Transitions

This worked for individual applications.

It does not scale across multiple products.

During the architecture review we recognized that every application was solving the same problems repeatedly:

navigation
URL handling
camera movement
motion
layout
routing
reduced motion
accessibility
workflows

Instead of solving these separately, we moved them into a shared runtime.

---

# Current Architecture

Today the platform is organized around the following concepts.
Spatial Runtime

    ↓

World

    ↓

Capabilities

    ↓

Actors

    ↓

Spaces

    ↓

Places

    ↓

Panels

This hierarchy will guide future development.

---

# Core Runtime Concepts

## World

The World is the infinite coordinate system.

It owns:

layout
overview
spatial memory
world coordinates

World coordinates are immutable.

---

## Camera

The Camera is machine-owned.

Only the runtime controls movement between Places.

Views never manipulate camera state.

---

## Projection

Projection converts

World

↓

Screen

Projection never owns navigation or motion.

---

## Spaces

Spaces represent logical application regions.

Examples:

Procurement
Sampatti
DeepTrack
Order Processing

---

## Actors

Actors are the primary UI primitive.

An Actor owns:

behavior
commands
summary
health
motion intent
contracts

Applications become collections of Actors.

---

## Capabilities

Capabilities provide reusable runtime behavior.

Examples:

Navigation
Motion
Workflow
Reporting
OCR
AI
Timeline

Applications consume capabilities instead of implementing them independently.

---

# Motion Philosophy

Motion exists to communicate meaning.

Motion is never decorative.

Spatial Runtime distinguishes two motion tiers.

## Tier 1 — World Travel

Purpose:

Travel between Places.

Examples:

Overview

↓

Procurement

↓

DeepTrack

Behavior:

Camera moves
World remains fixed
Projection updates

---

## Tier 2 — In-space Focus

Purpose:

Explore inside one Place.

Examples:

Issue

↓

Issue Detail

↓

Timeline

Behavior:

Camera remains fixed
Actors move
Focus changes
Elastic interactions

---

# Documentation Architecture

The runtime now has a structured documentation hierarchy.

Every engineer should follow this order.
North Star

↓

Canon

↓

Book

↓

Vocabulary

↓

Decision Tree

↓

Cookbook

↓

Architecture Readiness

↓

Constitution

↓

Contracts

↓

Implementation

---

# Documentation Purpose

## North Star

Defines what we are building.

---

## Canon

Explains why architectural decisions exist.

---

## Book

Introduces the runtime mental model.

---

## Vocabulary

Defines official terminology.

Examples:

World
Actor
Capability
Place
Intent
Projection

---

## Decision Tree

Explains where new functionality belongs.

---

## Cookbook

Provides implementation recipes.

---

## Constitution

Defines architectural laws.

---

## Contracts

Machine-readable architecture guarantees.

---

# Spatial Runtime Contract System (SRCS)

We are introducing SRCS to ensure architectural decisions remain enforceable.

Instead of relying on code reviews alone, contracts automatically verify runtime guarantees.

Examples include:

Single URL writer
Camera ownership
Motion ownership
Reduced motion behavior
Layout immutability
Navigation policy

Future versions will extend contracts to:

Actors
Plugins
Capabilities
Attention Runtime

---

# Engineering Governance

Every implementation must be traceable.
North Star

↓

Vocabulary

↓

Canon

↓

ADR / RFC

↓

Contract

↓

Capability

↓

Implementation

↓

Tests

↓

Evidence

Architecture should never exist only in developer memory.

---

# Runtime Maturity

## Stable

World
Camera
Navigation
Projection
Routing

## Evolving

Motion Runtime

## Experimental

Actor Runtime

## Research

Attention Runtime
Capability Runtime
Plugin Runtime
Spatial Inspector
Timeline / Replay

---

# Roadmap

## Phase 0

Architecture readiness.

Documentation.

Vocabulary.

Registries.

---

## Phase 1

Spatial Runtime Contract System.

Guided Spaces v1.5.

Contract enforcement.

---

## Phase 2

Motion Runtime.

Tier 2 motion.

Motion Tokens.

Playwright validation.

---

## Phase 3

Actor Runtime.

Actor surfaces.

Summaries.

Commands.

---

## Phase 4

Spatial Inspector.

Timeline.

Replay.

---

## Phase 5

Plugin certification.

---

# Engineering Principles

## 1.

The runtime owns platform behavior.

Applications own business behavior.

---

## 2.

Motion communicates intent.

Never animate for decoration.

---

## 3.

Camera moves between Places.

Actors move within Places.

---

## 4.

World coordinates never change.

---

## 5.

Capabilities are reusable.

Avoid product-specific implementations when platform capabilities exist.

---

## 6.

Every architectural decision must be traceable.

---

## 7.

Documentation is part of implementation.

A feature is incomplete if its architecture is undocumented.

---

# Current Deliverables

Completed:

✅ Spatial Runtime vision

✅ Documentation hierarchy

✅ North Star

✅ Canon

✅ Vocabulary

✅ Runtime Book

✅ Decision Tree

✅ Cookbook

✅ Architecture Readiness

✅ Motion Manifesto

✅ Actor Manifesto

✅ Spatial Runtime Contract System design

✅ Capability Registry

✅ Research Registry

✅ Phase roadmap

Next:

➡ Implement Phase 1

SRCS
Guided Spaces v1.5
Motion foundations
Contract automation
Evidence reporting

---

# Expectations for Contributors

Before implementing any feature:

1. Read the North Star.
2. Verify terminology in the Vocabulary.
3. Use the Decision Tree to identify the correct runtime layer.
4. Update Contracts if architectural behavior changes.
5. Add tests and evidence.
6. Preserve runtime principles.

---

# Long-Term Goal

Six months from now:

Every application runs on the same Spatial Runtime.
Motion feels consistent across products.
Architectural rules are enforced automatically.
New engineers can onboard by reading the handbook.
New products are assembled from reusable capabilities rather than duplicating infrastructure.

Spatial Runtime becomes the operating system for all DeepEcom business applications.

---

# Team Message

We are no longer building isolated applications.

We are building the platform those applications will run on.

Every pull request should strengthen the Spatial Runtime first and the application second.

When making architectural decisions ask one question:

 "Does this make the Spatial Runtime simpler, more reusable, more understandable, and more durable for every future product?"