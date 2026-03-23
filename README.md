# Paperclip Companies

Pre-built agent companies and shared skills for [Paperclip](https://github.com/paperclipai/paperclip). Each company directory contains a complete company configuration ready to deploy, and the top-level `skills/` directory holds reusable skills that can be shared across companies.

## Companies

### [GStack](./gstack)

An engineering company built around [Garry Tan's gstack workflow](https://github.com/garrytan/gstack). Work flows through distinct cognitive modes: founder taste, technical planning, paranoid review, fast execution, and systematic QA.

![GStack Org Chart](./gstack/images/org-chart.png)

**Agents:** CEO, CTO, Staff Engineer, Release Engineer, QA Engineer

### [Superpowers Dev Shop](./superpowers)

A disciplined development company built on [Jesse Vincent's Superpowers](https://github.com/obra/superpowers) methodology. Enforces test-driven development, systematic debugging, and quality gates at every stage.

![Superpowers Org Chart](./superpowers/images/org-chart.png)

**Agents:** CEO, Lead Engineer, Code Reviewer, Release Engineer

### [Default](./default)

Baseline agent configurations (CEO, default roles) used as the starting point when creating new companies.

## Structure

Each company directory contains:

- `COMPANY.md` — company metadata, description, and goals
- `agents/` — agent configurations with role-specific prompts
- `skills/` — workflow skills available to agents
- `README.md` — detailed company documentation
- `.paperclip.yaml` — Paperclip configuration

The repo also includes top-level shared skills:

- `skills/company-creator` — scaffolds new agent company packages that follow the Agent Companies spec
- `.agents/skills/company-creator` — compatibility symlink for agents that discover skills from `.agents/skills`

## Shared Skills

### `company-creator`

Use `company-creator` when you want an agent to create a new company package from scratch, turn an existing repo into a company, or scaffold a team around an existing workflow.

Canonical path: `skills/company-creator/SKILL.md`

Compatibility path for agent skill discovery: `.agents/skills/company-creator`

## Usage

These companies are managed by the Paperclip platform. See the [Paperclip docs](https://github.com/paperclipai/paperclip) for setup and deployment.

To use the shared `company-creator` skill with an agent:

```text
Use the company-creator skill to create a new company for a product design team.
```

```text
Use the company-creator skill to turn this repo into a Paperclip company package.
```

The skill will interview you, scaffold the company package, and write the output where you choose.

To import a generated company into Paperclip:

```bash
paperclipai company import --from /path/to/company
```
