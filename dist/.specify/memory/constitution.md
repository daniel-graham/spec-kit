<!-- Sync Impact Report
Version change: 1.1.0 -> 1.2.0
Modified principles: 
  - I. Deterministic Supremacy -> I. Prime Directives (Order of Precedence)
  - II. Capital Preservation First -> II. Risk Management Invariants (Futures-First)
  - III. Separation of Concerns -> III. Determinism, Auditability, Replay
  - IV. Safe AI Integration -> IV. Separation of Concerns (No Cross-Contamination)
  - V. Security & Observability -> V. Safe AI Integration (LLM Governance)
Added sections: 
  - VI. Data Integrity & Fail-Closed Trading
  - VII. Observability: No Silent Failure
  - VIII. Futures Session & VWAP Canonical Policy
Removed sections: None
Templates requiring updates:
- ✅ .specify/templates/plan-template.md 
- ✅ .specify/templates/spec-template.md 
- ✅ .specify/templates/tasks-template.md
Follow-up TODOs: Determine original RATIFICATION_DATE
-->
# Cloud-Edge Constitution

This Constitution defines **non‑negotiable invariants**. If any implementation, spec, or prompt conflicts with it, the implementation must change.

## 1) Prime Directives (Order of Precedence)
1. **Capital Preservation** (survive first).
2. **Deterministic Correctness** (no silent wrong signals or accidental orders).
3. **Risk Engine Supremacy** (risk has the final word).
4. **Operational Safety** (fail-closed on uncertainty).
5. **Performance & Features** (only after 1–4 are satisfied).

## 2) Risk Management Invariants (Futures-First)
### 2.1 Position sizing and max loss
- Each trade MUST have a **pre-defined risk** (stop distance + size) and MUST be sized by **risk**, not by conviction.
- Default per-trade risk target: **1%–2% of account equity** (configurable), with a hard ceiling enforced by the risk engine.
- The system MUST also enforce **absolute caps** (e.g., **-$500 max daily loss**) and **statistical guards** (e.g., pause after **5 consecutive losses**).

### 2.2 Stops are mandatory
- Every live trade MUST have a stop-loss at entry time (bracket OCO or equivalent).
- Stops MAY be tightened by deterministic rules; stops MUST NOT be loosened (no “hope” risk expansion).

### 2.3 Cut losses quickly, protect gains
- Losing trades MUST be exited at the stop or earlier per deterministic invalidation.
- “Once you’ve made a profit, don’t give it back” is implemented via daily max loss hard stop, optional trailing/BE logic, and stand-down rules after abnormal slippage/volatility spikes.

### 2.4 Never average down
- The system MUST NOT add size to a losing position (no martingale, no “improving average”).
- Scaling in is allowed only when **not in drawdown on the position** and only via explicit, tested rules.

### 2.5 Risk/Reward discipline
- Every entry MUST declare invalidation (stop) and at least one profit-taking/exit plan.
- The system SHOULD enforce a minimum R:R policy (e.g., >= 1.5R) validated in backtest.

### 2.6 Portfolio / exposure controls
- Maximum position size, maximum order rate, and maximum daily trades MUST be enforced.
- All positions MUST be flat by **EOD** unless an explicit, ratified swing-mode exists.

### 2.7 Costs matter
- Commissions and slippage MUST be tracked and surfaced; strategies must not rely on unrealistic fills.

## 3) Determinism, Auditability, Replay
### 3.1 Deterministic decisioning
Given the same input event stream, the system MUST reproduce the same:
- 233‑tick bars, indicators, regime states
- overlay outputs
- trade decisions and order intents

### 3.2 Replay is a first-class feature
- Archive raw Databento ticks from day 1 and internal events.
- Any production day MUST be replayable end-to-end with checksums for “same inputs → same outputs.”

### 3.3 Time discipline
All events MUST carry `event_ts` (source), `ingest_ts`, and `process_ts`. No service may substitute “now” during replay.

## 4) Separation of Concerns (No Cross-Contamination)
- Trading behavior logic MUST be isolated from infrastructure changes.
- Infrastructure changes MUST NOT change strategy outcomes except through explicitly versioned inputs.
- Contracts/schemas are law; breaking changes require versioning and migration plan.

## 5) Safe AI Integration (LLM Governance)
- LLMs are **contextual filters** and **structured rationale generators** only. They are **advisory by default**: they may downgrade confidence, block trades, or request additional confirmations, but may not “force” a trade.
- LLMs are explicitly prohibited from: risk calculations, stop placement/modification, position sizing / capital allocation, and direct order placement.
- LLM outputs MUST be schema-validated, time-bounded, token-bounded, and versioned (prompt hash + model id).
- Failure mode: **FAIL‑CLOSED for trading** unless explicitly ratified.

## 6) Data Integrity & Fail-Closed Trading
The system MUST stand down if any required gate fails (feed gaps, Redis health, clock skew, stale calendars). A trade is permitted only when all required services are `OK`.

## 7) Security & Observability 
**Security posture**: No public inbound ports; least privilege; secrets never stored in repos.

Two planes of observability MUST exist:
1. **Trading correctness**: PnL, drawdown, exposure, circuit breaker, decisions.
2. **System health**: ingest rates, reconnects, Redis health, vLLM queue.

**Auditability**: All decisions MUST log reason codes and the minimum sufficient state (bar id, indicator/regime versions, risk state, LLM hashes if used).
All critical safety events MUST alert via configured channels.

## 8) Technical Standards
- **Timeframes**: Primary execution relies on 233-tick bars with 1-minute confirmation bars.
- **Latency**: The system is latency-sensitive but not HFT. LLM filter TTFT should target p50 <= 100ms.
- **Deployment**: Local Windows host with WSL2 Ubuntu handling bare-metal LLM inference via vLLM, and Docker Compose for microservices. Cloud deployment should be vendor-agnostic (Terraform/containerized).

## 9) Futures Session & VWAP Canonical Policy
- VWAP and deviations are computed on **ETH+RTH** daily session aligned to the futures trading day rollover at **6:00pm ET (Sun–Fri)**.
- Any alternate VWAP anchors must be explicit, versioned, and tested.

## 10) Release Discipline (What “Shippable” Means)
A release to live requires unit/integration tests green, replay equivalence, risk tests, schema checks, rollback plan, and decision log entry.

## 11) Governance & Ratification
- This Constitution supersedes all other specs and unifies standards.
- Any change to core principles, risk invariants, safety rules, or LLM governance requires **version bump** and **re-ratification**.
- All PRs MUST explicitly declare constitutional impact.

**Version**: 1.1.0 (consolidated)  
**Ratified**: TODO(RATIFICATION_DATE)  
**Last Amended**: 2026-03-05
