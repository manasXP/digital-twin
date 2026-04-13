# Agent Orchestration for the Digital Twin

> **Status: DRAFT scouting note.** Surveys candidate **agent orchestration layers** for standing up the agent population inside the [[concepts/Digital Twin Transformation|digital twin]]. Not an endorsement of any one option — a framework for choosing, with a first-pass comparison across Paperclip, CrewAI, AutoGen, LangGraph, Temporal-style durable workflow orchestrators, and a bespoke build.

## Why this note exists

The twin thesis needs a population of role-shaped agents — PM, BA, dev, QA, reviewer, delivery lead — that can shadow-run real project history and, later, cross over into live work. The [[concepts/Cognee Knowledge Engine for the Digital Twin|knowledge engine]] gives those agents memory and grounding, but it does not give us the **org scaffolding**: who reports to whom, who is allowed to spend what, who can hire or retire an agent, and how delegated work is traced end-to-end.

That scaffolding is the **agent orchestration layer**. There are now several plausible choices, each with a different centre of gravity. This note defines what the twin actually needs from that layer, then scores the leading candidates against those needs.

## 1. What the twin needs from an orchestration layer

Independent of vendor, the twin's requirements are unusually opinionated:

1. **Org-shaped execution, not agent-shaped execution.** Work must flow through seats and reporting lines, not a single supervisor loop. Handoffs between roles are the primary unit of behaviour to measure.
2. **Seats vs. occupants.** A "dev seat" is a durable construct; the occupant (human, frontier model, cheap model, future replacement) rotates. Longitudinal twin analysis depends on seat-level history, not just agent-level history.
3. **Ticket-grained traceability.** Every delegated unit of work must carry a full audit trail — prompts, tool calls, intermediate decisions, citations back into the knowledge engine — so shadow runs can be compared against ground truth after the fact.
4. **Hard cost envelopes.** Per-agent, per-ticket, and per-run budget caps are a **contractual** requirement for the SMB offer in [[Business Model — SMB Services Transformation Offer]]. "LLM spend risk" must be a bounded quote, not a blank cheque.
5. **Multi-tenant isolation.** One deployment should host many client shadow-runs side by side without leakage. Twin-as-a-service assumes this.
6. **Human-in-the-loop gates.** Crossover from shadow to live must require explicit, multi-signer approval (delivery lead *and* client sponsor), timestamped, with a stored rationale. Demotion must be equally clean.
7. **Runtime heterogeneity.** Mixing frontier models for high-stakes seats (reviewer) with cheap models for commodity seats (BA, QA) is a margin requirement, not a nice-to-have.
8. **Deterministic-ish replay.** Before/after comparisons against a fixed historical baseline need some story for seeded reproducibility, or at minimum statistical replay.
9. **Pause / resume of an entire run.** Freezing a shadow run mid-stream for review without losing state is routine, not exceptional.
10. **Knowledge-engine integration surface.** Every agent must read/write the Cognee graph with citation and provenance through a single, swappable adapter.

These ten criteria become the scoring rubric.

## 2. Candidate orchestration layers

The market roughly clusters into four shapes:

