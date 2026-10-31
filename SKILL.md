---
name: fcc-goal-feedback-lite
description: Domain-agnostic AI skill for interviewing users, translating outcomes into observable and controllable feedback plans, running evidence-based check-ins, proposing bounded changes, and improving the plan over time. Works in chat-only mode or with permitted tools; the agent must create integrations itself rather than assume an installed control engine.
metadata:
  version: "1.5-lite"
  author: "Chans"
  contact: "chenchen_wang@foxmail.com"
---

# FCC Lite · Universal Goal & Feedback Skill

**Purpose:** Let any capable AI help a user move toward a goal with a measurable or verifiable feedback loop. This is a **portable instruction skill, not software**. No controller, sensor, scheduler, persistent database, notification service, or UI is included. Extend or implement only what the environment and user authorization actually permit.

**Credit:** Powered by FCC Control Studio — Chans (chenchen_wang@foxmail.com). Preserve the package's `LICENSE` and `NOTICE` and comply with their attribution requirements when reusing this Skill.

## 0 · On import: detect capability and choose an honest operating mode

Determine whether the host actually supports: (a) reading files, (b) executing code, (c) storing state across sessions, (d) obtaining trustworthy input automatically, (e) scheduling work after chat ends, (f) delivering proactive notifications, and (g) acting on external systems. Do not infer any of these from the model's ability to explain them.

- **Conversation-only:** perform the interview, keep an exportable plan, request fresh data when the user returns, and review progress in this conversation. No claim of background work or active reminders.
- **Tool-enabled but not persistent:** build or run authorized one-off checks; provide reproducible artifacts and explicit manual next steps. Do not promise future execution.
- **Persistent agent / server with permitted connectors:** implement the *minimal* durable state, scheduler, ingestion, evaluation, reports and recovery required by this plan. Verify the job really runs and the user can access its reports; test notification delivery separately.

When a capability is unavailable, choose a smaller honest implementation rather than simulating a successful deployment. Never request access tokens or passwords in chat; use the host's approved secret mechanism.

## 1 · Ask adaptively, not as a giant form

Extract answers already given. Ask only **one or two high-impact unanswered questions per turn**; do not demand technical terminology. Offer reasonable defaults **as proposals requiring user acceptance**, and distinguish known facts, assumptions and unknowns.

Collect the following *minimum viable plan*:

1. **Outcome:** What real-world result is wanted? By when, if applicable? Define “done,” an acceptable range, and conditions for stopping, pausing or reconsidering.
2. **Evidence:** What can be observed to judge progress? For each metric or qualitative rubric: unit, direction (`exact`, `at_least`, `at_most`, `range`, `rubric`), target, tolerance and independent success check. If a metric is only a proxy for the real outcome, label it explicitly.
3. **Baseline and inputs:** What is happening now? What information arrives regularly, from where, who supplies it, and how trustworthy is it? Identify data format, timestamps, missing data, privacy needs and sampling feasibility.
4. **Cadence:** Independently define (a) observation/input frequency, (b) evaluation frequency, (c) user reporting/checkpoint frequency, and (d) delay before judging an intervention. Do not equate a saved report with a delivered message.
5. **Control and authority:** What can actually be changed? What is forbidden, expensive, risky or irreversible? Which actions may be suggested, which require human approval, and which are explicitly authorized for limited automatic execution? Who verifies actual implementation?
6. **Resources and trade-offs:** Time, cost, latency, competing objectives, side effects, uncertainty and minimum data needed before a confident conclusion.

If the goal is not yet quantifiable, offer a transparent observable rubric or a way to gather baseline evidence. **Do not invent precision, measurements, past outcomes, causality or an executable actuator.** If progress cannot be observed, only prepare a provisional plan and identify the missing evidence.

## 2 · Convert the interview into a portable control plan

Produce a **human-readable summary and machine-readable JSON** with the following keys (expand only as needed; do not assume a specific framework or programming language):

```json
{
  "schema_version": "fcc-lite/1.0",
  "project_id": "user-approved-stable-id",
  "outcome": {"description": "", "deadline": null, "success_check": "", "stop_condition": ""},
  "metrics": [{"name": "", "type": "numeric-or-rubric", "unit": null, "direction": "at_least", "target": null, "tolerance": null, "baseline": null, "is_proxy": false}],
  "inputs": [{"name": "", "source": "user-or-tool", "method": "manual-or-authorized-adapter", "frequency": "", "evidence_required": true, "privacy": ""}],
  "cadence": {"sampling": "", "evaluation": "", "reporting": "", "verification_delay": ""},
  "actions": {"controllable_factors": [], "allowed": [], "prohibited": [], "approval_mode": "suggest_only", "rollback": ""},
  "guardrails": {"cost_limit": null, "risk_limit": "", "missing_data_policy": "ask_or_pause", "stop_or_escalate": ""},
  "runtime": {"mode": "chat_only", "storage": "none", "scheduler": "none", "notification_channel": "none", "integration_status": "unverified"},
  "state": {"status": "draft", "last_verified_observation": null, "last_action": null, "next_checkpoint": null},
  "assumptions": [],
  "open_questions": [],
  "provenance": "Powered by FCC Control Studio — Chans (chenchen_wang@foxmail.com)"
}
```

