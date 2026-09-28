# Shift Handoff Wizard, Simplified PRD (RouteLogic)

**Author:** Me · **Status:** Draft · **Target:** High-Fidelity Prototype · **Persona:** The Frontline Driver, A working delivery driver who performs the same core actions dozens of times per shift under real-world physical constraints.

## 1. The Big Picture
- **Vision:** Hand off a shift in one guided flow instead of several disconnected screens, so a driver with no spare hand and no spare time can close out and walk away.
- **Press release:** RouteLogic today announced the Shift Handoff Wizard, a guided, one-handed handoff flow for delivery drivers. Instead of hunting through multiple screens to report vehicle status, remaining packages, and compliance flags at the end of a shift, drivers now move through a single linear sequence with large tap targets and data that's already pre-filled from the shift in progress.

"The old handoff process assumed I had two free hands and five spare minutes at the exact moment I have neither," said a RouteLogic pilot driver. "Now it's mostly taps, not typing, and it works even when the yard has no signal." The Wizard is rolling out to 3 pilot accounts over a 4-week pilot, with a goal of cutting handoff-related time on task from 11.3 minutes toward a 4.5-minute benchmark.
- **Success metric:** Time on task for shift handoff 	11.3 min → 4.5 min
- **Guardrail:** the user churn rate for drivers against the complete shift handoff

## 2. The Details
### User stories
- As a driver ending my shift, I want to confirm my handoff in one guided flow, so that I'm not switching between screens with one free hand.
- As a driver, I want my current shift data (vehicle status, remaining packages, compliance flags) pre-filled into the handoff, so that I don't retype what the app already knows.
- As a driver finishing in a low-signal yard or dock, I want the handoff to save even if I lose connection, so that a bad signal doesn't force me to redo it or leave my shift open.
### Screens to build
- Entry Point
- Feature Core (Wizard)
- Success / Confirmation
### Functional requirements
- The wizard must complete in a single linear flow with no more than 3 screens from entry to confirmation.
- Vehicle status, remaining-package count, and open compliance flags must auto-populate from the active shift record on wizard load.
- Every interactive control must have a minimum tap target of 44x44px and be operable with one thumb.
- Handoff progress must persist locally and queue for submission when the device has no connectivity.
- Tapping "Complete Handoff" must mark the shift closed and compliant in a single action, with no secondary confirmation step.
- A timestamped handoff-completion event must be logged on submission, synced or queued.
- The wizard must load with pre-filled data in under 2 seconds on a mid-tier Android device on 3G.
### Smart behaviors (Situation → Outcome)
- If driver opens the wizard with an open compliance flag on the shift, then, flag is shown pre-checked into the flow; driver must tap to acknowledge before "Next" activates.
- If, driver opens the wizard with no changes since shift start, then, all fields show pre-filled values; driver can tap "Complete Handoff" directly from screen 2 with no edits required.
- If, Device loses connectivity mid-flow, then, Wizard continues to function locally; submission queues silently; UI shows "Saved — will sync" instead of blocking.
- If. handoff is submitted twice (e.g. duplicate tap or retry after reconnect), then, second submission is deduplicated against the queued event; shift is marked complete once, not twice.
### Technical constraints
- No external APIs or third-party integrations for this sprint — all data comes from existing shift records already available to the app.
- No new login or authentication flow — the wizard runs inside the driver's existing authenticated session.
- No new persistent state management layer — local component state (useState) is sufficient for a 3-screen linear flow; no global store, no backend schema changes beyond the single completion event.
- No coordinator-facing UI in this build — the wizard is driver-only; any coordinator view is out of scope (see Features Out).

## 3. The Logistics
### Features out
- Coordinator-side dashboard or approval step for handoffs — belongs to the Mobile-First Coordinator Dashboard (B4), a separate NEXT-lane project.
- Compliance audit trail / PDF export tied to handoff data — belongs to Compliance Audit Trail Export (B9); serves Legal/CS, not this sprint's driver-facing goal.
- Manager edit or override of a submitted handoff.
- Cross-driver handoff routing or scheduling logic (assigning who hands off to whom).
- AI-generated summarization of handoff notes.
### Edge cases & safety guard
- If the driver has no compliance flags and no changes to report, then the wizard still requires the explicit "Complete Handoff" tap; nothing is auto-submitted without driver action.
- If the driver force-quits the app mid-handoff, then progress is retained locally and reopening resumes at the last completed step, never a blank restart.
- If pre-filled data is wrong (e.g. stale package count), then the driver can correct it via the visible edit affordance on every pre-filled field before submitting.
- If the device never regains connectivity before the shift-end cutoff, then the handoff stays queued locally and is flagged to CS as an unsynced pilot event for manual follow-up, rather than silently lost.
- If the driver attempts to submit with an unacknowledged compliance flag, then "Complete Handoff" stays inactive until every flag is explicitly acknowledged — the flow cannot silently skip a compliance requirement.
### Decision log
- Ship as a driver-only flow with no coordinator visibility in sprint 1. This is because coordinator dashboarding is a platform-level rebuild (B4) that can't fit a 4-week, 2-engineer pilot; splitting it out keeps the Wizard shippable on time.
- Use local useState and existing shift records only — no new backend schema or third-party APIs. This is because, the Moment of Misery is about steps colliding with a physical moment, not missing data; the fix is re-sequencing and pre-filling data that already exists, not building new infrastructure
### Evals
- Time on task: Median seconds from "Start Handoff" to "Complete Handoff" ≤ 4.5 min (down from 11.3 min baseline).
- % accuracy: Completion accuracy — % of started handoffs that reach "Complete" without abandonment ≥ 90% across the 3 pilot accounts.
- Safety: Safety trigger rate — % of handoffs completed with an unacknowledged compliance flag = 0% — any non-zero rate is a launch blocker, not a tuning issue.

## MoSCoW scope
- **Must:** Single linear guided flow (one screen, one step at a time) replacing the current multi-screen handoff — this is the entire point of a "wizard"; without it, the feature doesn't exist. End-of-shift nudge/reminder if handoff hasn't started — improves completion rates, but a driver who's already in the flow doesn't need it.
- **Should:** Incoming driver's acknowledgment step (confirms receipt of the handed-off shift) — strengthens handoff integrity, but the outgoing driver's time-on-task metric doesn't depend on it.
- **Could:** Photo attachment for vehicle/equipment condition at handoff.
- **Won't (now):** Coordinator-side dashboard or approval step for handoffs — that's B4 (Mobile-First Coordinator Dashboard), a separate NEXT-lane feature; bundling it turns a 1-sprint wizard into a platform project.

---
**Builder hook:** Build a working prototype based on this PRD. Use the User Story as the core flow, Functional Requirements as build constraints, and prioritize speed and clarity over visual complexity.
