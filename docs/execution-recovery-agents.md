# Execution Recovery Agent

**Source:** https://project44.atlassian.net/wiki/x/CQEDzwg
**Last updated:** 2026-03-09

---

## Problem Statement

When a carrier cancels a confirmed freight booking close to the pickup time, the shipper is left without transportation. Today this requires manual intervention — the shipper must identify replacement carriers, call them for quotes, compare options, and create a new booking. This process is slow and error-prone, especially under time pressure.

The Rebooking Agent automates this entire flow. It detects the cancellation, finds available carriers, collects quotes in parallel, evaluates options considering cost and urgency, gets shipper confirmation, and books the replacement — all without human intervention.

---

## External Dependencies

- **Lunapath** — Third-party provider for emails and phone calls.
- **DFP** — Internal service that orchestrates asynchronous phone calls via Lunapath.
- **TMS** — Internal service that manages bookings.

---

## Workflow Steps

> All operations to be done as SHIPPER role since we are automating it on behalf of SHIPPER.

### Step 1: Cancellation Detection

Lunapath monitors a shared mailbox for cancellation emails. When one arrives, it sends a webhook to our service (via DFP) with the booking identifier, tenant information, and cancellation reason. This is the trigger that starts the entire flow.

**Attributes extracted from the email:**
- `shipment_identifiers`
- Shipper tenant details (inferred from email sender/extension)
- `action`
- `reason`

**Webhook endpoint:**
```
POST /rebooking/handle-booking-cancellation
Content-Type: application/json
x-tenant-id: <tenant_id>
x-tenant-uuid: <tenant_uuid>
x-user-id: <user_id>

{
  "shipment_identifiers": ["<identifier>"],
  "action": "DELAY/CANCEL",
  "role": "SHIPPER",
  "source": "EMAIL",
  "reason": "No driver available for pickup window"
}
```

**Bail out conditions:**
- Email doesn't provide the required attributes.

---

### Step 2: Fetch Original Booking

The agent calls TMS to fetch the full details of the booking. This gives us everything needed for the rebooking: the lane (origin and destination), equipment type required, scheduled pickup time, the carrier that cancelled, and the shipper's contact information.

**Fields used downstream:**
- For cancellation: `id`
- For prepare_rebooking logic: `routeSegments`, `capacityProviderIdentifier`, `totalRate`, `rateIdentifiers`, `transportationMode`
- For rebooking creation: `shipmentId`, `routeSegmentIds`, `shipperContactInfo`, `shippingDetails`

**Bail out conditions:**
- API fails post retries
- No booking found
- More than 1 booking found

---

### Step 3: Mark Booking as Cancelled

The agent calls TMS to mark the booking as cancelled so the system reflects reality. This also sends a webhook to the customer.

**Bail out conditions:**
- Status not 200 (OK) or 400 (already in CANCELLED/REJECTED/EXPIRED state)

---

### Step 4: Prepare for Rebooking

Three things happen in this step:

1. **Load surcharge policy** — The agent loads the tenant's rebooking policy with surcharge tiers based on urgency (e.g. +10% if >24h to pickup, +25% if <12h, +50% if <6h). Default: 5% always.

2. **Identify candidate carriers** — Pulls contract carriers for the lane (excluding the cancelled one) via dynamic route orchestrator, plus spot carriers for the lane.

3. **Retrieve contacts** — Uses lifecycle metadata service to get phone numbers for each candidate carrier (picks the most recently added contact).

**Rate logic:**
- Contract rate available → Baseline: contract rate, Max: contract + surcharge. If carrier asks more than contract, mark new booking as spot.
- No contract rate → Baseline: lowest of (market, spot), Max: original booking rate + surcharge. New booking is always spot.

**Bail out conditions:**
- No other carrier is applicable

---

### Step 5: Dispatch Carrier Quote Requests

The agent fires a DFP request to make a phone call (via Lunapath) to every candidate carrier back-to-back. Since DFP processes calls asynchronously, all calls happen in parallel. Each request includes a webhook URL and a deterministic thread ID so results can be routed back to the correct rebooking run.

