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
- **Means:** New top-level `plot_guardian/` hub package with Frigate-backed gate vision, pluggable plant vision, RTSP-first cameras, and SMS LinkPort (KTD1–KTD4).
- **Product authority:** This plan owns Plot Guardian Stage 2 (eyes and local AI) only. Stage 1 (water and link) and Stage 3 (robot gantry) are surrounding areas, not active scope.
- **Product Contract preservation:** Product Contract unchanged.
- **Open blockers:** None.
- **Execution profile:** Implement and verify under `plot_guardian/` with pytest; do not modify FarmBot `app/` or `frontend/` for Stage 2.
- **Who finishes:** Implementer ships `plot_guardian/` runnable on a Raspberry Pi–class hub with mocked integration tests green.

---

## Product Contract

### Summary

Plot Guardian Stage 2 is an on-plot hub that watches beds and the gate with local vision and texts the enrolled city owner a short daily coded digest, rare "still alive" heartbeats, and person-at-gate alerts. The owner can text a question and get an answer drawn from the latest on-plot images, with capture age disclosed. Messages ride Stage 1's low-connectivity channels so they still work when the public internet fails, including war or calamity outages. At least one local-network camera is required so vision survives those outages.

### Problem Frame

Romanian owners of vegetable plots 50–100 km from the city already drive out, pay someone on site, or install cameras. Those options still fail together: false alarms, no signal to review a live feed from the city, and no time to watch video. In war or calamity, public connectivity is worse, so a product that depends on pulling images to the phone is the wrong bet. The owner needs short, trustworthy text that arrives when the network is thin, without treating a silent monitoring path as "all clear."

### Key Decisions

- **Stage 2 only.** (session-settled: user-directed — chosen over Stage 1, Stage 3, or Stages 1+2: each stage is independently useful; this run owns eyes and local AI.) Surrounding stages stay context, not requirements. Governs product authority in Goal Capsule; Scope Boundaries.
- **Guardian digest plus rare heartbeat.** (session-settled: user-directed — chosen over Quiet sentinel or Twin audience: matches daily summary + gate alert + Q&A success bar while avoiding silence-as-OK.) Governs R3, R4, R5.
- **Proactive reports and text Q&A in the first ship.** (session-settled: user-directed — chosen over reports-only: first useful Stage 2 includes answering questions about what the hub saw.) Governs R6.
- **Soft image rule.** (session-settled: user-directed — chosen over strict on-plot-only or split plant/security camera rules: prefer local capture; vendor/cloud cameras allowed when the internet works, knowing some frames may already sit with the vendor.) Governs R9, R10. Local-network camera required for outage duty; Stage 2 must not upload hub-managed local frames off-plot.
- **Missed heartbeat means monitoring trouble.** Confirmed in synthesis and refined in review: absence of an expected heartbeat means the monitoring path is unreliable (hub or Stage 1 link) and plot status is unknown — not all-clear and not automatic plot emergency. Governs R5, AE4.
- **Beds and gate in the first ship.** Confirmed in synthesis: horticulture status and person-at-gate events both ship together, not phased. Governs R2, R3, R4.
- **Trustworthy short text over raw detections.** Review: person-at-gate uses a confirmation policy; digests and Q&A mark uncertain claims; expected-visitor quieting avoids caretaker cry-wolf. Governs R4, R7, R8.

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

- A1. **Owner** — city-based owner of a countryside vegetable plot, enrolled as the sole Stage 2 message party, who receives texts and may send questions.
- A2. **On-plot hub** — Stage 2 computer that wakes on a schedule or on motion, runs local vision, and exchanges short messages via the Stage 1 link.
- A3. **Cameras** — at least one local-network camera required for outage vision; vendor/cloud cameras optional when the public internet reaches the plot.
- A4. **Stage 1 link** — multi-channel messaging path assumed available; not built in this plan.

### Key Flows

