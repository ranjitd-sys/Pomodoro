# Tracking API Research — Issues & Findings

**Date:** 26 June 2026  
**Goal:** Bulk tracking status from CSV for Bluedart & Delhivery shipments

---

## 1. Ship24 (`api.ship24.com`)

### What we tried
- Found the internal API endpoint ship24's own website uses:
  `POST https://api.ship24.com/api/parcels/{trackingNumber}?lang=en`
- Attempted to replicate with browser-like headers (`User-Agent`, `Referer`, `Origin`)
- Payload was device/browser fingerprint JSON:
  ```json
  {
    "userAgent": "...",
    "os": "Linux",
    "browser": "Chrome",
    "browser_version": "149.0.0.0",
    "device": "Unknown",
    "deviceType": "desktop",
    "orientation": "landscape",
    "os_version": "unknown",
    "uL": "en-GB"
  }
  ```

### Issue
**403 Forbidden** — Ship24 enforces `cross-origin-resource-policy: same-origin`.  
Requests must originate from a browser session on `https://www.ship24.com`. Scripts are blocked.

### Verdict
❌ Dead end. Cannot be used from a server-side script.

---

## 2. Bluedart (`bluedart.com`)

### What we tried
- Looked for Bluedart's internal API gateway:
  `POST https://apigateway.bluedart.com/in/transportation/track/v1/shipments`
- Attempted to find a no-auth public endpoint

### Issue 1 — API requires credentials
Bluedart's official PackTrack API always requires:
- `loginid` — your Bluedart account login
- `lickey` — license key issued by Bluedart

No way around this without a Bluedart corporate account.

### Issue 2 — Website has CAPTCHA
Bluedart's tracking page (`https://www.bluedart.com`) has a CAPTCHA on the form — making web scraping impractical for bulk use.

### Verdict
❌ Cannot scrape. Needs Bluedart PackTrack credentials (free for existing customers — contact Bluedart account manager).

---

## 3. Delhivery (`track.delhivery.com`)

### What we found
Delhivery exposes a public tracking endpoint used by their website:
```
GET https://track.delhivery.com/api/v1/packages/json/?waybill={trackingNumber}&token=
```

- `token=` can be left **blank** for public tracking
- Supports **up to 30 AWB numbers** per request (comma-separated)
- Returns JSON with full shipment scan history

### Script built
- Reads `input.csv` with `tracking_number` column
- Batches into groups of 30
- 2 concurrent requests, 1s delay between batches
- Writes `output.csv` with: `status`, `last_event`, `last_location`, `expected_delivery`

### Verdict
✅ Works. No auth, no signup, free, supports bulk.

---

## Summary

| Courier | Approach | Result | Reason |
|---|---|---|---|
| Ship24 | Internal API (POST) | ❌ 403 | `cross-origin-resource-policy: same-origin` |
| Bluedart | API Gateway | ❌ Blocked | Requires `loginid` + `lickey` |
| Bluedart | Website scrape | ❌ Blocked | CAPTCHA on tracking form |
| Delhivery | Public endpoint | ✅ Works | No auth, batch of 30 |

---

## Recommendations

- **Delhivery** — use the script built today, no further setup needed.
- **Bluedart** — contact Bluedart account manager and request PackTrack API credentials (`loginid` + `lickey`). Free for existing customers.
- **Alternative for Bluedart** — sign up for AfterShip or TrackingMore if volume is under their free tier limits.