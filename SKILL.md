---
name: test-planner
description: >
  Generate a complete E2E test suite for any web application using Autonoma's 4-step workflow:
  knowledge base generation, test data scenarios, E2E test case generation, and Environment
  Factory implementation. Use when the user wants to set up Autonoma testing, generate E2E
  tests, or implement the Environment Factory for their application.
---

# Autonoma Test Planner

This skill orchestrates the Autonoma Test Planner workflow - four sequential steps that produce a production-ready E2E test suite from any web application codebase.

## Before you start

**Always fetch the latest documentation before beginning any step:**

1. Fetch `https://docs.agent.autonoma.app/llms.txt` to get the documentation index.
2. From that index, find the **Test Planner** section. It contains five pages: an overview and four step pages (Step 1: Knowledge Base, Step 2: Scenarios, Step 3: E2E Tests, Step 4: Implement Scenarios).
3. Fetch each of the four step pages. Each one contains the **full prompt** you must follow to execute that step. These prompts are the source of truth - the docs may have been updated since this skill was installed.
4. Also fetch the **Environment Factory Guide** and any **framework example** that matches the user's stack. You will need these for Steps 2 and 4.

**Use the full prompt from each step page as your instructions.** Do not summarize or abbreviate the prompts. Execute them as written.

## Workflow

Execute the four steps in order. **Do not start the next step until the current step's review checkpoint is complete and the user has confirmed.**

**Before starting each step**, compact the conversation context and tell the user what is about to happen:

> "I'm about to start **Step N: [Name]**. This step will [brief description of what it does and what it produces]. I'll follow the full prompt from the docs. Here's what to expect: [what the user will need to review or answer]."

This keeps the user informed and ensures context stays manageable across the full workflow.

### Step 1: Generate Knowledge Base

**What it does:** Analyzes the frontend codebase and produces `AUTONOMA.md` (a user-perspective guide to every page, flow, and interaction) plus `skills/` files (step-by-step navigation guides).

**Execute:** Follow the Step 1 prompt from the docs. Use subagents aggressively to parallelize codebase exploration.

**Review checkpoint - Core flows:**
After completing the knowledge base, present the identified core flows to the user and explain:

> "I've identified these as the core flows - the 2-4 workflows that represent the primary reason users use this product:
>
> [list core flows]
>
> **Why this matters:** Core flows determine how the test budget is distributed. They'll receive 50-60% of all generated tests. If the wrong flows are marked as 'core,' your test suite will be heavily weighted toward less critical functionality.
>
> For each flow, ask yourself: 'If this broke silently, would users immediately notice and stop using the product?'"

**Then use the `AskUserQuestion` tool** with a multiple choice question to block until the user responds. This ensures they see the checkpoint and know the process needs their input before continuing. Example options: "Looks good, proceed to Step 2", "I want to add/remove a core flow", "I want to change priorities".

### Step 2: Generate Scenarios

**What it does:** Explores the data model and designs three named test data environments (`standard`, `empty`, `large`) with concrete entity names, counts, and relationships.

**Execute:** Follow the Step 2 prompt from the docs. The knowledge base from Step 1 must be available.

**Review checkpoint - Scenario data:**
After generating scenarios, present a summary and explain the full causal chain:

> "Here's what I've generated for the `standard` scenario:
>
> [summary of key entities, counts, relationships]
>
> **What scenarios are:** Each scenario is a named test data environment. Before each test run, Autonoma calls your Environment Factory to create this data fresh. After the run, it tears it all down. Every run starts clean.
>
> **Why exact values matter:** The test generation agent (Step 3) will write tests that assert against these specific names and counts. For example, if the scenario says there are 15 failed runs, a test will literally assert 'the runs page shows 15 failed runs.' If the Environment Factory (Step 4) creates different data, those tests fail - not because of a bug, but because of a mismatch.
>
> **What to check:**
> - Are the entity names realistic and distinguishable?
> - Are the counts reasonable for testing all filter/category options?
> - Are relationships explicit (which test is in which folder)?
> - Does every status/type/category enum value have at least one entity?"

**Then use the `AskUserQuestion` tool** with a multiple choice question to block until the user responds. Example options: "Looks good, proceed to Step 3", "I want to adjust entity names or counts", "I want to change relationships".

### Step 3: Generate E2E Tests

**What it does:** Produces an exhaustive set of E2E test cases as markdown files, distributed across tiers (core flows get 50-60%), with an adversarial review pass to find gaps.

**Execute:** Follow the Step 3 prompt from the docs. Both the knowledge base and scenarios must be available.

**Review checkpoint - Test sampling:**
After generating and packaging all tests, sample 3-5 Journey tests and 3-5 Critical-priority tests from each core flow. Present each sampled test to the user with an explanation:

> "Here are some key tests from the suite for you to review:
>
> **Journey Test: [name]**
> - Chains these flows: [flow 1] -> [flow 2] -> [flow 3]
> - What it does: [plain language walkthrough]
> - What bug it catches: [explanation]
>
> **Critical Test: [name]** (from [core flow])
> - What it tests: [description]
> - Key assertions: [list the specific text/UI state being asserted]
> - What bug it catches: [explanation]
>
> [repeat for each sampled test]
>
> **What to look for:**
> - Do steps reference actual button labels and field names from your UI?
> - Are assertions specific (exact toast text, exact error message)?
> - Does the test budget feel right for your core flows?"

**Then use the `AskUserQuestion` tool** with a multiple choice question to block until the user responds. Example options: "Looks good, proceed to Step 4", "I want to adjust some tests", "I want to regenerate tests for a specific flow".

### Step 4: Implement Scenarios

**What it does:** Takes `scenarios.md` and implements the Environment Factory endpoint (`discover`, `up`, `down`) following the protocol documented at `https://docs.agent.autonoma.app`.

**Execute:** Follow the Step 4 prompt from the docs. This step involves:
1. Asking the user about endpoint location, database layer, auth mechanism, and signing secret
2. **Going into plan mode** - presenting the full implementation plan
3. Waiting for plan approval before writing any code
4. Implementing the endpoint with integration tests

**Review checkpoint:** The plan-mode approval gate IS the review checkpoint. The agent presents the entity creation order, teardown order, auth strategy, and security implementation. The user approves before any code is written.

Read the Environment Factory Guide and the relevant framework example from the docs when implementing. The guide contains the full protocol specification, security model, and teardown rules.

## Rules

- **Always fetch `https://docs.agent.autonoma.app/llms.txt` before starting Step 1.** From the index, fetch each Test Planner step page to get the full prompts. These are your instructions - execute them as written, not from memory.
- **Never skip review checkpoints.** Errors in early steps compound into later steps. A wrong core flow identification leads to a poorly weighted test suite. Wrong scenario data leads to failing tests. The checkpoints exist to catch these errors early.
- **Always use `AskUserQuestion` at review checkpoints.** Present your review summary as a message, then call `AskUserQuestion` with multiple choice options so the user gets an interactive prompt. Never just end with a plain text question - the user may miss it and think the process is done.
- **Complete steps in order.** Step 2 depends on Step 1's output. Step 3 depends on Steps 1 and 2. Step 4 depends on Step 2. Don't skip ahead.
- **Use subagents for parallel exploration** in Steps 1, 2, and 3. These are large tasks - serial exploration will take too long.
- **In Step 4, always go into plan mode before writing code.** Present the full implementation plan and wait for approval.
- **Reference the docs framework examples in Step 4.** If the user's stack matches one of the documented examples (Next.js, React+Vite, Elixir/Phoenix, TanStack Start), use that example as the primary reference for implementation patterns.
