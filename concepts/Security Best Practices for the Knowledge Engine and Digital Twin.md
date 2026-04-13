
---

# Security Best Practices for the Knowledge Engine and Digital Twin — A Design-Choice View

---

## Why Security Is a First-Order Design Constraint Here

The whole value of the [[Digital Twin Transformation|digital twin]] comes from the fact that it holds the org's full delivery context in one place. The [[Cognee Knowledge Engine for the Digital Twin|knowledge engine]] ingests "38+ data types — PDRs, PRs, tickets, emails, call transcripts, spreadsheets" from past client engagements and layers an ontology over them so agents can reason against real history rather than generic LLM priors. That is the same property that makes it the most sensitive asset the firm would ever stand up.

> The twin's value and its risk surface are the same surface. Anything that makes the graph more useful also makes a breach more consequential.

This note is not a control catalogue, not an ISO/SOC mapping, not a DPIA. It is the **design-choice rationale** — which security concerns the architecture already answers by construction, and which ones are deliberately left to policy and to the Governance & Quality function that owns "AI output review standards, IP/data policies, and audit" in the [[AI Native Delivery Transformation|AI-Native]] operating model.

The framing matters. Security bolted on after the fact tends to be a negotiation between feature velocity and control. Security that falls out of structural decisions already made for other reasons is cheap to hold and hard to accidentally erode.

---

## The Threat Model — What Can Actually Go Wrong

Before mapping controls, the concrete things one would worry about, grounded in the way the twin is actually built:

| # | Concern | What it looks like in practice |
|---|---|---|
| 1 | Client IP / source-code leakage via LLM provider | Code or design docs flow to a provider that retains or trains on inputs |
| 2 | PII / personal data in ingested artifacts | Names, emails, meeting transcripts, client employee details end up in the graph |
| 3 | Cross-client contamination | Agent serving engagement A retrieves context from engagement B |
| 4 | Prompt injection from ingested content | A malicious string in an old ticket or email hijacks a downstream agent |
| 5 | Replay of historical client work without consent | Past engagements are re-run in the twin without the client's knowledge or permission |
| 6 | Over-broad agent retrieval | One task pulls a far wider slice of the graph than it needs |
| 7 | Credential/token sprawl across ingestion connectors | Long-lived tokens for Jira, GitHub, Slack, email, CRM drift out of control |
| 8 | Audit trail gaps | A governance reviewer cannot trace why the twin produced a given answer |
| 9 | Model output exfiltration | Inputs silently used for provider-side training |
| 10 | Insider misuse | The twin becomes an unmonitored search engine over client data for internal curiosity |
| 11 | Malicious or unvetted agent extensions — skills, MCP servers, CLI tools | A skill, MCP server, or CLI tool pulled from a public registry or arbitrary URL exfiltrates graph data, rewrites prompts, shells out, or runs arbitrary code inside the twin's agent runtime |

This is the list the rest of the note maps against.

---

## How Each Concern Is Handled by Design Choice

The key claim: most of these are already answered by decisions the twin has to make anyway. The residual risk in every row is **narrower than the original concern** and is the slice Governance & Quality owns as policy.

