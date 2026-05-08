# EasyCodingFlow

A unified agent orchestration skill that integrates Superpowers, OpenSpec, and Compound Engineering into a four-layer architecture.

## Overview

EasyCodingFlow is a Claude Code orchestration skill that coordinates multiple development frameworks for workflow orchestration based on user usage scenarios:

| Framework | Layer | Purpose |
|-----------|-------|---------|
| **OpenSpec** | Specification Contract Layer | Requirement Specification, Change Management |
| **Superpowers** | Executive Level | TDD, Brainstorming, Plan Execution |
| **Compound Engineering** | Knowledge Accumulation Layer | Solution Archiving, Knowledge Reuse |

**Core principle**: Each user request must first undergo intent recognition before being routed to the corresponding workflow

## Quick Start

### Prerequisites

- [Claude Code CLI](https://github.com/anthropics/claude-code) - AI coding assistant
- Git - Version control

### Installation

**1. Clone the project**

```bash
git clone https://gitcode.com/xuguoliang3/EasyCodingFlow.git
cd ecf
```

**2. Install skills**

```bash
# Copy to Claude Code skills directory
mkdir -p ~/.claude/skills

cp -r skills/ecf ~/.claude/skills/
cp -r skills/ecf-init ~/.claude/skills/
cp -r skills/ecf-execute ~/.claude/skills/
cp -r skills/ecf-verify ~/.claude/skills/
```

**3. Initialize your project**

Run in your project directory:

```bash
/ecf-init
```

This creates:
- `docs/solutions/` - Knowledge base
- `docs/architectures/` - Architecture docs
- `.claude/ecf_config.yaml` - Project config

### Verify Installation

```bash
ls ~/.claude/skills/
# Should show: ecf/ ecf-init/ ecf-execute/ ecf-verify/
```

## How It Works

### Four-Layer Architecture

```
┌─────────────────────────────────────────┐
│ Layer 0: Orchestration                   │
│ Intent → Route → Team → Monitor          │
├─────────────────────────────────────────┤
│ Layer 1: Contract                        │
│ OpenSpec (propose → apply → archive)     │
│ Brainstorming                            │
├─────────────────────────────────────────┤
│ Layer 2: Execution                       │
│ Writing-Plans → Executing-Plans → Test   │
├─────────────────────────────────────────┤
│ Layer 2.5: Verification                  │
│ Consistency-Verification                 │
├─────────────────────────────────────────┤
│ Layer 3: Knowledge                       │
│ Compound Engineering                     │
└─────────────────────────────────────────┘
```

**Core Process**: Every request first goes through intent recognition, then routes to the corresponding workflow, executing layer by layer.

### Supported Scenarios

| Scenario | Keywords | Workflow |
|----------|----------|----------|
| New requirement development | Development, new feature, implementation, creation | OpenSpec → Brainstorming → Writing-Plans → Executing-Plans → **Verification → Archive** |
| Incremental development | Expansion, iteration, enhancement | OpenSpec → Executing-Plans → **Verification → Archive** |
| Skills Development | skill, skills, optimize skills, SKILL.md | OpenSpec → **skill-creator** → Skill-Quality-Verification → **Archive** → Compound |
| Bug Repair | bug, error, failure, repair | Systematic-Debugging → Fix → **Verification** → Compound |
| Code Refactoring | Refactoring and Optimizing Structure | Brainstorming → Writing-Plans → Executing-Plans → **Verification** |
| Code Review | review、审查 | Requesting-Review → Receiving-Review |
| Document Update | Documentation, Readme | Direct Execution |
| Use case completion | Testing, use case | Behavior-Driven-Development → **Verification** |
| Knowledge Search | Previous, Similar, Historical | Search docs/solutions/ |

## Features

### Core Features

| Feature | Description |
|---------|-------------|
| **Intent Recognition** | Auto-identify request type, confidence ≥0.7 routes directly |
| **Skill Detection** | Detect skill-related tasks, enforce TDD workflow |
| **Parallel Execution** | DAG scheduling, >6 tasks use Agent Team parallelism |
| **Consistency Verification** | spec↔design↔code↔tests four-layer consistency check |
| **Knowledge Compound** | Auto-archive solutions, support history retrieval |
| **Artifact Conversion** | Auto-convert OpenSpec artifacts to execution layer input |

### Key Skills

| Skill | Command | Function |
|-------|---------|----------|
| ecf | `/ecf` | Main entry, intent recognition & orchestration |
| ecf-init | `/ecf-init` | Initialize project structure |
| ecf-execute | `/ecf-execute` | Execute plans with concurrency |
| ecf-verify | `/ecf-verify` | Consistency verification |

## Usage

### Basic Usage

Describe your task directly to Claude, the skill auto-recognizes and routes:

```
User: "Develop a user authentication system"
Claude executes: Intent → OpenSpec → Brainstorming → Plans → Execute → Verify → Archive

User: "Fix the login page bug"
Claude executes: Debugging → Fix → Verify

User: "How did we handle authentication before?"
Claude executes: Knowledge search docs/solutions/
```

### Configuration

Config file: `.claude/ecf_config.yaml`

```yaml
defaults:
  language: en-US
  max_parallel_agents: 5

models:
  brainstorming: sonnet
  execution: sonnet
  review: opus
```

### Project Structure

```
your-project/
├── docs/
│   ├── solutions/           # Knowledge base (required)
│   ├── architectures/       # Architecture docs
│   └── plans/               # Design plans
├── openspec/                # Managed by OpenSpec CLI
│   ├── changes/             # Change proposals
│   └ changes/archive/       # Archived changes
└── .claude/
    └── ecf_config.yaml      # Project config
```

## Contributing

Contributions are welcome! Feel free to submit code, report issues, or suggest features.

### Development Process

1. Fork the project
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'feat: add amazing feature'`)
4. Push branch (`git push origin feature/amazing-feature`)
5. Create Pull Request

### Skill Modification Guidelines

Skill modifications must follow TDD workflow:

```
/opsx:propose → skill-creator → skill-quality-verification → /opsx:archive → ce:compound
```

**Do NOT** directly edit SKILL.md files.

See: [Skills Development Workflow Guidelines](docs/solutions/workflow-issues/skills-development-workflow-enforcement-2026-05-04.md)

## Special Thanks

[OpenSpec](https://github.com/Fission-AI/OpenSpec)
[superpowers@frad-dotclaude](https://github.com/FradSer/dotclaude)
[compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)

## License

MIT License

---

**Note**: This skill requires Claude Code CLI to function.