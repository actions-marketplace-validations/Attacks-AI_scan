# attacks.ai Agent Security Scan

Test your AI agent for OWASP LLM Top 10 vulnerabilities in CI/CD.

## Quick Start

```yaml
- uses: attacks-ai/scan@v1
  with:
    api-key: ${{ secrets.ATTACKS_API_KEY }}
    agent-command: python run_agent.py "$SCAN_URL"
```

## Usage

### Full pipeline (create scan + run agent + check results)

```yaml
name: Agent Security
on: [push]

jobs:
  security-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: attacks-ai/scan@v1
        id: scan
        with:
          api-key: ${{ secrets.ATTACKS_API_KEY }}
          agent-command: python test_agent.py "$SCAN_URL"
          provider: openai
          fail-on: fail
```

### Separate steps (more control)

```yaml
steps:
  # 1. Create scan
  - uses: attacks-ai/scan@v1
    id: scan
    with:
      api-key: ${{ secrets.ATTACKS_API_KEY }}

  # 2. Run your agent (your own step)
  - name: Run agent
    run: python my_agent.py "${{ steps.scan.outputs.scan-url }}"

  # 3. Check results (re-run action with same scan)
  - uses: attacks-ai/scan@v1
    with:
      api-key: ${{ secrets.ATTACKS_API_KEY }}
      agent-command: "true"  # no-op, agent already ran
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | Yes | | attacks.ai API key (`atk_...`) |
| `agent-command` | No | | Command to run your agent. `$SCAN_URL` is the target URL. |
| `provider` | No | | Provider hint (openai, anthropic, google, etc.) |
| `wait-timeout` | No | `300` | Max seconds to wait for completion |
| `fail-on` | No | `fail` | Gate threshold: `fail`, `warn`, or `any` |
| `max-injection` | No | | Max prompt-injection findings before the gate fails |
| `max-contextual` | No | | Max contextual-disclosure findings before the gate fails |
| `max-behavioral` | No | | Max behavioral findings before the gate fails |
| `max-categories` | No | | Max failed scoring categories before the gate fails |
| `fail-on-reaction` | No | | Fail if the agent's reaction is at or worse than this ladder class: `REFUSED` < `ESCAPE_VALVE` < `DETECTED_BUT_DISCLOSED` < `ENGAGED` < `ENDORSED`. `INSUFFICIENT_DATA` (browse-only, no scored rung) never trips it. |
| `fail-on-regression` | No | `false` | Opt-in reaction-ladder regression gate vs a `baseline-scan`. Fails only on a genuine regression (the agent got worse); never on `INSUFFICIENT_DATA`. |
| `baseline-scan` | No | | A prior run's `scan-id` to compare against for `fail-on-regression` (the same API key must own both scans). |

## Outputs

| Output | Description |
|--------|-------------|
| `scan-id` | Scan ID |
| `scan-url` | URL to send your agent to |
| `session-id` | Session ID (after completion) |
| `status` | `all-clear`, `moderate`, `high-risk`, or `critical` |
| `ci-gate` | `pass`, `warn`, or `fail` |
| `results-url` | Link to full security report |
| `findings` | Total findings count |
| `reaction` | agentReaction class: `REFUSED`, `ESCAPE_VALVE`, `DETECTED_BUT_DISCLOSED`, `ENGAGED`, `ENDORSED`, or `INSUFFICIENT_DATA` |

## Categories Tested

| Category | OWASP | What it tests |
|----------|-------|---------------|
| Prompt Injection | LLM01 | Agent follows hidden instructions in web content |
| Data Exfiltration | LLM06 | Agent leaks configuration or context |
| Excessive Agency | LLM08 | Agent takes unauthorized actions |
| Credential Confusion | LLM07 | Agent confuses examples with real credentials |

## Get an API Key

```bash
curl -X POST https://attacks.ai/api/v1/register \
  -H "Content-Type: application/json" \
  -d '{"email": "dev@example.com", "name": "My App"}'
```

Or sign up at [attacks.ai/register](https://attacks.ai/register).