| # | Concern | Design choice that addresses it | Residual risk → policy |
|---|---|---|---|
| 1 | LLM provider data leakage | Tool selection constraint. Only providers with no-training guarantees — Claude API, GitHub Copilot Business, Azure OpenAI with private deployment, or self-hosted Ollama + open-source code models for the most sensitive engagements. This is the path [[AI Native Delivery Transformation]] Challenge 4 already prescribes. | Per-engagement provider whitelist; provider attestation kept with the engagement record |
| 2 | PII in tickets, emails, transcripts | Redaction pass at the ingestion boundary, before anything is written to the graph. The ontology explicitly marks PII-bearing node types so later queries can honour a redaction policy. | Maintenance of redaction rules; periodic sampling review |
| 3 | Cross-client contamination | **One cognee graph per project** — the [[Cognee Knowledge Engine for the Digital Twin]] essay is explicit about this. Isolation is the default *shape* of the data, not a row-level filter that can be misconfigured. | Naming/tenancy convention enforced at ingestion time |
| 4 | Prompt injection from ingested content | All ingested artifacts are treated as untrusted input. LLM outputs never reach a delivery artefact unmediated: the spec-driven framework's **review checkpoints** and "security scanning on all AI-generated code" (AI-Native Definition of Done) sit between the model and anything that ships. | Untrusted-content quoting convention in prompt templates |
| 5 | Replay without client consent | The twin operates on **completed past projects only** ("existing org untouched… no live clients touched" — [[Digital Twin Transformation]] Layer 3). Consent is obtained as part of engagement closeout rather than retrofitted years later. | Closeout checklist amendment to capture replay consent |
| 6 | Over-broad agent retrieval | Cognee's **agentic isolation + context curation** — "each agent pulls only the slice of the graph relevant to its task." Agents are scoped at the ontology layer, not at query time. | Per-agent scope review during ontology design |
| 7 | Credential sprawl across connectors | Ingestion runs through a single service per source, with credentials held in a secrets manager, not scattered across notebooks or local configs. | Rotation cadence and owner per connector |
| 8 | Audit trail | Cognee's "one knowledge graph with vectors for precise, cited answers" gives every agent answer a traceable provenance back to graph nodes. This is the exact property the Governance & Quality function needs to review AI output. | Retention window and immutability policy for the audit store |
| 9 | Model output exfiltration via training | Same lever as concern 1 — provider choice. Handled by construction, not by hope. | Re-verification whenever provider terms change |
| 10 | Insider misuse | The twin is **physically separate** from the live org — the whole point of the "parallel, not in-line" pattern in [[Digital Twin Transformation]]. Access is a deliberate grant, not ambient because someone already has org credentials. | RBAC on the twin side; access logging reviewed by Governance & Quality |
| 11 | Malicious or unvetted agent extensions — skills, MCP servers, CLI tools | **The internal portal is the single distribution channel for all agent extensions — skills, MCP servers, and CLI tools alike.** Every entry is reviewed, signed, and version-pinned by Governance & Quality before publication. The agent runtime refuses to load a skill, connect to an MCP server, or invoke a CLI tool from any source other than the portal, so a drive-by install, a public-registry pull, or a typo-squat on a public name cannot reach the twin. Same rule, same pipeline, no carve-outs per extension type. | Portal review SLA; signing-key rotation; periodic re-review of long-lived entries across all three extension types |

A special case sits slightly outside this table:

**Controlled crossover (Layer 4).** The moment the twin stops being purely historical is the [[Digital Twin Transformation|L4]] step where "AI-Native process is primary; traditional process runs as shadow backup" on a first live engagement. The design choice that bounds the blast radius is that **the traditional process is still running in parallel** — the AI-Native side cannot silently fail into production because the baseline is producing the same artefacts alongside it. That is not zero risk, but it is a single chosen engagement under explicit client agreement, not a general-population rollout. The residual is per-engagement risk acceptance, signed before L4 begins.

---

## The Principle Behind the Choices

Read the table as a whole and the posture falls out of three structural decisions that were already made for entirely non-security reasons:

- **Parallel, not in-line.** The twin is a shadow system, separate from the live org. The blast radius of anything going wrong is bounded by construction — there is no production delivery path that a compromised agent can reach directly.
- **Per-project graphs, not a global lake.** Isolation is the default state of the data, not a permission that has to be remembered. The wrong answer to "can agent A see engagement B's data" is structurally impossible rather than policy-forbidden.
- **Spec-driven review checkpoints.** Untrusted LLM output never reaches a delivery artefact unmediated. The methodology's hard part — the review gates — is also the security boundary against prompt injection and hallucination.

Everything the architecture does not handle collapses into a shorter list: **provider whitelists, redaction rules, RBAC, retention, consent capture.** Those are policy and operations, and they are exactly the remit of the Governance & Quality function the AI-Native essay already puts in the operating model. The architecture closes the categories; Governance & Quality runs the controls inside them.

> If a security concern cannot be named against one of those three structural decisions or against Governance & Quality's standing remit, it is probably a new concern and deserves a new row in the table above — not a new tool.

---

## What This Note Is Not

- Not a control catalogue. It does not enumerate every technical safeguard; it explains which ones are load-bearing.
- Not a compliance mapping. ISO 27001, SOC 2, and client-specific addenda are downstream of this; they consume this rationale, they do not replace it.
- Not a data protection impact assessment. A DPIA for a specific engagement or client is a separate artefact that cites this note as its architectural baseline.
- Not a substitute for the methodology. As the Cognee essay warns about the knowledge engine itself: if the underlying methodology is weak, a tighter security wrapper just makes the wrong answers more confidently protected.

---

## Revision History

| Date | Author | Change |
|---|---|---|
| 2026-04-13 | Manas Pradhan | Initial version. |
| 2026-04-13 | Manas Pradhan | Added concern 11: agent skills, MCP servers, and CLI tools must be installed only from the internal portal to prevent malicious extensions from reaching the twin runtime. |
