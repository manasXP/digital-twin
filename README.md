# Digital Twin

A writing and research repository on **digital-twin-led organizational transformation** for IT services firms — specifically, how to adopt an AI-Native delivery model by building a shadow organization (a "digital twin"), validating it through replays of past projects, and only then crossing the new model over into live delivery.

The repo lives inside an Obsidian vault and mixes long-form essays, formal series documents, and diagrams.

## Contents

### Essays (vault root)

Narrative, opinionated pieces that set up the core arguments. Use wiki-links (`[[...]]`) between them — open them in Obsidian for proper cross-navigation.

| File | Purpose |
|---|---|
| `Digital Twin Transformation.md` | The central thesis — don't transform the existing org; build a digital twin, design the AI-Native model inside it, validate via shadow runs, cross over gradually. |
| `AI Native Delivery Transformation.md` | A practical guide to AI-Native delivery itself — spec-driven development, phased rollout, realistic ROI, common failure modes. This is the model the twin is used to design and validate. |
| `Cognee Knowledge Engine for the Digital Twin.md` | Why an agentic knowledge engine ([cognee](https://www.cognee.ai/)) belongs on the twin side — it is the substrate that lets shadow runs reason over replayed history with full fidelity. |

### Series documents (`docs/`)

Formal, structured blueprints written in a consistent "AI-Native Transformation Series" style (Arial cover, numbered H1 sections, styled tables). Each `.docx` is the source of truth; matching `.pdf` files are generated artifacts and are git-ignored.

| Document | Subject |
|---|---|
| `twin_modelling_architecture.docx` | Five-layer architecture for building the Organisational Digital Twin — data ingestion, process model, twin runtime, governance, implementation timeline. |
| `shadow_project_methodology.docx` | Methodology for replaying past projects through the AI-Native twin to produce hard comparative data before any live crossover. |
| `spec_driven_framework.docx` | Complete framework for writing structured specifications that serve as both human requirements and AI-pipeline inputs. |
| `skills_driven_delivery.docx` | Framework for defining, adopting, governing, and compounding AI delivery skills across the org. |
| `knowledge_engine_deployment.docx` | Deployment blueprint for operating cognee as the twin's retrieval and reasoning core — Docker Compose on a VPS, with one engine instance per client engagement for clean access control and isolation. |

### Diagrams (`diagrams/`)

Excalidraw source files. Open in [excalidraw.com](https://excalidraw.com) or Obsidian's Excalidraw plugin.

| File | Shows |
|---|---|
| `AI Native Delivery Transformation.excalidraw` | The four-phase rollout (Spec-Driven Framework → Pilot → Toolchain + Guardrails → Scale & Train) with core team, outcomes, and the core mental-model banner. |
| `Digital Twin Transformation.excalidraw` | Two parallel tracks (existing org untouched vs. shadow twin) with the four twin layers, the crossover arrow, and the four problems the twin approach uniquely solves. |

## Conventions

- **Revision history.** Every `.md` note and every `docs/*.docx` ends with a Revision History section (Date | Author | Change). New rows are appended only for **material** changes — typos and reformatting do not get a row.
- **PDFs are generated.** `docs/*.pdf` files are produced from the `.docx` sources via `textutil` → headless Chrome. They are listed in `.gitignore` and should not be checked in.
- **`CLAUDE.md` is local.** It exists to brief Claude Code on how to work in this repo and is git-ignored.
- **Paths contain spaces and a tilde** (`iCloud~md~obsidian/...`). Always quote the vault path in shell commands.

## Regenerating PDFs

From the repo root:

```bash
cd docs
for f in *.docx; do
  base="${f%.docx}"
  textutil -convert html "$f" -output "/tmp/${base}.html"
  "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
    --headless --disable-gpu --no-pdf-header-footer \
    --print-to-pdf="${base}.pdf" "file:///tmp/${base}.html"
done
```

## Revision History

| Date | Author | Change |
|---|---|---|
| 2026-04-12 | Manas Pradhan | Initial version. |