After dispatching all calls, the agent records a quote deadline (current time + 1 hour) and pauses. Graph state is checkpointed to PostgreSQL. The background thread is freed — no resources are held while waiting.

**Call prompt essence:**
> "We have a rebooking request for a {equipment_type} shipment from {origin_city}, {origin_state} to {destination_city}, {destination_state}, pickup needed by {pickup_time}. Can you provide a rate quote and confirm availability for this lane?"

---

### Step 6: Collect Carrier Quotes (10-minute / 1-hour window)

While the graph is paused, DFP completes phone calls and sends individual results to a carrier-quote webhook. Each webhook stores the quote and checks whether it's time to resume.

**Resume conditions (either):**
- All carriers have responded, OR
- Quote deadline has passed

**Two-tier deadline:**
- Zero quotes received → deadline is 1 hour from call dispatch
- At least one quote received → deadline tightens to 10 minutes from call dispatch

A per-minute cron job (APScheduler) runs as a safety net — queries for rebooking runs in `COLLECTING_QUOTES` status past their deadline, and resumes them with whatever quotes are available.

**Bail out conditions:**
- No quote within 1 hour

---

### Step 7: Evaluate Quotes

Combines deterministic scoring with LLM validation.

1. **Weighted score per carrier:**
   - Cost: 40%
   - Reliability: 30%
   - Tracking: 20%
   - Availability: 10%
   - (Contracted carriers get additional weight)

2. **LLM review** — Pre-ranked list + urgency context + budget ceiling sent to LLM. LLM can suggest reordering only with explicit justification. If LLM is unavailable, deterministic ranking is used as-is.

**Output:** Top-ranked carrier = preferred option. Next 1–2 = alternatives to present to shipper.

---

### Step 8: Get Shipper Confirmation

The agent initiates a DFP phone call to the shipper presenting the preferred carrier (rate + reasoning) and alternatives. Graph pauses again waiting for shipper's response via a shipper-choice webhook.

**Call prompt essence:**
> "Hi, this is an automated call regarding your {equipment_type} shipment from {origin_city} to {destination_city}, pickup by {pickup_time}. We have carrier options available for your review. Our recommended option is {preferred_carrier_name} at ${preferred_rate} — {preferred_reasoning}. We also have {alt1_carrier_name} at ${alt1_rate} and {alt2_carrier_name} at ${alt2_rate} as alternatives. Which carrier would you like to proceed with?"

**Bail out conditions:**
- No response from shipper or they do not approve.

---

### Step 9: Create Replacement Booking and Confirm

The agent calls TMS to create a new booking with the carrier the shipper selected, at the quoted rate, for the original lane and new pickup time. The rebooking is complete.

**Bail out conditions:**
- Creating new booking fails

---

## Status (as of 2026-03-09)

A few connector and code issues were found during E2E testing in production, primarily due to inability to test DFP side callback flow and differences in carrier information between QA and Prod.

**Current state:**
- Issues found in tms-agents during E2E testing are fixed in QA. Deploying to prod on 16th March for a final round of E2E testing.
- New APIs created in tms-agents:
  - Fetch existing booking
  - Cancel existing booking
  - Get other carrier options
  - Make DFP → Lunapath → carrier calls to get quotations sequentially
  - Create new booking from the first accepted carrier

**Connectors built:**
- `ANY.SHIPMENT_STATUS.CARRIER_PUSH.REA_LUNAPATH_TRIGGER` — Emails to booking.recovery@project44.com trigger the flow via tms-agents API
- `ANY.SHIPMENT_STATUS.CARRIER_PULL.REA_LUNAPTHOUT` — Sends request to Lunapath to make carrier calls (tested)
- `ANY.SHIPMENT_STATUS.CARRIER_PUSH.REA_LUNAPATH_WEBHOOK` — Handles carrier response callback to tms-agents (tested)

**Lunapath prompts:**
- Email: Shipper Email Classifier
- Call: Carrier Quotation Rebook_v1 (tested over multiple cases including non-acceptance and price negotiation)
