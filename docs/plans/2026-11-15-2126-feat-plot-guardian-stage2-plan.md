---
title: Plot Guardian Stage 2 Eyes and Local AI - Plan
type: feat
date: 2026-11-15
topic: plot-guardian-stage2
artifact_contract: ce-unified-plan/v1
product_contract_source: ce-brainstorm
execution: code
---

# Plot Guardian Stage 2 Eyes and Local AI - Plan

## Goal Capsule

- **Objective:** A city owner of a countryside vegetable plot can rely on short text reports about beds and the gate without driving out, including when the public internet is down.
- **Means:** An on-plot hub runs local vision, sends a daily coded digest, rare heartbeats, and person-at-gate alerts, and answers text questions from the latest images over Stage 1's low-connectivity link.
- **Product authority:** This plan owns Plot Guardian Stage 2 (eyes and local AI) only. Stage 1 (water and link) and Stage 3 (robot gantry) are surrounding areas, not active scope.
- **Open blockers:** None. Remaining items are deferred to planning.

---

## Product Contract

### Summary

Plot Guardian Stage 2 is an on-plot hub that watches beds and the gate with local vision and texts the city owner a short daily coded digest, rare "still alive" heartbeats, and person-at-gate alerts. The owner can text a question and get an answer drawn from the latest on-plot images. Messages ride Stage 1's low-connectivity channels so they still work when the public internet fails, including war or calamity outages.

### Problem Frame

Romanian owners of vegetable plots 50–100 km from the city already drive out, pay someone on site, or install cameras. Those options still fail together: false alarms, no signal to review a live feed from the city, and no time to watch video. In war or calamity, public connectivity is worse, so a product that depends on pulling images to the phone is the wrong bet. The owner needs short, trustworthy text that arrives when the network is thin, without treating a silent dead hub as "all clear."

### Key Decisions

- **Stage 2 only.** (session-settled: user-directed — chosen over Stage 1, Stage 3, or Stages 1+2: each stage is independently useful; this run owns eyes and local AI.) Surrounding stages stay context, not requirements. Governs product authority in Goal Capsule; Scope Boundaries.
- **Guardian digest plus rare heartbeat.** (session-settled: user-directed — chosen over Quiet sentinel or Twin audience: matches daily summary + gate alert + Q&A success bar while avoiding silence-as-OK.) Governs R3, R4, R5.
- **Proactive reports and text Q&A in the first ship.** (session-settled: user-directed — chosen over reports-only: first useful Stage 2 includes answering questions about what the hub saw.) Governs R6.
- **Soft image rule.** (session-settled: user-directed — chosen over strict on-plot-only or split plant/security camera rules: prefer local capture; vendor/cloud cameras allowed when the internet works, knowing some frames may already sit with the vendor.) Governs R7, R8.
- **Missed heartbeat means trouble.** Confirmed in synthesis: absence of an expected heartbeat is a problem signal, not "everything is fine." Governs R5, AE4.
- **Beds and gate in the first ship.** Confirmed in synthesis: horticulture status and person-at-gate events both ship together, not phased. Governs R2, R3, R4.

<!-- ce-section: work-relationships -->
### How This Work Fits Together

This plan covers **Stage 2 — Eyes and local AI** only. The broader Plot Guardian breakdown below is the current understanding, not a committed roadmap.

- **Stage 1 — Water and link**
  - **Enables** Stage 2: compact multi-channel messaging (mobile SMS/low-power data, LoRa relays, later amateur shortwave) and the physical plot context.
  - **Can proceed independently of** Stage 2.
- **Stage 2 — Eyes and local AI** (this plan)
  - **Depends on** Stage 1's link for outbound reports and inbound questions.
  - **Can proceed independently of** Stage 3.
- **Stage 3 — Robot gantry** (optional)
  - **Can proceed independently of** Stage 2 for mechanical work; imaging may later **Share** on-plot vision with Stage 2.
  - Still to decide in a later plan: how much Stage 2 report vocabulary Stage 3 reuses.

### Actors

