# Plannotator (OpenCode AI Fix)

> Forked from [backnotprop/plannotator](https://github.com/backnotprop/plannotator).  
> Fix: **Ask AI in OpenCode now reuses the existing client instead of spawning a new server.**

## What This Fixes

When using Plannotator's "Ask AI" feature inside the OpenCode desktop app, clicking "Ask AI" would show a **"Session error"**. This happened because the AI runtime (`createAIRuntime`) created a brand new `OpenCodeProvider` that independently tried to attach to the OpenCode server, instead of reusing the already-authenticated client from the embedded server.

### Root Cause

In `packages/server/ai-runtime.ts`, `createAIRuntime()` called `createProvider({ type: "opencode-sdk", cwd })` without passing the existing `opencodeClient`. The new `OpenCodeProvider` then tried to spawn its own `opencode serve` process or attach to the server independently — which failed in the desktop app's session/auth model.

### Fix (3 files changed)

| File | Change |
|------|--------|
| `packages/ai/types.ts` | Added optional `client` field to `OpenCodeConfig` |
| `packages/ai/providers/opencode-sdk.ts` | Constructor accepts pre-existing client; `ensureServer` skips if client provided |
| `packages/server/ai-runtime.ts` | `createAIRuntime` accepts `opencodeClient` option and passes it to the provider |
| `packages/server/index.ts` | `startPlannotatorServer` passes `options.opencodeClient` to `createAIRuntime` |

## Features

Plannotator is a local, browser-based review surface for AI coding agents. It supports:

- **Plan Review** — interactive annotation UI with approve/request-changes flow
- **Code Review** — review diffs and PRs with inline comments
- **HTML Artifact Review** — render and annotate HTML pages
- **Ask AI** — ask questions about the content being reviewed (now works with OpenCode!)
- **Multi-round feedback** — annotations flow back to the agent for iterative refinement

## Supported Agents

- OpenCode
- Claude Code
- Codex
- Copilot CLI
- Gemini CLI
- Kiro
- Droid
- Amp
- Pi

## Installation

### OpenCode

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    ["@plannotator/opencode@latest", {
      "workflow": "plan-agent",
      "planningAgents": ["plan", "read"]
    }]
  ]
}
```

Then restart OpenCode.

### CLI

```bash
curl -fsSL https://plannotator.ai/install.sh | bash
```

## Paper Reading Workflow (with OpenCode)

This fork is designed to work with a paper-reading workflow in OpenCode:

1. Download/load a paper (arXiv ID, URL, or local file)
2. Draft a report in memory
3. Call `submit_plan` — opens Plannotator review UI
4. Multi-round annotation and feedback
5. Approve → write `report.md` → wiki ingest → knowledge graph update

## License

MIT — same as upstream [backnotprop/plannotator](https://github.com/backnotprop/plannotator).