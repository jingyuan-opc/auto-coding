# Auto-Coding

Combining the structured specification power of [OpenSpec](https://openspec.dev) with the multi-agent orchestration of [Superpowers](https://github.com/astropub/superpowers), Auto-Coding provides a three-phase AI agent skill pipeline that takes an idea from brainstorming through reviewed implementation to archival — all within your AI coding assistant.

## Why

- **OpenSpec** gives you rigorous, versioned specifications — but doesn't orchestrate who builds what or review the result.
- **Superpowers** gives you powerful multi-agent workflows — but doesn't enforce a spec-first process.

Auto-Coding bridges both: every feature starts as an OpenSpec proposal, gets implemented by isolated subagents with two-stage review, then auto-archives with spec sync. The best of both worlds, wired together as three ready-to-use agent skills.

## SOC Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  /soc-spec  │───>│  /soc-build │───>│ /soc-archive │
│             │    │             │    │              │
│ Brainstorm  │    │ Implement   │    │ Archive      │
│ Design      │    │ Review x2   │    │ Sync specs   │
│ OpenSpec    │    │ Iterate     │    │ Clean up     │
└─────────────┘    └─────────────┘    └─────────────┘
```

### `/soc-spec` — Brainstorm to Proposal

Interactive dialogue that explores your project context, asks clarifying questions one at a time, proposes 2-3 approaches with trade-offs, then generates a complete OpenSpec proposal (proposal, design, tasks) and reviews it for completeness.

Optional **Visual Companion**: a browser-based tool for showing mockups and diagrams during brainstorming. Write HTML fragments, the user sees them in their browser and clicks to select options.

### `/soc-build` — Subagent-Driven Implementation

Each task from the OpenSpec proposal is dispatched to an isolated subagent with precisely crafted context. After every task, two independent reviews run:

1. **Spec compliance review** — verify the implementer built exactly what was requested (code is read directly, reports are not trusted)
2. **Code quality review** — verify the implementation is clean, secure, and tested

Tasks retry up to 2 times per review stage. Three consecutive failures pause the pipeline for human intervention.

### `/soc-archive` — Auto Archive

Triggered automatically after all tasks complete. Syncs delta specs, moves the change to the archive directory, and reports a summary.

## Installation

### Prerequisites

- [OpenSpec CLI](https://openspec.dev) installed globally


### Install the Skills

Copy or symlink the three skill directories into your project's skill directory:

```bash
git@github.com:jingyuan-opc/auto-coding.git
cd auto-coding

# For Claude Code — copy to your project's .claude/skills/ directory
cp -r soc-spec soc-build soc-archive /path/to/your-project/.claude/skills/

# Or symlink for easier updates
ln -s /path/to/auto-coding/soc-spec  /path/to/your-project/.claude/skills/soc-spec
ln -s /path/to/auto-coding/soc-build /path/to/your-project/.claude/skills/soc-build
ln -s /path/to/auto-coding/soc-archive /path/to/your-project/.claude/skills/soc-archive
```

Verify installation by checking the skill list in your AI tool — you should see `soc-spec`, `soc-build`, and `soc-archive`.

### Install the Collaboration Constitution (Optional)

The `CLAUDE.md` file provides a rigorous set of coding guidelines for your AI assistant — pre-flight checks, AI anti-patterns, arbitration priorities, and quality gates. Copy it to your project root:

```bash
cp CLAUDE.md /path/to/your-project/CLAUDE.md
```

## Usage

### Start a New Feature

```
> /soc-spec 我想实现一个xxx功能
```

The skill will explore your project, brainstorm with you, and generate an OpenSpec proposal. Review the generated artifacts, then build:

```
> /soc-build
```

The skill implements each task by supagent, reviews it twice, and auto-archives when all tasks pass:

```
> /soc-archive    # runs automatically after /soc-build
```


## Project Structure

```
auto-coding/
├── CLAUDE.md                          # Collaboration constitution (optional)
├── soc-spec/                          # /soc-spec skill
│   ├── SKILL.md                       # Skill definition
│   ├── visual-companion.md            # Browser-based visual brainstorming guide
│   ├── spec-reviewer-prompt.md        # Spec document reviewer prompt
│   └── scripts/                       # Visual companion server
│       ├── server.cjs
│       ├── helper.js
│       ├── frame-template.html
│       ├── start-server.sh
│       └── stop-server.sh
├── soc-build/                         # /soc-build skill
│   ├── SKILL.md                       # Skill definition
│   ├── implementer-prompt.md          # Implementer subagent prompt
│   ├── spec-reviewer-prompt.md        # Spec compliance reviewer prompt
│   └── code-quality-reviewer-prompt.md # Code quality reviewer prompt
└── soc-archive/                       # /soc-archive skill
    └── SKILL.md                       # Skill definition
```

## License

MIT
