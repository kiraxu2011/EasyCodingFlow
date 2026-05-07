# EasyCodingFlow

A unified agent orchestration skill that integrates Superpowers, OpenSpec, and Compound Engineering into a four-layer architecture.

## Quick Start

### Prerequisites

- [Claude Code CLI](https://github.com/anthropics/claude-code) - AI coding assistant
- Git - Version control

### Installation

**1. Clone the project**

```bash
git clone https://github.com/kiraxu2011/EasyCodingFlow
cd EasyCodingFlow
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
| New Feature | develop, new feature, implement | OpenSpec → Brainstorming → Plans → Execute → Verify → Archive |
| Incremental | extend, iterate | OpenSpec → Execute → Verify → Archive |
| Bug Fix | bug, error, fix | Debugging → Fix → Verify |
| Refactor | refactor, optimize | Brainstorming → Plans → Execute → Verify |
| Code Review | review, audit | Review → Compound |
| Documentation | doc, readme | Direct Execution |
| Knowledge Search | previous, similar | Search docs/solutions/ |

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

## License

MIT License

Copyright (c) 2026 xuguoliang3

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

**Note**: This skill requires Claude Code CLI to function.