```mermaid
flowchart TB
  wake[Hub wakes on schedule or motion] --> capture[Capture from available cameras]
  capture --> infer[Local vision writes coded findings]
  infer --> decide{What to send?}
  decide -->|daily window| digest[Daily coded digest]
  decide -->|person confirmed| alert[Person-at-gate alert]
  decide -->|heartbeat due| beat[Rare alive heartbeat]
  digest --> link[Stage 1 link to enrolled owner]
  alert --> link
  beat --> link
  ownerQ[Owner texts a question] --> hubQ[Hub answers with image age]
  hubQ --> link
  miss[Expected heartbeat missing] --> ownerWorry[Owner treats monitoring path as unknown]
```

- F1. Daily coded digest
  - **Trigger:** Scheduled wake in the daily reporting window.
  - **Actors:** A2, A3, A4, A1
  - **Steps:** Hub captures; local vision encodes bed and notable findings; claims the pipeline cannot support are marked uncertain; hub sends one short coded digest over Stage 1 to the enrolled owner.
  - **Outcome:** Owner receives a compact plot status text without opening a video feed.
  - **Covered by:** R2, R3, R7, R11, R13

- F2. Person-at-gate alert
  - **Trigger:** Motion or vision detects a person at the gate (or equivalent watched entry) outside any expected-visitor / quiet window.
  - **Actors:** A2, A3, A4, A1
  - **Steps:** Hub wakes if needed; applies the confirmation / confidence policy; if it passes and is not quieted, sends a short alert with time to the enrolled owner.
  - **Outcome:** Owner learns of unexpected entry events without live video, without cry-wolf paging for routine caretakers.
  - **Covered by:** R4, R8, R11, R13

- F3. Rare heartbeat
  - **Trigger:** Heartbeat interval elapses with hub healthy.
  - **Actors:** A2, A4, A1
  - **Steps:** Hub sends a short alive / last-seen line to the enrolled owner. If an expected heartbeat does not arrive, the owner treats plot status as unknown because the monitoring path failed — not as all-clear and not as automatic plot emergency.
  - **Outcome:** Silence is not mistaken for safety; link blips are not mistaken for plot crisis.
  - **Covered by:** R5, AE4

- F4. Owner Q&A
  - **Trigger:** Enrolled owner texts a question about the plot.
  - **Actors:** A1, A4, A2, A3
  - **Steps:** Stage 1 delivers a question attributable to enrollment; hub uses images within the stale threshold or captures fresh ones; answer includes capture age / last-seen time and marks unsupported claims uncertain; answer returns as short text. Images are not sent to the owner.
  - **Outcome:** Owner can ask what the hub saw without pulling a feed or trusting silent stale frames.
  - **Covered by:** R6, R7, R10, R11, R13

### Requirements

**Reporting and awareness**

- R1. The Stage 2 hub operates on the plot and produces owner-facing value without a cloud vision service as its primary path.
- R2. Local vision covers vegetable beds (condition such as wilting and weeds) and at least one watched entry (gate / person).
- R3. The hub sends the enrolled owner a short daily coded digest of bed status and notable findings.
- R4. The hub sends a short person-at-gate (or equivalent entry) alert when that event is detected and passes a product-defined confirmation / confidence policy that prevents recreating the false-alarm failure mode named in Problem Frame; the alert includes time. Exact thresholds are deferred to planning.
- R5. The hub sends rare alive heartbeats on a longer interval than the daily digest; missing an expected heartbeat means the monitoring path is unreliable (hub or Stage 1 link) and plot status is unknown — not an all-clear and not an automatic plot emergency. Owner-facing meaning must say that.
- R6. The enrolled owner can text a question and receive a short-text answer derived from on-plot images. Every answer includes the capture age or last-seen time of the images used. When those frames are older than a product-defined stale threshold, the hub captures fresh images before answering. Exact threshold deferred to planning.
- R7. Digests and Q&A only assert bed or gate claims the local vision pipeline can support; they emit uncertain / unknown when confidence is low.
- R8. The owner can define expected-visitor or quiet windows (or equivalent) so routine caretaker or neighbor presence is not paged as an unexpected person-at-gate. Exact UX deferred to planning. Twin audience (second recipient) stays deferred.

**Cameras, privacy, and link**

