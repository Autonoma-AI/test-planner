# Autonoma Test Planner

A Claude Code skill that generates a complete E2E test suite for any web application — from codebase analysis to working test infrastructure — in four guided steps.

Built by [Autonoma](https://getautonoma.com), the AI-powered testing platform that finds bugs before your users do.

## What it does

The Test Planner skill turns your frontend codebase into a production-ready E2E test suite without writing a single test by hand. It works as an interactive workflow inside Claude Code, walking you through four steps with review checkpoints at each stage so you stay in control of what gets generated.

**The four steps:**

| Step | What happens | What you get |
|------|-------------|--------------|
| **1. Knowledge Base** | Analyzes your frontend codebase — pages, flows, interactions, navigation | `AUTONOMA.md` (user-perspective app guide) + `skills/` (navigation guides) |
| **2. Scenarios** | Explores your data model and designs three test data environments (`standard`, `empty`, `large`) | `scenarios.md` with exact entity names, counts, and relationships |
| **3. E2E Tests** | Generates test cases distributed by priority — core flows get 50-60% of the test budget | Markdown test files with steps, assertions, and scenario references |
| **4. Environment Factory** | Implements the backend endpoint that provisions and tears down test data for each run | Working `discover` / `up` / `down` endpoint with integration tests |

Each step builds on the previous one. The knowledge base informs scenario design, scenarios feed into test generation, and tests reference the exact data the Environment Factory creates. Review checkpoints between steps let you catch issues before they compound downstream.

## Quick start

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured
- A web application codebase (any frontend framework)

### Install the skill

From your project root:

```bash
# Create the Claude Code skills directory if it doesn't exist
mkdir -p .claude/skills

# Download the skill
curl -o .claude/skills/test-planner.md \
  https://raw.githubusercontent.com/Autonoma-AI/test-planner/main/SKILL.md
```

### Run it

Open Claude Code in your project and invoke the skill:

```
/test-planner
```

The skill will fetch the latest documentation from [docs.agent.autonoma.app](https://docs.agent.autonoma.app) and guide you through each step interactively.

## How it works

### Step 1: Knowledge Base

The skill launches subagents to explore your codebase in parallel — pages, components, routes, and user flows. It produces two artifacts:

- **`AUTONOMA.md`** — A user-perspective guide describing every page, flow, and interaction in your app. Written as if explaining the product to a new user, not a developer.
- **`skills/`** — Step-by-step navigation guides that the Autonoma test agent uses to interact with your app during test execution.

You review the identified **core flows** (the 2-4 workflows that represent why users use your product) before moving on. Core flows receive the majority of the test budget, so getting this right matters.

### Step 2: Scenarios

Three named test data environments are designed based on your data model:

- **`standard`** — Realistic variety of entities covering all status types, categories, and filter options
- **`empty`** — Empty state testing (new user experience, zero-data views)
- **`large`** — Pagination, performance, and bulk operation testing

Each scenario specifies exact names, counts, and relationships. These aren't abstract — Step 3 will generate tests that assert against these specific values (e.g., "the dashboard shows 15 failed runs"), so accuracy here prevents false failures later.

### Step 3: E2E Tests

Test cases are generated as markdown files, distributed across priority tiers. Core flows receive 50-60% of the test budget. The generation includes:

- Happy path and validation tests
- State persistence and navigation tests
- Cross-flow **journey tests** that chain multiple flows together
- An adversarial review pass to identify coverage gaps

You review sampled tests before proceeding to verify that button labels, field names, and assertions match your actual UI.

### Step 4: Environment Factory

The skill implements a backend endpoint following the [Autonoma Environment Factory protocol](https://docs.agent.autonoma.app/guides/environment-factory/):

- **`discover`** — Lists available test scenarios
- **`up`** — Creates isolated test data for a specific scenario
- **`down`** — Tears down test data after a run completes

It asks about your stack (database layer, auth mechanism, endpoint location) and enters plan mode before writing any code. Framework-specific examples are available for Next.js, React+Vite, Elixir/Phoenix, and TanStack Start.

## Supported frameworks

The skill works with any frontend framework. The Environment Factory implementation (Step 4) has documented examples for:

- Next.js
- React + Vite
- Elixir / Phoenix
- TanStack Start

Other stacks are supported — the skill adapts to your project's database layer and auth mechanism.

## Documentation

- [Autonoma Docs](https://docs.agent.autonoma.app) — Full documentation including the Environment Factory protocol and framework examples
- [getautonoma.com](https://getautonoma.com) — Product overview

## Contributing

Contributions are welcome. If you find issues with the skill prompts, checkpoint flows, or documentation references, please open an issue or submit a pull request.

## License

MIT
