# Agents

A roster of specialist subagents for AI coding assistants at SunCulture. Each
agent is a self-contained `.md` file with YAML frontmatter (`name`,
`description`, `model`) followed by its operating instructions. Agents follow
the [Claude Code subagent](https://docs.claude.com/en/docs/claude-code/sub-agents)
format and are auto-discovered when placed in an agent directory — the
`description` field drives delegation.

Each agent is named for the **specialty it handles** rather than a codename, so
the right specialist is obvious at a glance.

## Roster

Agents are grouped by where they sit in the delivery lifecycle.

### Product & design

| Agent | Specialty |
| --- | --- |
| [`product-strategy`](product-strategy.md) | Product vision, PRDs, OKRs, North-Star metrics, prioritisation (RICE/WSJF/Kano), discovery, MVP scoping, GTM. |
| [`product-design`](product-design.md) | UX/UI, information architecture, design systems, accessibility (WCAG 2.2 AA), prototyping, usability testing. |
| [`solutions-architect`](solutions-architect.md) | End-to-end system design, HLD/LLD, trade-off analysis, DDD, integration patterns, multi-region & DR. |

### Build

| Agent | Specialty |
| --- | --- |
| [`fullstack-engineer`](fullstack-engineer.md) | Principal-level fullstack build, refactor, and debug across Go, Python, .NET, TypeScript, mobile, and infra. |
| [`data-engineering`](data-engineering.md) | ETL/ELT, warehouses & lakehouses, streaming, CDC, data quality, MLOps, vector DBs, RAG. |
| [`machine-learning`](machine-learning.md) | Model selection, training, fine-tuning (LoRA/QLoRA/RLHF), evaluation, quantisation, governance. |

### Infrastructure & delivery

| Agent | Specialty |
| --- | --- |
| [`infrastructure-as-code`](infrastructure-as-code.md) | Multi-cloud Terraform / OpenTofu, state & repo layout, IAM/VPC/KMS posture, drift detection. |
| [`ci-cd-pipelines`](ci-cd-pipelines.md) | CI/CD pipelines, image builds & signing, release management, deployment strategies, GitOps. |
| [`edge-networking`](edge-networking.md) | CDN, edge compute, DNS, TLS/mTLS, global load balancing, WAF/DDoS, Core Web Vitals. |
| [`cloud-finops`](cloud-finops.md) | Cloud cost engineering, FinOps framework, RIs/Savings Plans, unit economics, GreenOps. |

### Quality, reliability & operations

| Agent | Specialty |
| --- | --- |
| [`security-architecture`](security-architecture.md) | Threat modelling (STRIDE/PASTA/LINDDUN), secure design, authn/authz, applied crypto, compliance. |
| [`quality-assurance`](quality-assurance.md) | Test strategy, the test pyramid, coverage thresholds, defect lifecycle, exit-criteria gating. |
| [`code-reviewer`](code-reviewer.md) | Brutal line-by-line code review across correctness, architecture, performance, security, and ops. |
| [`site-reliability`](site-reliability.md) | SRE, observability, SLIs/SLOs, error budgets, incident command, blameless postmortems, DORA. |

### Communication & coordination

| Agent | Specialty |
| --- | --- |
| [`technical-writing`](technical-writing.md) | ADRs, READMEs, RFCs, API docs, release notes, changelogs, diagrams, stakeholder comms. |
| [`project-management`](project-management.md) | Work breakdown (WBS), epics/stories, estimation, sprint sequencing, RAID logs, RACI, status reports. |
| [`customer-success`](customer-success.md) | Customer success & support ops, SLAs, health scoring, churn/retention, voice-of-customer. |
| [`agent-orchestrator`](agent-orchestrator.md) | Meta-orchestrator and routing across the roster — delegation plans, handoffs, conflict resolution. |

## Naming

Files were renamed from internal codenames to their handled specialty:

| Specialty file | Original codename |
| --- | --- |
| `infrastructure-as-code` | anvil |
| `solutions-architect` | cartwright |
| `cloud-finops` | coinsmith |
| `data-engineering` | datasmith |
| `product-design` | designsmith |
| `edge-networking` | edgesmith |
| `agent-orchestrator` | foreman |
| `security-architecture` | locksmith |
| `technical-writing` | loresmith |
| `machine-learning` | modelsmith |
| `ci-cd-pipelines` | pipesmith |
| `site-reliability` | probesmith |
| `customer-success` | pulsesmith |
| `product-strategy` | roadsmith |
| `fullstack-engineer` | stacksmith |
| `project-management` | tasksmith |
| `quality-assurance` | touchstone |
| `code-reviewer` | whetstone |

## Installation

Copy an agent into your assistant's agent directory (e.g. `~/.claude/agents/`):

```bash
cp agents/fullstack-engineer.md ~/.claude/agents/
```

Claude Code auto-discovers agents in `~/.claude/agents/` (user scope) or
`.claude/agents/` (project scope) and routes to them based on the `description`.

## Notes

- Agents reference a persistent memory directory at
  `~/.claude/agent-memory/<agent-name>/`. The path is home-relative and portable.
- Several agents auto-load the [`fullstack-engineering`](../skills/engineering/fullstack-engineering)
  skill for stack conventions.