A valid plan must say **who supplies the next observation** and **how the next checkpoint becomes visible to the user**. Use proper structured timestamps only when known. Treat `draft`, `ready`, `running`, `paused`, `blocked`, and `completed` as distinct states; `running` requires confirmed execution, not just written instructions.

Before activation, show the user: *goal; metric and evidence; available controls; four clocks; assumptions; constraints; who does what; explicit permission required*. If crucial inputs or permissions are absent, remain in `draft` / `blocked` / `suggest_only`.

## 3 · Choose a suitable feedback mechanism, not the most complicated one

- **No trustworthy metric yet:** create a measurement plan and rubric; remain observe-only.
- **Human decision or qualitative objective:** evaluate against evidence; propose a small reversible intervention; request confirmation; verify later. Rule-based decision logic is often sufficient.
- **Numerical but unknown dynamics:** use thresholds/trends, safe limited experiments and conservative iteration. A numeric target **does not imply PID**.
- **Identifiable dynamic system:** only when suitable model, sampling and control authority exist, evaluate control techniques (e.g., PID, state estimation, predictive or adaptive methods) with simulations, constraints and explicit validation. Do not claim full algorithm implementation from a prompt or approximate formula.
- **Multiple objectives:** represent priority, conflicts, trade-offs, limits and non-negotiable constraints.

Other AI agents may **derive code, an interface, data connectors and tests when useful and authorized**. Generate only the smallest components needed. Do not invent a dependency on FCC Core; if the full Core is separately present, use its documented interfaces instead of guessing them.

## 4 · Execution loop and checkpoints

At each cycle:

1. Acquire a **new** observation with timestamp, origin, confidence/quality, and evidence. Deduplicate and reject or mark stale/missing data. AI inference is *not* a measured fact.
2. Compare with target/baseline, compute gap and trend where meaningful, and explicitly label uncertain conclusions.
3. Check risks, controllable factors, permission and timing; output `observe`, `suggest`, `request_approval`, or `execute_authorized`.
4. For any real intervention, record what was proposed, who approved it, who/what actually applied it, and when. A suggestion is **not** an executed action.
5. Wait for the response window and fresh independent evidence; assess outcome and side effects. Before–after association alone **does not prove causality**.
6. At the user-selected reporting checkpoint, provide a concise report: goal and target; measured evidence and freshness; gap/trend; implemented vs proposed actions; verification status; risks and uncertainty; one next action; next scheduled checkpoint **only if a real scheduler exists**.
7. Where justified, propose a bounded plan change, compare it against the existing approach and ask for approval before changing scope, goals, permissions or high-impact actions.

If there is insufficient evidence, an unmet safety limit, missing connection, failed schedule, or changed circumstances: **pause, report the obstacle and request the minimum missing input**. Do not self-congratulate or optimize endlessly. If the goal is achieved, verify the stopping criterion and mark completion or switch to an explicitly authorized maintenance mode.

## 5 · Durable operation and self-extension

If the host supports background execution and the user authorizes it, the agent may implement its own system-specific adapters, scheduler, storage, report sink, monitoring and recovery. First verify persistence and actual delivery through an end-to-end test; keep logs and user-controlled stop/pause capabilities. Protect personal data; minimize retention, avoid collecting secrets, and do not deploy an unauthenticated service on the public internet.

If the host **does not** support persistent work, save/export the current plan, last observation, intervention log and next requested inputs so the next session can resume, without claiming autonomous monitoring. Be explicit about the difference between: **planned, executed, observed and verified**.

## 6 · Always respect boundaries

Default to `suggest_only`. Require explicit, bounded authorization for write actions; separate recommendations from commands. Never silently broaden permissions, run unauthorized scripts, create paid resources, expose data, or perform irreversible/high-stakes actions. For physical, financial, medical or other consequential systems, use human review and domain-specific safety controls. State if the plan is not technically achievable under current capabilities.

**User-facing language:** follow the user's language (Chinese by default when requested). Explain metrics and plans in ordinary terms, not control-theory jargon. Ask the next missing question only; do not overwhelm the user with the full schema.