- A1. **Owner** — city-based owner of a countryside vegetable plot who receives texts and may send questions.
- A2. **On-plot hub** — Stage 2 computer that wakes on a schedule or on motion, runs local vision, and exchanges short messages via the Stage 1 link.
- A3. **Cameras** — local-network cameras always usable when powered; vendor/cloud cameras usable when the public internet reaches the plot.
- A4. **Stage 1 link** — multi-channel messaging path assumed available; not built in this plan.

### Key Flows

```mermaid
flowchart TB
  wake[Hub wakes on schedule or motion] --> capture[Capture from available cameras]
  capture --> infer[Local vision writes coded findings]
  infer --> decide{What to send?}
  decide -->|daily window| digest[Daily coded digest]
  decide -->|person at gate| alert[Person-at-gate alert]
  decide -->|heartbeat due| beat[Rare alive heartbeat]
  digest --> link[Stage 1 link to owner]
  alert --> link
  beat --> link
  ownerQ[Owner texts a question] --> hubQ[Hub answers from latest images]
  hubQ --> link
  miss[Expected heartbeat missing] --> ownerWorry[Owner treats hub as possibly dead]
```

- F1. Daily coded digest
  - **Trigger:** Scheduled wake in the daily reporting window.
  - **Actors:** A2, A3, A4, A1
  - **Steps:** Hub captures; local vision encodes bed and notable findings; hub sends one short coded digest over Stage 1.
  - **Outcome:** Owner receives a compact plot status text without opening a video feed.
  - **Covered by:** R2, R3, R9

- F2. Person-at-gate alert
  - **Trigger:** Motion or vision detects a person at the gate (or equivalent watched entry).
  - **Actors:** A2, A3, A4, A1
  - **Steps:** Hub wakes if needed; confirms person event; sends a short alert with time.
  - **Outcome:** Owner learns of the event without live video.
  - **Covered by:** R4, R9

- F3. Rare heartbeat
  - **Trigger:** Heartbeat interval elapses with hub healthy.
  - **Actors:** A2, A4, A1
  - **Steps:** Hub sends a short alive / last-seen line. If the owner does not receive an expected heartbeat, they treat the hub as possibly dead.
  - **Outcome:** Silence is not mistaken for safety.
  - **Covered by:** R5, AE4

- F4. Owner Q&A
  - **Trigger:** Owner texts a question about the plot.
  - **Actors:** A1, A4, A2, A3
  - **Steps:** Stage 1 delivers the question; hub answers from the latest relevant on-plot images; answer returns as short text. Images are not sent to the owner.
  - **Outcome:** Owner can ask what the hub saw without pulling a feed.
  - **Covered by:** R6, R8, R9

### Requirements

**Reporting and awareness**

- R1. The Stage 2 hub operates on the plot and produces owner-facing value without a cloud vision service as its primary path.
- R2. Local vision covers vegetable beds (condition such as wilting and weeds) and at least one watched entry (gate / person).
- R3. The hub sends the owner a short daily coded digest of bed status and notable findings.
- R4. The hub sends a short person-at-gate (or equivalent entry) alert when that event is detected, including time.
- R5. The hub sends rare alive heartbeats on a longer interval than the daily digest; missing an expected heartbeat is a problem signal for the owner, not an all-clear.
- R6. The owner can text a question and receive a short-text answer derived from the latest on-plot images.

**Cameras, privacy, and link**

- R7. When the public internet is down, the hub continues vision from cameras reachable on the plot's local network.
- R8. When the public internet works, the hub may also use vendor/cloud cameras; local capture remains preferred. "Images stay on the plot" is the preference for hub-managed capture, not a guarantee for vendor cameras that already upload off-plot.
- R9. All owner-facing Stage 2 messages use Stage 1's compact multi-channel link so short texts still leave the plot when internet and high bandwidth are unavailable.
- R10. Owner-facing answers and reports do not transmit plot images to the city; only coded or short natural-language text leaves via the link.

**Resilience positioning**

- R11. Stage 2 remains useful under public-network failure (including war or calamity conditions), not only under ordinary weak rural internet.

### Acceptance Examples

- AE1. Daily digest
  - **Covers:** R3, R9
  - **Given:** Hub and Stage 1 link are up; morning report window arrives.
  - **When:** The hub completes its scheduled vision pass.
  - **Then:** The owner receives one short coded text summarizing beds and notable findings, with no image attachment.

