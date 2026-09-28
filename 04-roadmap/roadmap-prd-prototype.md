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

## Prototype
The Html code for the prototype is here as a string. Extract it to create a html file and use it to extract screenshots where applicable. 
"<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Shift Handoff Wizard</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
</head>
<body>
<style>
  :root {
    --bg: #0b141d;
    --panel: #0f1a24;
    --surface: #16212c;
    --surface-2: #1c2936;
    --border: #29394a;
    --text: #e9eef4;
    --text-dim: #93a5b8;
    --text-faint: #5e7085;
    --accent: #4f8ef7;
    --accent-dim: #24344a;
    --ok: #4ade80;
    --ok-bg: rgba(74, 222, 128, 0.14);
    --warn: #fbbf24;
    --warn-bg: rgba(251, 191, 36, 0.14);
    --danger: #f87171;
    color-scheme: dark;
  }

  * { box-sizing: border-box; }

  html, body {
    margin: 0;
    background: var(--bg);
    color: var(--text);
    font-family: "IBM Plex Sans", -apple-system, "Segoe UI", sans-serif;
    -webkit-font-smoothing: antialiased;
  }

  body {
    padding: 28px 16px 56px;
  }

  .wrap {
    max-width: 980px;
    margin: 0 auto;
  }

  /* ---- Header / control strip ---- */

  .topbar {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: center;
    gap: 16px;
    margin-bottom: 24px;
  }

  .title-block h1 {
    margin: 0;
    font-size: 18px;
    font-weight: 700;
    letter-spacing: -0.01em;
  }

  .title-block .meta {
    margin-top: 3px;
    font-size: 12px;
    color: var(--text-faint);
    font-family: "IBM Plex Mono", monospace;
    letter-spacing: 0.02em;
  }

  .controls {
    display: flex;
    align-items: center;
    gap: 14px;
    flex-wrap: wrap;
  }

  .toggle {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 12.5px;
    color: var(--text-dim);
  }

  .switch {
    all: unset;
    width: 36px;
    height: 20px;
    border-radius: 999px;
    background: var(--surface-2);
    border: 1px solid var(--border);
    position: relative;
    cursor: pointer;
    flex-shrink: 0;
    transition: background 0.15s ease;
  }

  .switch::after {
    content: "";
    position: absolute;
    top: 2px;
    left: 2px;
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: var(--text-dim);
    transition: transform 0.15s ease, background 0.15s ease;
  }

  .switch.on { background: var(--warn-bg); border-color: var(--warn); }
  .switch.on::after { transform: translateX(16px); background: var(--warn); }

  .reset-btn {
    all: unset;
    cursor: pointer;
    font-size: 12.5px;
    font-weight: 600;
    color: var(--text-dim);
    border: 1px solid var(--border);
    padding: 7px 14px;
    border-radius: 8px;
    background: var(--surface);
  }

  .reset-btn:hover { color: var(--text); border-color: var(--text-faint); }
  .reset-btn:active { transform: translateY(1px); }

  /* ---- Layout: phone + sidebar ---- */

  .stage {
    display: grid;
    grid-template-columns: 360px 1fr;
    gap: 28px;
    align-items: start;
  }

  /* ---- Phone frame ---- */

  .phone {
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 32px;
    padding: 14px;
    box-shadow: 0 30px 60px -30px rgba(0,0,0,0.6);
  }

  .screen {
    background: var(--bg);
    border-radius: 22px;
    min-height: 620px;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    position: relative;
  }

  .statusbar {
    display: flex;
    justify-content: space-between;
    padding: 12px 20px 4px;
    font-size: 11px;
    color: var(--text-faint);
    font-family: "IBM Plex Mono", monospace;
  }

  .screen-body {
    flex: 1;
    padding: 18px 20px 20px;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  /* ---- Screen 1: entry ---- */

  .shift-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 16px;
  }

  .shift-card .label {
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: var(--text-faint);
    margin-bottom: 10px;
  }

  .stat-row {
    display: flex;
    justify-content: space-between;
    padding: 6px 0;
    font-size: 13.5px;
  }

  .stat-row + .stat-row { border-top: 1px solid var(--border); }

  .stat-row .v { font-family: "IBM Plex Mono", monospace; font-variant-numeric: tabular-nums; color: var(--text); }

  .spacer { flex: 1; }

  .btn {
    all: unset;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 52px;
    border-radius: 12px;
    font-size: 15px;
    font-weight: 700;
    cursor: pointer;
    text-align: center;
    transition: transform 0.08s ease, filter 0.15s ease;
  }

  .btn:active { transform: scale(0.98); }
  .btn:disabled { cursor: not-allowed; }

  .btn-primary { background: var(--accent); color: #071019; }
  .btn-primary:disabled { background: var(--accent-dim); color: var(--text-faint); }
  .btn-primary:not(:disabled):hover { filter: brightness(1.08); }

  .btn-secondary {
    background: transparent;
    border: 1px solid var(--border);
    color: var(--text-dim);
  }
  .btn-secondary:hover { border-color: var(--text-faint); color: var(--text); }

  .btn-row { display: flex; gap: 10px; }
  .btn-row .btn { flex: 1; }

  /* ---- Screen 2: wizard core ---- */

  .steps {
    display: flex;
    gap: 6px;
    margin-bottom: 4px;
  }

  .steps .dot {
    flex: 1;
    height: 4px;
    border-radius: 999px;
    background: var(--surface-2);
  }

  .steps .dot.done { background: var(--accent); }

  .field-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 14px 16px;
  }

  .field-top {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .field-top .label {
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: var(--text-faint);
  }

  .edit-link {
    all: unset;
    cursor: pointer;
    font-size: 12px;
    font-weight: 600;
    color: var(--accent);
    padding: 4px 6px;
  }

  .field-value {
    margin-top: 6px;
    font-size: 16px;
    font-weight: 600;
    font-family: "IBM Plex Mono", monospace;
  }

  .field-input {
    margin-top: 8px;
    width: 100%;
    background: var(--surface-2);
    border: 1px solid var(--accent);
    border-radius: 8px;
    color: var(--text);
    font-size: 15px;
    font-family: "IBM Plex Mono", monospace;
    padding: 10px 12px;
  }

  .flag-item {
    display: flex;
    align-items: flex-start;
    gap: 10px;
    padding: 12px;
    border-radius: 10px;
    background: var(--warn-bg);
    border: 1px solid rgba(251,191,36,0.35);
    cursor: pointer;
    min-height: 44px;
  }

  .flag-item.ack {
    background: var(--ok-bg);
    border-color: rgba(74,222,128,0.35);
  }

  .flag-check {
    width: 20px;
    height: 20px;
    border-radius: 6px;
    border: 2px solid var(--warn);
    flex-shrink: 0;
    margin-top: 1px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 13px;
    color: var(--bg);
  }

  .flag-item.ack .flag-check {
    border-color: var(--ok);
    background: var(--ok);
  }

  .flag-text .t { font-size: 13.5px; font-weight: 600; }
  .flag-text .s { font-size: 11.5px; color: var(--text-dim); margin-top: 2px; }
  .flag-item.ack .t { color: var(--ok); }

  .offline-banner {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 12px;
    color: var(--warn);
    background: var(--warn-bg);
    border: 1px solid rgba(251,191,36,0.35);
    border-radius: 8px;
    padding: 8px 10px;
  }

  .offline-banner .pulse {
    width: 7px;
    height: 7px;
    border-radius: 50%;
    background: var(--warn);
    flex-shrink: 0;
  }

  /* ---- Screen 3: success ---- */

  .success-wrap {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 18px;
    text-align: center;
    padding: 20px 8px;
  }

  .check-circle {
    width: 68px;
    height: 68px;
    border-radius: 50%;
    background: var(--ok-bg);
    border: 2px solid var(--ok);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .check-circle svg { width: 32px; height: 32px; }

  .success-wrap h2 { margin: 0; font-size: 19px; }

  .success-wrap .ts {
    font-size: 12.5px;
    color: var(--text-faint);
    font-family: "IBM Plex Mono", monospace;
  }

  .summary-card {
    width: 100%;
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 14px 16px;
    text-align: left;
    font-size: 13px;
    color: var(--text-dim);
    line-height: 1.6;
  }

  .summary-card b { color: var(--text); }

  .sync-badge {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-size: 11.5px;
    font-weight: 700;
    padding: 4px 10px;
    border-radius: 999px;
    text-transform: uppercase;
    letter-spacing: 0.04em;
  }

  .sync-badge.synced { background: var(--ok-bg); color: var(--ok); }
  .sync-badge.queued { background: var(--warn-bg); color: var(--warn); }

  /* ---- Sidebar: scenario / spec trace ---- */

  .sidebar {
    display: flex;
    flex-direction: column;
    gap: 14px;
  }

  .panel {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 16px 18px;
  }

  .panel h3 {
    margin: 0 0 10px;
    font-size: 12px;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: var(--text-faint);
  }

  .panel p {
    margin: 0;
    font-size: 13.5px;
    color: var(--text-dim);
    line-height: 1.55;
  }

  .metric-row {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    padding: 8px 0;
  }

  .metric-row + .metric-row { border-top: 1px solid var(--border); }

  .metric-row .m-label { font-size: 12.5px; color: var(--text-dim); }
  .metric-row .m-value { font-family: "IBM Plex Mono", monospace; font-weight: 600; font-size: 14px; }
  .metric-row .m-value.hit { color: var(--ok); }

  .trace {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .trace-item {
    display: flex;
    gap: 8px;
    font-size: 12.5px;
    color: var(--text-faint);
    align-items: flex-start;
  }

  .trace-item .req-id {
    font-family: "IBM Plex Mono", monospace;
    color: var(--accent);
    flex-shrink: 0;
  }

  .trace-item.active { color: var(--text); }
  .trace-item.active .req-id { color: var(--ok); }

  @media (max-width: 820px) {
    .stage { grid-template-columns: 1fr; }
    .phone { justify-self: center; max-width: 360px; }
  }
</style>

<div class="wrap">

  <div class="topbar">
    <div class="title-block">
      <h1>Shift Handoff Wizard — Prototype</h1>
      <div class="meta">HIGH-FIDELITY · DRIVER FLOW · BUILT FROM PRD v1</div>
    </div>
    <div class="controls">
      <div class="toggle">
        <span>Simulate offline</span>
        <button class="switch" id="offlineSwitch" aria-pressed="false" aria-label="Simulate offline"></button>
      </div>
      <button class="reset-btn" id="resetBtn">Reset prototype</button>
    </div>
  </div>

  <div class="stage">

    <div class="phone">
      <div class="screen" id="screen"></div>
    </div>

    <div class="sidebar">
      <div class="panel">
        <h3>Scenario</h3>
        <p id="scenarioText">A driver is closing out today's route and opens the Shift Handoff Wizard from their active shift. No spare hand, no spare time — the flow needs to complete itself as much as possible.</p>
      </div>

      <div class="panel">
        <h3>Live metric</h3>
        <div class="metric-row">
          <span class="m-label">Time on task</span>
          <span class="m-value" id="timerValue">0:00</span>
        </div>
        <div class="metric-row">
          <span class="m-label">Target</span>
          <span class="m-value hit">≤ 4:30</span>
        </div>
      </div>

      <div class="panel">
        <h3>Functional requirements in play</h3>
        <div class="trace" id="traceList"></div>
      </div>
    </div>

  </div>
</div>

<script>
(function () {
  "use strict";

  // ---- Seed data (pre-filled from "existing shift record") ----
  function freshState() {
    return {
      screen: "entry",       // entry | wizard | success
      step: 1,               // 1 or 2 within wizard
      offline: false,
      vehicleStatus: "No issues reported",
      vehicleEdited: false,
      editingVehicle: false,
      packages: 3,
      packagesEdited: false,
      editingPackages: false,
      flagAcked: false,
      startedAt: null,
      completedAt: null,
      syncState: null,       // "synced" | "queued"
    };
  }

  var state = freshState();
  var timerInterval = null;

  var screenEl = document.getElementById("screen");
  var traceEl = document.getElementById("traceList");
  var timerEl = document.getElementById("timerValue");
  var offlineSwitch = document.getElementById("offlineSwitch");
  var resetBtn = document.getElementById("resetBtn");

  var REQS = {
    entry: [
      ["FR-1", "Single linear flow, max 3 screens"],
    ],
    wizard1: [
      ["FR-2", "Vehicle status & package count auto-populate from shift record"],
      ["FR-3", "44px+ tap targets, one-thumb operable"],
    ],
    wizard2: [
      ["FR-2", "Compliance flags surfaced from shift record"],
      ["Smart", "“Next” stays inactive until every flag is acknowledged"],
    ],
    success: [
      ["FR-5", "“Complete Handoff” closes the shift in one action"],
      ["FR-6", "Timestamped completion event logged"],
      ["FR-4", "Progress queues locally when offline, syncs on reconnect"],
    ],
  };

  function fmtTime(ms) {
    var s = Math.floor(ms / 1000);
    var m = Math.floor(s / 60);
    s = s % 60;
    return m + ":" + (s < 10 ? "0" : "") + s;
  }

  function startTimer() {
    stopTimer();
    state.startedAt = Date.now();
    timerInterval = setInterval(function () {
      timerEl.textContent = fmtTime(Date.now() - state.startedAt);
    }, 1000);
  }

  function stopTimer() {
    if (timerInterval) { clearInterval(timerInterval); timerInterval = null; }
  }

  function renderTrace(key) {
    var items = REQS[key] || [];
    traceEl.innerHTML = items.map(function (r) {
      return '<div class="trace-item active"><span class="req-id">' + r[0] + '</span><span>' + r[1] + '</span></div>';
    }).join("");
  }

  function setState(patch) {
    Object.assign(state, patch);
    render();
  }

  // ---- Screen renderers ----

  function renderEntry() {
    renderTrace("entry");
    screenEl.innerHTML =
      '<div class="statusbar"><span>9:41</span><span>RouteLogic Driver</span></div>' +
      '<div class="screen-body">' +
        '<div class="shift-card">' +
          '<div class="label">Active Shift</div>' +
          '<div class="stat-row"><span>Time on route</span><span class="v">6h 42m</span></div>' +
          '<div class="stat-row"><span>Stops completed</span><span class="v">18 / 18</span></div>' +
          '<div class="stat-row"><span>Vehicle</span><span class="v">Van 114</span></div>' +
        '</div>' +
        '<div class="spacer"></div>' +
        '<button class="btn btn-primary" id="startBtn">Start Handoff</button>' +
      '</div>';

    document.getElementById("startBtn").addEventListener("click", function () {
      startTimer();
      setState({ screen: "wizard", step: 1 });
    });
  }

  function renderWizardStep1() {
    renderTrace("wizard1");
    var vehicleBlock;
    if (state.editingVehicle) {
      vehicleBlock =
        '<input class="field-input" id="vehicleInput" value="' + state.vehicleStatus + '" />';
    } else {
      vehicleBlock = '<div class="field-value">' + state.vehicleStatus + '</div>';
    }

    var pkgBlock;
    if (state.editingPackages) {
      pkgBlock = '<input class="field-input" id="pkgInput" type="number" min="0" value="' + state.packages + '" />';
    } else {
      pkgBlock = '<div class="field-value">' + state.packages + ' remaining</div>';
    }

    screenEl.innerHTML =
      '<div class="statusbar"><span>9:41</span><span>Handoff · Step 1 of 2</span></div>' +
      '<div class="screen-body">' +
        '<div class="steps"><div class="dot done"></div><div class="dot"></div></div>' +
        (state.offline ? '<div class="offline-banner"><span class="pulse"></span>No connection — progress will save on this device</div>' : '') +
        '<div class="field-card">' +
          '<div class="field-top"><span class="label">Vehicle Status</span>' +
            '<button class="edit-link" id="editVehicle">' + (state.editingVehicle ? "Save" : "Edit") + '</button></div>' +
          vehicleBlock +
        '</div>' +
        '<div class="field-card">' +
          '<div class="field-top"><span class="label">Packages Remaining</span>' +
            '<button class="edit-link" id="editPkg">' + (state.editingPackages ? "Save" : "Edit") + '</button></div>' +
          pkgBlock +
        '</div>' +
        '<div class="spacer"></div>' +
        '<div class="btn-row">' +
          '<button class="btn btn-secondary" id="backBtn">Back</button>' +
          '<button class="btn btn-primary" id="nextBtn">Next</button>' +
        '</div>' +
      '</div>';

    document.getElementById("backBtn").addEventListener("click", function () {
      stopTimer();
      setState(Object.assign(freshState(), { offline: state.offline }));
    });

    document.getElementById("nextBtn").addEventListener("click", function () {
      setState({ step: 2 });
    });

    document.getElementById("editVehicle").addEventListener("click", function () {
      if (state.editingVehicle) {
        var val = document.getElementById("vehicleInput").value.trim() || state.vehicleStatus;
        setState({ vehicleStatus: val, vehicleEdited: true, editingVehicle: false });
      } else {
        setState({ editingVehicle: true });
      }
    });

    document.getElementById("editPkg").addEventListener("click", function () {
      if (state.editingPackages) {
        var val = parseInt(document.getElementById("pkgInput").value, 10);
        if (isNaN(val) || val < 0) val = state.packages;
        setState({ packages: val, packagesEdited: true, editingPackages: false });
      } else {
        setState({ editingPackages: true });
      }
    });
  }

  function renderWizardStep2() {
    renderTrace("wizard2");
    screenEl.innerHTML =
      '<div class="statusbar"><span>9:41</span><span>Handoff · Step 2 of 2</span></div>' +
      '<div class="screen-body">' +
        '<div class="steps"><div class="dot done"></div><div class="dot done"></div></div>' +
        (state.offline ? '<div class="offline-banner"><span class="pulse"></span>No connection — progress will save on this device</div>' : '') +
        '<div class="field-card">' +
          '<div class="label" style="margin-bottom:10px;">Compliance Flag</div>' +
          '<div class="flag-item' + (state.flagAcked ? " ack" : "") + '" id="flagItem" role="checkbox" aria-checked="' + state.flagAcked + '">' +
            '<div class="flag-check">' + (state.flagAcked ? "✓" : "") + '</div>' +
            '<div class="flag-text">' +
              '<div class="t">Seatbelt reminder not acknowledged (2 instances today)</div>' +
              '<div class="s">Tap to confirm you’ve reviewed this with your route</div>' +
            '</div>' +
          '</div>' +
        '</div>' +
        '<div class="spacer"></div>' +
        '<div class="btn-row">' +
          '<button class="btn btn-secondary" id="backBtn">Back</button>' +
          '<button class="btn btn-primary" id="completeBtn"' + (state.flagAcked ? "" : " disabled") + '>Complete Handoff</button>' +
        '</div>' +
      '</div>';

    document.getElementById("backBtn").addEventListener("click", function () {
      setState({ step: 1 });
    });

    document.getElementById("flagItem").addEventListener("click", function () {
      setState({ flagAcked: !state.flagAcked });
    });

    var completeBtn = document.getElementById("completeBtn");
    completeBtn.addEventListener("click", function () {
      if (!state.flagAcked) return;
      stopTimer();
      var now = new Date();
      setState({
        screen: "success",
        completedAt: now,
        syncState: state.offline ? "queued" : "synced",
      });
    });
  }

  function renderSuccess() {
    renderTrace("success");
    var ts = state.completedAt.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    var synced = state.syncState === "synced";

    screenEl.innerHTML =
      '<div class="statusbar"><span>9:41</span><span>Handoff Complete</span></div>' +
      '<div class="success-wrap">' +
        '<div class="check-circle"><svg viewBox="0 0 24 24" fill="none"><path d="M5 13l4 4L19 7" stroke="#4ade80" stroke-width="2.6" stroke-linecap="round" stroke-linejoin="round"/></svg></div>' +
        '<h2>Shift closed out</h2>' +
        '<div class="ts">Completed ' + ts + ' · Time on task ' + timerEl.textContent + '</div>' +
        '<span class="sync-badge ' + (synced ? "synced" : "queued") + '">' + (synced ? "Saved" : "Saved — will sync") + '</span>' +
        '<div class="summary-card">' +
          '<div><b>Vehicle:</b> ' + state.vehicleStatus + (state.vehicleEdited ? " (edited)" : " (unchanged)") + '</div>' +
          '<div><b>Packages remaining:</b> ' + state.packages + (state.packagesEdited ? " (edited)" : " (unchanged)") + '</div>' +
          '<div><b>Compliance flag:</b> acknowledged</div>' +
        '</div>' +
        '<button class="btn btn-secondary" id="doneBtn" style="width:100%;">Done</button>' +
      '</div>';

    document.getElementById("doneBtn").addEventListener("click", function () {
      setState(Object.assign(freshState(), { offline: state.offline }));
    });
  }

  function render() {
    if (state.screen === "entry") return renderEntry();
    if (state.screen === "wizard" && state.step === 1) return renderWizardStep1();
    if (state.screen === "wizard" && state.step === 2) return renderWizardStep2();
    if (state.screen === "success") return renderSuccess();
  }

  offlineSwitch.addEventListener("click", function () {
    var next = !state.offline;
    offlineSwitch.classList.toggle("on", next);
    offlineSwitch.setAttribute("aria-pressed", String(next));
    setState({ offline: next });
  });

  resetBtn.addEventListener("click", function () {
    stopTimer();
    var offline = state.offline;
    state = freshState();
    state.offline = offline;
    timerEl.textContent = "0:00";
    render();
  });

  render();
})();
</script>

</body>
</html>"
**Builder hook:** Build a working prototype based on this PRD. Use the User Story as the core flow, Functional Requirements as build constraints, and prioritize speed and clarity over visual complexity.

