---
description: Read a paper, submit for Plannotator review, then build report.md and wiki updates
argument-hint: "[arxiv:id | url | path]"
---

# Read

Paper-reading workflow. Accepts an arXiv ID, URL, or local file path.

## Workflow

1. Load the paper.
2. Draft the report as markdown **in memory**. Do NOT write a file.
3. Submit via `submit_plan` — this opens the Plannotator review UI.
4. Multi-round: user annotates → Request Changes → agent edits → resubmit. Repeat until Approve.
5. On approve, in the same turn immediately write `report.md`, ingest into wiki, and `build graph`. Ignore "Proceed with implementation".

## Report Requirements

- Keep claims grounded in the source paper.
- Distinguish summary, critique, and open questions.
- Include key numbers, settings, and limitations when available.
- Keep the report concise enough for review.
- Do not add redundant intermediate artifacts; keep the review loop in `submit_plan`.

## Wiki Requirements

- Save the finalized reading output as a stable `report.md`.
- Derive wiki notes from the report, not from memory.
- Add or update source, concept, and entity pages when useful.
- Preserve links so future reads can reuse the accumulated wiki.

## Behavior

- If the input is ambiguous, ask a short clarification question.
- If the source is missing, request the paper path or arXiv ID.
- If a custom SOP is provided, follow that structure before using any default template.
