# Plannotator — Paper Reading Mode for OpenCode

> Forked from [backnotprop/plannotator](https://github.com/backnotprop/plannotator).  
> **Turn Plannotator into an interactive paper-reading workflow for OpenCode.**

## What This Is

A modified Plannotator that replaces the plan-review flow with a **paper-reading workflow**. Given an arXiv ID, it downloads the paper, generates a structured report, opens it in Plannotator for multi-round annotation and feedback, then persists the final report and updates a knowledge graph.

## Fix: Ask AI Now Works with OpenCode

The original Plannotator's "Ask AI" showed a **"Session error"** in the OpenCode desktop app. The AI runtime created a new `OpenCodeProvider` instead of reusing the existing authenticated client. This fork fixes that.

### Changes (4 files)

| File | Change |
|------|--------|
| `packages/ai/types.ts` | Added optional `client` field to `OpenCodeConfig` |
| `packages/ai/providers/opencode-sdk.ts` | Constructor accepts pre-existing client |
| `packages/server/ai-runtime.ts` | `createAIRuntime` accepts `opencodeClient` option |
| `packages/server/index.ts` | Passes `opencodeClient` to `createAIRuntime` |

## Paper Reading Workflow

### Flow

1. **Download/load paper** — arXiv ID, URL, or local PDF/TXT
2. **Draft report in memory** — markdown, following the Report SOP (do NOT write to disk yet)
3. **Call `submit_plan`** — opens Plannotator review UI in browser
4. **Multi-round annotation** — annotate → Request Changes → agent edits → resubmit
5. **Approve** → write `report.md` → wiki ingest → `build graph`

### Report SOP

The default report structure follows these 7 items:

1. 论文研究的问题是什么
2. 为什么这个问题值得研究，应用在哪
3. 与其相关工作对比，这个工作特点在哪
4. 技术栈选型是什么
5. 架构流程图是怎么样的
6. 数据流和控制流是怎么做的
7. 结论是什么，有什么 insight

## Supported Agents

- **OpenCode** (only)

## Installation

### 1. Add the plugin to your OpenCode config

```jsonc
// .opencode/opencode.jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": {
    "paths": [".agents/skills"]
  },
  "plugin": [
    "opencode-chrome-devtools@latest",
    "@tarquinen/opencode-dcp@latest",
    ["@plannotator/opencode@latest", {
      "workflow": "plan-agent",
      "planningAgents": ["plan", "read"]
    }],
    "@mem0/opencode-plugin@latest"
  ]
}
```

### 2. Add the agent and command files

Copy `paper-read/` contents to your `.opencode/` directory:

```
.opencode/
├── agents/read.md
├── command/read.md
├── commands/read.md
├── opencode.jsonc
└── skills/paper-read/SKILL.md
```

### 3. Install the Plannotator CLI

```bash
curl -fsSL https://plannotator.ai/install.sh | bash
```

### 4. Restart OpenCode

## Usage

```
/read arxiv:2606.18112
```

Or switch to `read` mode and type the arXiv ID directly.

## License

MIT — same as upstream [backnotprop/plannotator](https://github.com/backnotprop/plannotator).