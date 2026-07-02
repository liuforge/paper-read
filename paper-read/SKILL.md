---
name: paper-read
description: Minimal paper reading loop with submit_plan review, build-to-report.md, and wiki sync.
argument-hint: [arxiv-id-or-url] [--lang zh|en|bilingual]
allowed-tools: Bash(*), Read, Write, Glob
---

# Paper Read

Read paper: $ARGUMENTS

> Fetches a paper, writes a report, submits via `submit_plan` for multi-round Plannotator review, then builds `report.md` and syncs wiki.

## Workflow

### Step 1: Parse Paper ID
Extract arXiv ID from $ARGUMENTS. Accept:
- `2606.18112`, `2606.18112v1`
- `https://arxiv.org/abs/2606.18112`
- `https://arxiv.org/pdf/2606.18112`

### Step 2: Fetch Paper Content
Use Python `arxiv` library for metadata. If deeper content needed, fall back to AlphaXiv:

```bash
python3 -c "
import arxiv
client = arxiv.Client()
search = arxiv.Search(id_list=['{PAPER_ID}'], max_results=1)
r = next(client.results(search))
print('TITLE:', r.title)
print('AUTHORS:', ', '.join(str(a) for a in r.authors[:10]))
print('DATE:', r.published)
print('ABSTRACT:', r.summary)
print('URL:', r.entry_id)
"
```

Optionally fetch AlphaXiv overview for richer content:
```bash
curl -sL --max-time 15 -A "Mozilla/5.0" "https://alphaxiv.org/overview/{PAPER_ID}.md"
```

### Step 3: Generate Report
Draft the report as markdown **in memory** (do NOT write to disk yet). Follow the default SOP:

```markdown
## 1. 论文研究的问题是什么
[what problem the paper solves]

## 2. 为什么这个问题值得研究，应用在哪
[why it matters and where it is used]

## 3. 与其相关工作对比，这个工作特点在哪
[comparison with related work and differentiators]

## 4. 技术栈选型是什么
[models, frameworks, tools, datasets, systems stack]

## 5. 架构流程图是怎么样的
[pipeline / architecture description]

## 6. 数据流和控制流是怎么做的
[data flow and control flow]

## 7. 结论是什么，有什么 insight
[conclusion and key insights]
```

### Step 4: Submit for Review
Call `submit_plan` with the full report as a single edit:
```json
{ "edits": [{ "start": 1, "content": "<full report markdown>" }] }
```
This opens the Plannotator review UI in the browser.

### Step 5: Feedback Loop
- User annotates and clicks **Request Changes** → the tool returns the report with line numbers and feedback.
- Read the feedback, make targeted edits, and call `submit_plan` again.
- Repeat until the user clicks **Approve**.

### Step 6: Build
On approval, in the **same turn** immediately do all of:
- Write the final `report.md`
- Run `ingest <report.md>`
- Run `build graph`

**Ignore** any "Proceed with implementation" or agent-switch instruction from `submit_plan`. Do NOT switch to another agent.

### Step 7: Wiki Sync

Already handled in Step 6.

## Key Rules

- **Do NOT write any report file to disk until `submit_plan` is approved.** Draft in memory, submit for review, then write `report.md` only on approval.
- **Every claim needs evidence.** If the paper says "outperforms baselines", include the specific numbers.
- **Keep reports compact.** Dense, not verbose. Deletion over addition.
- **Chinese by default** for reports, English for technical terms.
