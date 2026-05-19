# Agent Team Orchestrator Configuration Schema

Configuration is read from `.claude/ecf_config.yaml` under the `agent_team_orchestrator` key. Environment variables override YAML settings.

## Schema

```yaml
agent_team_orchestrator:
  # Whether the orchestrator is enabled.
  # When false, brainstorming/code_review fall back to direct skill invocation.
  # Default: true
  enabled: true

  # List of model IDs to use for Phase 1 agents.
  # Number of models determines number of agents (N models = N agents).
  # Valid values: opus, sonnet, haiku
  # Default: [opus, sonnet, haiku]
  models:
    - opus
    - sonnet
    - haiku

  # Number of cross-calibration rounds in Phase 2.
  # 1 = single round of cross-review (default, sufficient for most cases)
  # 2 = agents review responses to cross-review, deeper but higher cost
  # Default: 1
  cross_calibration_rounds: 1

  # Phase 2 cross-calibration strategy.
  # "distributed" (default) - each agent reviews all others' output
  # "centralized" - single synthesizer agent compares all outputs (faster, less detailed)
  # Default: "distributed"
  cross_calibration_strategy: "distributed"

  # Phase 3 consensus convergence strategy.
  # "centralized" (default) - orchestrator synthesizes all outputs into consensus report in the main context
  # "decentralized_debate" - all agents participate in multi-round debate to converge on consensus collectively
  # Default: "centralized" (backward compatible)
  consensus_strategy: "centralized"

  # Maximum debate rounds for decentralized_debate strategy.
  # Stops after this many rounds even if some disagreements remain.
  # Default: 2
  consensus_max_rounds: 2

  # Stop early if all disagreements are resolved before reaching consensus_max_rounds.
  # Only applies to decentralized_debate strategy.
  # Default: true
  consensus_early_stop: true

  # Enable output of all phase artifacts to markdown files for human review.
  # When enabled, each phase writes full output to a separate markdown file.
  # Default: true
  enable_file_output: true

  # Base output directory for phase artifact files.
  # Each run creates a subdirectory `{output_dir}/{mode}-{timestamp}/` containing:
  # - phase-1-independent-thinking.md
  # - phase-2-cross-calibration.md
  # - phase-3-consensus-report.md
  # Default: ".agent-team-output"
  output_dir: ".agent-team-output"
```

## Environment Variables

| Variable | Format | Example | Overrides |
|----------|--------|---------|-----------|
| `ATO_ENABLED` | `true`/`false` | `ATO_ENABLED=false` | `enabled` |
| `ATO_MODELS` | comma-separated | `ATO_MODELS=opus,sonnet` | `models` |
| `ATO_CROSS_CALIBRATION_ROUNDS` | integer | `ATO_CROSS_CALIBRATION_ROUNDS=2` | `cross_calibration_rounds` |
| `ATO_CONSENSUS_STRATEGY` | `centralized`/`decentralized_debate` | `ATO_CONSENSUS_STRATEGY=decentralized_debate` | `consensus_strategy` |
| `ATO_CONSENSUS_MAX_ROUNDS` | integer | `ATO_CONSENSUS_MAX_ROUNDS=3` | `consensus_max_rounds` |
| `ATO_CONSENSUS_EARLY_STOP` | `true`/`false` | `ATO_CONSENSUS_EARLY_STOP=false` | `consensus_early_stop` |
| `ATO_ENABLE_FILE_OUTPUT` | `true`/`false` | `ATO_ENABLE_FILE_OUTPUT=false` | `enable_file_output` |
| `ATO_OUTPUT_DIR` | path | `ATO_OUTPUT_DIR=/tmp/agent-output` | `output_dir` |

## Precedence

1. Environment variables (highest)
2. Project config: `.claude/ecf_config.yaml`
3. Global config: `~/.claude/skills/ecf/ecf_config.yaml`
4. Built-in defaults (lowest)

## Single-Invocation Override

Users can skip the orchestrator for a single request by including one of these in their prompt:
- `--no-orchestrator`
- `快速模式`
- `跳过编排`

This is NOT a config setting — it's detected from the user's input text.