- R9. A Stage 2 install must include at least one plot-local-network camera that can serve vision when public internet is down. When the public internet is down, the hub continues vision from those local-network cameras.
- R10. When the public internet works, the hub may also use vendor/cloud cameras as optional additives; they are never a sufficient sole source for R9 / R12 duties. Local capture remains preferred. Stage 2 must not upload or forward hub-managed local-network camera frames off-plot. Any off-plot frames vendor cameras already hold are vendor behavior Stage 2 acknowledges, not a path Stage 2 may open for local capture.
- R11. All owner-facing Stage 2 messages use Stage 1's compact multi-channel link so short texts still leave the plot when internet and high bandwidth are unavailable.
- R12. Stage 2 remains useful under public-network failure (including war or calamity conditions), not only under ordinary weak rural internet — which requires the local-network camera in R9 and Stage 1 messaging in R11.
- R13. Stage 2 binds outbound digests, person-at-gate alerts, and heartbeats, and inbound Q&A, to the enrolled owner channel(s). The hub answers only questions attributable to that enrollment (via Stage 1 identity guarantees or an equivalent Stage 2 enrollment when Stage 1 is an opaque pipe).
- R14. Owner-facing answers and reports do not transmit plot images to the city; only coded or short natural-language text leaves via the link.

### Acceptance Examples

- AE1. Daily digest
  - **Covers:** R3, R7, R11
  - **Given:** Hub and Stage 1 link are up; morning report window arrives.
  - **When:** The hub completes its scheduled vision pass.
  - **Then:** The enrolled owner receives one short coded text summarizing beds and notable findings, with unsupported claims marked uncertain, and with no image attachment.

- AE2. Person at gate
  - **Covers:** R4, R8
  - **Given:** A person appears at the watched gate outside any quiet window, and detection passes the confirmation policy.
  - **When:** The hub processes the event.
  - **Then:** The enrolled owner receives a short alert that names the gate event and time.

- AE3. Q&A without images
  - **Covers:** R6, R7, R14
  - **Given:** Images on the hub are within the stale threshold (or the hub can capture fresh ones).
  - **When:** The enrolled owner texts a question such as whether bed 2 looks wilted.
  - **Then:** The owner receives a short-text answer that includes capture age / last-seen time; no image is sent to the city.

- AE4. Missed heartbeat
  - **Covers:** R5
  - **Given:** A heartbeat was expected and none arrives within the grace the product defines.
  - **When:** The owner notices the gap.
  - **Then:** The situation is treated as monitoring-path unreliable and plot status unknown — not as an all-clear plot and not as automatic plot emergency.

- AE5. Internet outage, local cameras remain
  - **Covers:** R9, R11, R12
  - **Given:** Public internet to the plot is down; the required local-network camera still works; Stage 1 link still delivers short messages.
  - **When:** The daily window or a gate event occurs.
  - **Then:** The enrolled owner still receives the corresponding short text sourced from local cameras.

- AE6. Vendor camera only while net is up
  - **Covers:** R9, R10
  - **Given:** A vendor/cloud camera is configured alongside the required local camera, and the public internet later drops.
  - **When:** The hub needs images during the outage.
  - **Then:** The hub does not depend on that vendor camera; it uses the local-network camera for Stage 2 duties.

- AE7. Expected visitor quieted
  - **Covers:** R8
  - **Given:** The owner has defined a quiet window covering a caretaker visit.
  - **When:** The caretaker appears at the gate during that window.
  - **Then:** The hub does not send an unexpected person-at-gate alert for that presence.

- AE8. Non-owner question ignored
  - **Covers:** R13
  - **Given:** A short text question arrives that is not attributable to the enrolled owner.
  - **When:** The hub would otherwise answer Q&A.
  - **Then:** The hub does not return plot status to that sender.

### Success Criteria

- After a week of use, the enrolled owner can skip a routine drive because daily digests, person-at-gate alerts, and Q&A about what the hub saw are trustworthy enough in short text (including age-marked answers and controlled false alarms).
- During a public-network outage that still allows Stage 1 messaging, the owner continues to receive digests/alerts/Q&A from the required local-network camera.
- A missed heartbeat is understood as monitoring-path risk and unknown plot status, not reassurance about the plot and not automatic plot emergency.

### Scope Boundaries