- AE2. Person at gate
  - **Covers:** R4
  - **Given:** A person appears at the watched gate.
  - **When:** The hub detects the event.
  - **Then:** The owner receives a short alert that names the gate event and time.

- AE3. Q&A without images
  - **Covers:** R6, R10
  - **Given:** Recent images exist on the hub.
  - **When:** The owner texts a question such as whether bed 2 looks wilted.
  - **Then:** The owner receives a short-text answer from those images; no image is sent to the city.

- AE4. Missed heartbeat
  - **Covers:** R5
  - **Given:** A heartbeat was expected and none arrives within the grace the product defines.
  - **When:** The owner notices the gap (or an absence signal the product exposes).
  - **Then:** The situation is treated as hub-possibly-dead, not as an all-clear plot.

- AE5. Internet outage, local cameras remain
  - **Covers:** R7, R9, R11
  - **Given:** Public internet to the plot is down; local-network cameras still work; Stage 1 link still delivers short messages.
  - **When:** The daily window or a gate event occurs.
  - **Then:** The owner still receives the corresponding short text sourced from local cameras.

- AE6. Vendor camera only while net is up
  - **Covers:** R7, R8
  - **Given:** A vendor/cloud camera is configured and the public internet later drops.
  - **When:** The hub needs images during the outage.
  - **Then:** The hub does not depend on that vendor camera; it uses local-network cameras for Stage 2 duties.

### Success Criteria

- After a week of use, the owner can skip a routine drive because daily digests, person-at-gate alerts, and Q&A about what the hub saw are trustworthy enough in short text.
- During a public-network outage that still allows Stage 1 messaging, the owner continues to receive digests/alerts/Q&A from local cameras.
- A missed heartbeat is understood as risk to the hub, not reassurance about the plot.

### Scope Boundaries

**Deferred for later**

- Neighbor or caretaker as a second recipient of security alerts (Twin audience).
- Quiet event-only mode without a daily digest.
- Strict ban on any vendor/cloud camera.
- Live video to the owner's phone as the primary experience.
- Stage 3 robot gantry seeding, weeding, and gantry imaging product work.

**Outside this product's identity**

- Replacing Stage 1 water hardware or inventing the multi-channel link inside Stage 2.
- Becoming a general neighborhood security network.
- Depending on cloud vision or FarmBot cloud image upload as the primary Stage 2 path (this repo's FarmBot web app stores webcam URLs and GCS-backed device photos for a different product model).

### Dependencies / Assumptions

- Stage 1 link exists or will exist soon enough to carry compact inbound and outbound messages (SMS / low-power data, LoRa relay chain, later amateur shortwave as channels evolve).
- The plot has power for the hub and at least one local-network camera when internet is down.
- Coded report vocabulary can be compact enough for Stage 1 message size limits.
- Verified contrast in this repository: no Plot Guardian product exists today; FarmBot webcam feeds are external URLs rendered in the browser; device photos use GCS; weed detection / `take_photo` are device camera ops; cloud OpenAI here assists Lua/sequences, not on-plot offline vision reporting.

### Outstanding Questions

**Resolve Before Planning**

- None.

**Deferred to Planning**

- Exact coded digest field set and encoding (beyond examples like wilting / weeds / person time).
- Heartbeat interval and grace period before "missed."
- How vendor/cloud camera adapters are selected and authenticated when the net is up.
- Hub wake policy details (motion vs few-times-daily power budget).
- Whether any FarmBot on-device camera/plant-detection pieces are reused as optional Stage 3 adjacency later, without pulling Stage 2 onto the FarmBot cloud path.

### Sources / Research

- Dialogue settled Stage 2 scope, Guardian digest + rare heartbeat, soft image rule, and calamity-oriented durability.
- Repo grounding dossier (scratch): confirms FarmBot cloud webcam / GCS / device photo / OpenAI-Lua patterns as contrast, not Stage 2 precursors — see claim verification against `app/models/webcam_feed.rb`, `frontend/controls/webcam/webcam_img.tsx`, `app/models/image.rb`, `frontend/photos/actions.ts`, `frontend/devices/actions.ts`, `app/controllers/api/ais_controller.rb`.