- **Org-as-software orchestrators.** [Paperclip](https://paperclip.ing/) is the clearest example: agents-as-employees, org charts, tickets, board governance, per-agent budgets, multi-company isolation. MIT-licensed, self-hosted Node + Postgres, runtime-agnostic workers (Claude, Cursor, OpenClaw, HTTP webhooks).
- **Crew / role-based frameworks.** [CrewAI](https://www.crewai.com/), [AutoGen](https://microsoft.github.io/autogen/), and similar. Agents have roles and goals; crews coordinate on tasks. Lightweight, Python-first, strong community, weaker governance and budget primitives.
- **Graph / supervisor frameworks.** [LangGraph](https://langchain-ai.github.io/langgraph/), supervisor/worker patterns. Models agent coordination as a state machine or graph. Very flexible, low opinionation — everything above the graph (org chart, budgets, audit) is yours to build.
- **Durable workflow orchestrators repurposed for agents.** [Temporal](https://temporal.io/), Inngest, Restate. Not agent frameworks; they give you durable execution, retries, deterministic replay, and pause/resume — on top of which you hand-roll the agent layer.
- **Bespoke build.** A thin orchestrator written directly against the knowledge engine and model APIs. Maximum fit, maximum maintenance.

## 3. Comparison against twin criteria

Scoring is a first-pass judgement, not a benchmark: ●●● strong fit, ●● partial, ● weak, — absent. Expect this table to move as spikes land.

| Criterion                            | Paperclip | CrewAI / AutoGen | LangGraph | Temporal + custom | Bespoke |
|--------------------------------------|:---------:|:----------------:|:---------:|:-----------------:|:-------:|
| 1. Org-shaped execution              |    ●●●    |        ●●        |     ●     |         ●         |   ●●●   |
| 2. Seats vs. occupants               |    ●●     |         ●        |     ●     |        ●●         |   ●●●   |
| 3. Ticket-grained traceability       |    ●●●    |         ●        |    ●●     |        ●●●        |   ●●    |
| 4. Hard cost envelopes               |    ●●●    |         —        |     —     |         ●         |   ●●    |
| 5. Multi-tenant isolation            |    ●●●    |         ●        |     ●     |        ●●         |   ●●    |
| 6. Human-in-the-loop gates           |    ●●     |         ●        |    ●●     |        ●●●        |   ●●    |
| 7. Runtime heterogeneity             |    ●●●    |        ●●        |    ●●●    |        ●●●        |   ●●●   |
| 8. Deterministic-ish replay          |     ●     |         —        |     ●     |        ●●●        |   ●●    |
| 9. Pause / resume whole run          |    ●●     |         ●        |    ●●     |        ●●●        |   ●●    |
| 10. Knowledge-engine adapter surface |    ●●     |        ●●        |    ●●●    |        ●●         |   ●●●   |

**Headline reading of the table:**

- **Paperclip** is the only option that treats *org, budgets, and multi-tenancy as first-class primitives*. That is exactly the shape of the twin's hardest requirements (1, 4, 5). Its weaknesses are replay determinism and project maturity.
- **CrewAI / AutoGen** get to a demo fastest, but leave every governance concern (budgets, audit, multi-tenant, gates) as homework. Good for an internal prototype, risky for a client engagement.
- **LangGraph** is the best neutral substrate if the team wants to own the org model end-to-end. Strong on integration surface and flexibility; weak on everything governance-shaped, which would then need to be built.
- **Temporal-style + custom** is the strongest story for **replay, pause/resume, and durability** — which matter for shadow-run reproducibility — but it is not an agent framework, and you pay for the missing agent abstractions.
- **Bespoke** wins on fit but loses on every month of maintenance. Only defensible if none of the above clears the bar after spikes.

## 4. How the choice maps into the transformation arc

- **Phase 1 — As-Is twin build.** Orchestration layer is not needed. The twin is still passive: ingest history into the knowledge engine, reconstruct project state.
- **Phase 2 — Shadow runs.** Strongest differentiation between options. Each shadow project becomes a "company" / tenant / workflow; the historical team is re-cast as seats; tickets replay the historical backlog. Budget caps and audit trails earn their keep here.
- **Phase 3 — Controlled crossover.** Human-in-the-loop gates and clean demote paths dominate the choice. Whichever layer makes multi-signer approval cheap wins points.
- **Phase 4 — AI-Native steady state.** The orchestration layer becomes the delivery org's actual runtime, not a scaffold. At that point, switching cost is prohibitive, so the Phase 2/3 choice is effectively load-bearing for years.

## 5. Cross-cutting risks

These apply regardless of which layer is chosen and should shape the spike design.

- **Seat-vs-occupant modelling.** Most frameworks conflate agents and roles. The twin depends on keeping them separate. Whichever option is adopted, the seat abstraction may have to live in **our** adapter, not in the framework.
- **Knowledge-engine integration surface.** Every candidate expects you to bring your own memory. The Cognee adapter should be a single, swappable module owned by us, not scattered through agent prompts.
- **Governance semantics.** "Approval" primitives are often single-flag, single-signer. Crossover gates in the twin are multi-signer with stored rationale. Expect to wrap whatever the framework provides.
- **Cost attribution granularity.** Per-agent cost is common; **per-ticket, end-to-end cost across all agents that touched a user story** is not. This is the unit economics the SMB offer needs, and it may require custom instrumentation.
- **Deterministic replay.** No agent framework gives this for free. It has to come either from a durable workflow layer underneath, or from disciplined seeding and fixture management.
- **Operational burden.** Every self-hosted option (Paperclip, Temporal, bespoke) carries on-call and data-handling obligations. For a boutique delivery model, that burden must be priced in or partnered out.
- **Lock-in by convention.** Even with permissive licenses, adopting a framework's vocabulary (crews, graphs, companies) shapes how the twin's data model evolves. Be deliberate rather than drifting.

## 6. Proposed next steps

The goal of the spikes below is to produce enough evidence to make a **documented adopt / wrap / pass** decision, not to pick a favourite on paper.

1. **Spike A — Single-role shadow replay on the leading candidate.** Based on the table above, Paperclip is the current front-runner for the Phase 2 requirements. Stand it up locally, register one Claude-backed agent as a "BA" seat, replay a small historical backlog against a Cognee-backed knowledge engine. Measure: cost, trace completeness, time-to-first-useful-output.
2. **Spike B — Same scenario on LangGraph.** Build the minimum org scaffolding ourselves. This is the honest control: if LangGraph plus ~1 week of glue reaches parity on the twin criteria, the governance primitives we get "for free" from Paperclip are worth less than they look.
3. **Spike C — Two-role delegation.** On whichever of A or B is stronger, add a "dev" seat under the BA and verify ticket delegation, audit trail shape, and seat-vs-occupant fidelity. This is the minimum viable org.
4. **Integration adapter sketch.** Define the contract between agents and the Cognee graph — read, write, citation, provenance — as a single adapter module. It must be portable between candidate frameworks; that portability is a first-class design goal, not a side effect.
5. **Durability probe.** On the leading candidate, deliberately kill a run mid-ticket and restart. Observe what survives. If nothing useful does, reconsider whether a Temporal-style substrate has to sit underneath.
6. **Decision record.** Write a one-page ADR in this folder capturing the choice, the evidence, and the exit conditions under which the decision would be revisited. No client-facing work depends on the orchestration layer until that record exists.

## 7. Open questions to resolve before adoption

These apply to whichever candidate is chosen and should be answered in the spike write-ups.

- Does the framework represent **seats separately from occupants**, or are agents and roles conflated? If conflated, what does our adapter need to do to restore the distinction?
- Can the approval primitive express **multi-signer** gates (delivery lead *and* client sponsor), or only single-approver flows?
- Is there a clean hook for **pausing an entire run** (freezing shadow execution mid-stream for review) without losing ticket state?
- What is the pattern for **injecting ground-truth historical artefacts** (the original PR, the original ticket outcome) as hidden context that shadow agents must not see until after their decision?
- Can the framework surface **per-ticket cost** vs **per-agent cost** — i.e. answer "what did this user story cost us, end-to-end, across all agents that touched it"?
- Is there a sensible story for **deterministic replay** of a shadow run, or is every run non-deterministic by construction? Determinism matters for before/after comparisons against a fixed baseline.
- What is the **upgrade and contribution-back posture** if the chosen project is open-source? Who patches, who merges, who owns the fork if one is needed?
- What is the **operational model** — does Manas run the layer for clients (TaaS), or does the client self-host? The answer reshapes both the pricing in [[Business Model — SMB Services Transformation Offer]] and the liability surface.

## Revision History

| Date       | Author        | Change                                                                |
|------------|---------------|-----------------------------------------------------------------------|
| 2026-04-13 | Manas Pradhan | Initial scouting note on Paperclip as twin agent layer.               |
| 2026-04-13 | Manas Pradhan | Generalised to orchestration-layer-agnostic survey; added comparison. |