**Deferred for later**

- Neighbor or caretaker as a second recipient of security alerts (Twin audience).
- Quiet event-only mode without a daily digest.
- Strict ban on any vendor/cloud camera.
- Live video to the owner's phone as the primary experience.
- Stage 3 robot gantry seeding, weeding, and gantry imaging product work.
- Explicit proactive "heartbeat missing" push notifications beyond the owner noticing the gap.

**Outside this product's identity**

- Replacing Stage 1 water hardware or inventing the multi-channel link inside Stage 2.
- Becoming a general neighborhood security network.
- Depending on cloud vision or FarmBot cloud image upload as the primary Stage 2 path (this repo's FarmBot web app stores webcam URLs and GCS-backed device photos for a different product model).

### Dependencies / Assumptions

- Stage 1 link exists or will exist soon enough to carry compact inbound and outbound messages (SMS / low-power data, LoRa relay chain, later amateur shortwave as channels evolve).
- Stage 2 requires from Stage 1 either sender/recipient authenticity sufficient for enrolled-owner binding (R13), or an opaque pipe plus Stage 2's own enrollment mechanism.
- The plot has power for the hub and at least one local-network camera when internet is down.
- Coded report vocabulary can be compact enough for Stage 1 message size limits.
- Verified contrast in this repository: no Plot Guardian product exists today; FarmBot webcam feeds are external URLs rendered in the browser; device photos use GCS; weed detection / `take_photo` are device camera ops; cloud OpenAI here assists Lua/sequences, not on-plot offline vision reporting.

### Outstanding Questions

**Resolve Before Planning**

- None.

**Deferred to Implementation**

- Exact plant/weed model weights and training set for Romanian vegetable beds (U4 ships interface + stub; real weights are a data task).
- Vendor/cloud camera adapter auth details when internet is up (optional path; stub OK for first ship).
- Whether any FarmBot on-device camera/plant-detection pieces are reused as optional Stage 3 adjacency later (explicitly out of Stage 2 units).

### Sources / Research

- Dialogue settled Stage 2 scope, Guardian digest + rare heartbeat, soft image rule, and calamity-oriented durability.
- Repo research: no on-plot hub exists; FarmBot GCS/WebcamFeed/MQTT patterns are contrast-only; new top-level package is greenfield vs Rails monolith.
- External landscape: Frigate (MIT) for local RTSP person events; Pi 5 + Hailo/NCNN YOLO for edge weeds; Gammu/SMSD-style GSM modem for offline SMS; LoRa alone is insufficient for 50–100 km city reach; Ultralytics AGPL caution for closed hubs.
- Document review (2026-11-15): local-camera must-have, confidence policy, quiet windows, owner binding, no local-frame upload, Q&A freshness, monitoring-path heartbeat meaning.

---

## Planning Contract

### Key Technical Decisions

- KTD1. **New top-level `plot_guardian/` package.** (session-settled: user-approved — chosen over extending FarmBot `app/`/`frontend/`: cloud GCS/webcam/MQTT patterns fight on-plot image retention.) Hub runtime is Python + Docker Compose on a Raspberry Pi–class SBC. No Stage 2 changes under `app/` or `frontend/`.
- KTD2. **Frigate for person/gate + pluggable plant vision.** Prefer Frigate (MIT) for RTSP person events and local retention; plant/wilt/weed runs as a separate detector plugin (ONNX/NCNN). Avoid Ultralytics AGPL weights as the default plant stack unless Enterprise is acquired.
- KTD3. **`LinkPort` abstraction; first adapter is local GSM/SMS.** Stage 2 does not build the LoRa relay chain. SMS via hub modem (Gammu/SMSD-style) is the first concrete adapter; a Stage1Stub adapter supports tests. LoRa remains an interface stub for Stage 1.
- KTD4. **Enrollment = allowlist + shared secret or challenge–response.** From-number alone is spoofable. Outbound digests/alerts/heartbeats and inbound Q&A bind to enrolled owner channel(s) per R13.
- KTD5. **Compact text codec, not JSON images.** One short line / few-line schema for digest, alert, heartbeat, Q&A; never attach images (R14).
- KTD6. **Configurable defaults for intervals.** Digest daily at configured local time; heartbeat every 12h with 6h grace; Q&A stale threshold 2h; person confidence score threshold configurable (default 0.7). Owner-facing missed-heartbeat copy uses monitoring-path unknown wording (R5).

### Alternatives Considered

- **Extend FarmBot device photo + MQTT logs** — rejected: GCS upload and browser webcam URLs contradict R1/R10/R14 and calamity offline vision.
- **Single custom YOLO stack for gate and beds** — deferred as optional later consolidation; Frigate wins first ship for person/events/retention; plant models remain swappable.
- **Twilio cloud SMS as primary adapter** — rejected for calamity path (needs internet); may be a later city-side companion, not hub primary.
- **LoRa as sole 50–100 km pipe** — rejected by range research; LoRa is plot↔relay (Stage 1), SMS/cell is last mile.

### High-Level Technical Design

```mermaid
flowchart TB
  cams[RTSP/USB cameras] --> frigate[Frigate person events]
  cams --> plant[Plant vision plugin]
  frigate --> hub[plot_guardian hub]
  plant --> hub
  hub --> codec[Compact message codec]
  codec --> link[LinkPort]
  link --> sms[SMS modem adapter]
  link --> stub[Stage1Stub tests]
  owner[Enrolled owner phone] --> sms
  sms --> owner
  cfg[hub.yaml enrollment quiet windows] --> hub
```

### Output Structure

```text
plot_guardian/
  README.md
  pyproject.toml
  docker-compose.yml          # hub + Frigate
  config/hub.example.yaml
  src/plot_guardian/
    __init__.py
    codec.py                  # compact messages
    link/
      base.py                 # LinkPort
      sms_modem.py
      stub.py
      enrollment.py
    cameras/
      rtsp.py
      store.py                # local frames; no upload
    vision/
      person_frigate.py
      plant_plugin.py
      confidence.py
    schedule/
      digest.py
      heartbeat.py
      quiet_windows.py
    qa/
      service.py
    hub.py                    # orchestration
  tests/
    test_scaffold_imports.py
    test_codec.py
    test_enrollment.py
    test_link_stub.py
    test_camera_store.py
    test_confidence.py
    test_quiet_windows.py
    test_qa_freshness.py
    test_digest_heartbeat.py
    test_hub_flows.py
```

### Assumptions

- Hub has LAN reach to at least one RTSP or USB camera and enough power for Pi-class duty cycles.
- First ship may run plant vision as a stub returning `uncertain` until weights exist — gate/person path must still work.
- Owner can complete one-time enrollment while a trusted channel exists (physical presence or temporary connectivity).
- Stage 1 LoRa relays are not required for Stage 2 SMS adapter tests.

### System-Wide Impact

- Adds a sibling product tree beside FarmBot web; CI should run `plot_guardian` tests without coupling to `rspec`/`bun test` of the web app unless a root CI matrix is extended later.
- Does not change FarmBot device firmware, GCS images, or OpenAI Lua paths.

### Implementation Constraints

- Do not add dependencies that require cloud inference at runtime for the primary path.
- Do not upload hub-managed frames off-plot (R10).
- Prefer MIT/Apache-licensed default detectors; document AGPL if an optional plugin uses it.

### Sequencing

U1 first. U2→U3 and U4→U5 can proceed in parallel after U1 (U4 does not need LinkPort). U6 needs U2–U5; U7 integrates all.

---

## Implementation Units

### U1. Scaffold `plot_guardian` package

- **Goal:** Create the runnable package layout, example config, and Docker Compose skeleton with Frigate service stubbed.
- **Requirements:** R1
- **Dependencies:** None
- **Files:** `plot_guardian/pyproject.toml`, `plot_guardian/docker-compose.yml`, `plot_guardian/config/hub.example.yaml`, `plot_guardian/src/plot_guardian/__init__.py`, `plot_guardian/README.md`, `plot_guardian/tests/test_scaffold_imports.py`
- **Approach:**
  1. Add Python package metadata and pytest.
  2. Example `hub.yaml` keys: owner enrollment, camera URLs, digest time, heartbeat interval, stale threshold, confidence threshold, quiet windows.
  3. Compose file mounts config and declares Frigate + hub services (hub command can be placeholder until U7).
- **Patterns to follow:** Greenfield package; do not mirror Rails `app/mutations`.
- **Test scenarios:**
  - Package imports cleanly under pytest.
  - Example config loads and exposes required keys.
- **Verification:** `cd plot_guardian && pytest tests/test_scaffold_imports.py`

### U2. Compact message codec

- **Goal:** Encode/decode digest, person-at-gate alert, heartbeat, and Q&A reply/request as short texts without images.
- **Requirements:** R3, R4, R5, R6, R7, R14; F1–F4; AE1–AE4
- **Dependencies:** U1
- **Files:** `plot_guardian/src/plot_guardian/codec.py`, `plot_guardian/tests/test_codec.py`
- **Approach:**
  1. Define message kinds: `DIGEST`, `ALERT`, `HEARTBEAT`, `QA_REQ`, `QA_ANS`.
  2. Digest fields include bed codes and `uncertain` markers (R7).
  3. Alert includes gate id + time; heartbeat includes last-seen; Q&A answer includes image age.
  4. Reject any payload that would embed image bytes/URLs for owner-facing messages.
- **Test scenarios:**
  - Round-trip each message kind.
  - Uncertain wilt claim encodes visibly.
  - Codec refuses image attachment fields.
  - Covers AE1 / AE2 / AE3 / AE4 wording shapes at the text layer.

### U3. LinkPort, SMS adapter, enrollment

- **Goal:** Deliver and accept short texts only for the enrolled owner.
- **Requirements:** R11, R13; AE8; KTD3, KTD4
- **Dependencies:** U2
- **Files:** `plot_guardian/src/plot_guardian/link/base.py`, `plot_guardian/src/plot_guardian/link/sms_modem.py`, `plot_guardian/src/plot_guardian/link/stub.py`, `plot_guardian/src/plot_guardian/link/enrollment.py`, `plot_guardian/tests/test_enrollment.py`, `plot_guardian/tests/test_link_stub.py`
- **Approach:**
  1. `LinkPort.send(text)` / `LinkPort.poll_inbound()` interface.
  2. `Stage1Stub` records outbound and injects inbound for tests.
  3. SMS modem adapter wraps local modem/SMSD; keep hardware optional behind interface.
  4. Enrollment store: owner address + secret; inbound auth = allowlist + secret/challenge (not From alone).
- **Test scenarios:**
  - Covers AE8: non-enrolled inbound Q&A ignored.
  - Enrolled inbound accepted after secret/challenge.
  - Stub send records digest text for later hub tests.
  - Spoofed From without secret rejected.

### U4. Cameras and local frame store

- **Goal:** Require local RTSP/USB capture; store frames on hub; never upload hub-managed frames.
- **Requirements:** R9, R10, R12, R14; AE5, AE6
- **Dependencies:** U1
- **Files:** `plot_guardian/src/plot_guardian/cameras/rtsp.py`, `plot_guardian/src/plot_guardian/cameras/store.py`, `plot_guardian/tests/test_camera_store.py`
- **Approach:**
  1. Config must list ≥1 local camera; boot fails closed if missing.
  2. Frame store writes under hub data dir with capture timestamps.
  3. Explicit `upload`/`forward` APIs absent; tests assert no HTTP upload helpers for local frames.
  4. Optional vendor camera source gated on `internet_ok` flag (mockable).
- **Test scenarios:**
  - Covers AE5: with internet down, local camera still yields frames.
  - Covers AE6: vendor-only config without local camera fails validation.
  - Stored frame exposes age for Q&A.
  - No off-plot upload function invoked for local frames.

### U5. Vision: person confidence + plant plugin

- **Goal:** Person-at-gate events with confirmation policy; plant plugin returns wilt/weed or uncertain.
- **Requirements:** R2, R4, R7, R8; AE2, AE7; KTD2, KTD6
- **Dependencies:** U4
- **Files:** `plot_guardian/src/plot_guardian/vision/person_frigate.py`, `plot_guardian/src/plot_guardian/vision/plant_plugin.py`, `plot_guardian/src/plot_guardian/vision/confidence.py`, `plot_guardian/src/plot_guardian/schedule/quiet_windows.py`, `plot_guardian/tests/test_confidence.py`, `plot_guardian/tests/test_quiet_windows.py`
- **Approach:**
  1. Map Frigate MQTT/event JSON into internal `PersonEvent` with score.
  2. Emit alert only if score ≥ threshold and outside quiet windows.
  3. Plant plugin interface; default stub returns uncertain until weights supplied.
  4. Quiet windows from config (cron-like or start/end list).
- **Test scenarios:**
  - Covers AE2: high-score person outside quiet window → alert candidate.
  - Low-score person suppressed.
  - Covers AE7: person inside quiet window suppressed.
  - Plant stub yields uncertain, never false-certain wilt.

### U6. Digest, heartbeat, Q&A services

- **Goal:** Daily digest, rare heartbeats, and age-aware Q&A answers.
- **Requirements:** R3, R5, R6, R7; F1, F3, F4; AE1, AE3, AE4
- **Dependencies:** U2, U3, U4, U5
- **Files:** `plot_guardian/src/plot_guardian/schedule/digest.py`, `plot_guardian/src/plot_guardian/schedule/heartbeat.py`, `plot_guardian/src/plot_guardian/qa/service.py`, `plot_guardian/tests/test_qa_freshness.py`, `plot_guardian/tests/test_digest_heartbeat.py`
- **Approach:**
  1. Digest job pulls latest plant+person summaries, encodes via codec, sends via LinkPort.
  2. Heartbeat job sends alive line on interval from KTD6.
  3. Q&A: if latest frame age > stale threshold, capture fresh; always include age in answer; mark uncertain per R7.
- **Test scenarios:**
  - Covers AE1: digest send uses stub LinkPort.
  - Covers AE3: answer includes age; stale frames trigger recapture hook.
  - Covers AE4: with KTD6 interval/grace constants, a missed heartbeat is classified as monitoring-path unknown (not all-clear, not plot emergency); owner-facing copy matches that constant.
  - Heartbeat text matches alive schema.

### U7. Hub orchestration and flow tests

- **Goal:** Wire schedule, vision, cameras, and link into one hub process; prove end-to-end flows with mocks.
- **Requirements:** R1–R14 as integrated; F1–F4; AE1–AE8
- **Dependencies:** U1–U6
- **Files:** `plot_guardian/src/plot_guardian/hub.py`, `plot_guardian/tests/test_hub_flows.py`, `plot_guardian/README.md`
- **Approach:**
  1. Hub loads config, validates local camera + enrollment, starts loops.
  2. Integration tests inject Frigate events and inbound SMS via stubs.
  3. README documents enrollment, missed-heartbeat meaning, and that FarmBot cloud is out of path.
- **Execution note:** Prefer characterization-style hub flow tests before any hardware smoke.
- **Test scenarios:**
  - Covers AE1–AE8 via stubbed cameras/link/vision where applicable.
  - Boot fails without local camera or enrollment.
  - Internet-down flag disables vendor source only.

---

## Verification Contract

| Gate | Command | Applies |
|------|---------|---------|
| Unit + flow tests | `cd plot_guardian && pytest` | Always for Stage 2 units |
| Lint (when configured) | `cd plot_guardian && ruff check src tests` | After U1 adds ruff |
| FarmBot suite | Do **not** require `rspec` / `bun test` for Stage 2-only diffs | Unless `app/` or `frontend/` touched (they should not be) |
| Hardware smoke | Optional: Frigate + one RTSP cam + modem on Pi | Manual; not CI-blocking |

---

## Definition of Done

- All U1–U7 test scenarios pass under `pytest`.
- Product Contract R1–R14 each cited by at least one unit.
- No Stage 2 code path uploads hub-managed frames or depends on FarmBot GCS/OpenAI vision.
- README states enrollment, monitoring-path missed-heartbeat meaning, and Stage 1/3 boundaries.
- Abandoned experiment code removed from the diff.
- PR description notes `plot_guardian/` is intentionally separate from FarmBot web runtime.
