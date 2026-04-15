# CLAUDE.md

## What is this repo
Documentation site for Syntaxia (docs.syntaxia.com). Built on Mintlify. Pages are MDX files with YAML frontmatter. Configuration lives in `docs.json`.

## What is Syntaxia
B2B SaaS platform. The operating system for business meaning. Unified truth for go-to-market data. We help RevOps teams reconcile fragmented CRM data into a single source of truth.

Syntaxia is meant for humans and AI.

- Website: www.syntaxia.com
- Webapp: app.syntaxia.com
- Docs: docs.syntaxia.com
- Support: support@syntaxia.com

## Stack
- Mintlify (docs platform)
- MDX (Markdown + JSX components)
- `docs.json` (site configuration: navigation, theme, branding)

## Commands
```bash
mint dev              # Preview locally at http://localhost:3000
mint broken-links     # Check for broken links
mint update           # Update Mintlify CLI to latest
```

## Conventions
- No em dashes in copy or comments. Use commas, periods, or "to" instead.
- Use active voice and second person ("you")
- Keep sentences concise. One idea per sentence.
- Use sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references
- Lead with the goal: start instructions with what the user wants to accomplish
- Use consistent terminology. Don't alternate between synonyms for the same concept.

## Key files
| File | Purpose |
|---|---|
| `docs.json` | Site config: name, colors, logos, navigation, navbar, footer |
| `index.mdx` | Landing page |
| `favicon.svg` | Syntaxia pinwheel favicon |
| `logo/light.svg` | Syntaxia wordmark for light backgrounds (black text + green pinwheel) |
| `logo/dark.svg` | Syntaxia wordmark for dark backgrounds (white text + green pinwheel) |
| `.mintignore` | Files excluded from Mintlify build |
| `snippets/` | Reusable MDX snippets |
| `images/` | Static images |

## Brand

### Colors
| Token | Hex | Use |
|---|---|---|
| Primary | `#18E299` | Bright green. Links, buttons, accents |
| Dark | `#0C8C5E` | Dark green. Hover states, dark mode accents |
| Logo text (light mode) | `#000000` | Black wordmark on light backgrounds |
| Logo text (dark mode) | `#FFFFFF` | White wordmark on dark backgrounds |

### Logo
The Syntaxia logo is a pinwheel mark (three curved segments in `#18E299` and `#0C8C5E`) followed by the "mintlify" wordmark. SVGs are in `logo/light.svg` and `logo/dark.svg`. The favicon at `favicon.svg` is the pinwheel mark only.

### Typography
- **Body:** Roobert (sans-serif). Self-hosted WOFF2 in the docs repo at `/fonts/`.
- **Code:** JetBrains Mono (monospace).
- **Marketing site** and **docs site** both use Roobert. The product app uses Inter.

### Icons
Mintlify uses Font Awesome icons natively in `icon` props on Card, Tab, and Anchor components. Use Font Awesome icon names (e.g., `"rocket"`, `"database"`, `"plug"`).

The product app uses Tabler Icons, but Mintlify does not support them. Use the closest Font Awesome equivalent.

## docs.json structure

Navigation is organized as tabs > groups > pages:

```json
{
  "navigation": {
    "tabs": [
      {
        "tab": "Tab Name",
        "groups": [
          {
            "group": "Group Name",
            "pages": ["path/to/page"]
          }
        ]
      }
    ]
  }
}
```

Every page referenced in `docs.json` must have a corresponding `.mdx` file, or the build breaks.

### Mintlify font config
Fonts are configured in `docs.json` via the `fonts` key. Supports Google Fonts by family name:
```json
{
  "fonts": {
    "body": { "family": "Inter" },
    "heading": { "family": "Inter", "weight": 600 }
  }
}
```

### Mintlify theme options
Available themes: `mint`, `maple`, `palm`, `willow`, `linden`, `almond`, `aspen`, `sequoia`, `luma`.

### Background decorations
Available: `gradient`, `grid`, `windows`.

## MDX page template
Every page needs YAML frontmatter with `title` and `description`:

```mdx
---
title: "Page title"
description: "One-line description for SEO and navigation."
---

Page content here.
```

## Common Mintlify components
```mdx
<Card title="Title" icon="icon-name" href="/path">Description</Card>
<Columns cols={2}>...</Columns>
<Info>Informational callout</Info>
<Warning>Warning callout</Warning>
<Tip>Tip callout</Tip>
<Tabs><Tab title="Tab 1">...</Tab></Tabs>
<CodeGroup>...</CodeGroup>
<Steps><Step title="Step 1">...</Step></Steps>
```

## Product concepts (for writing docs)

### Authentication
Passwordless via magic link codes. No passwords anywhere.
- Identity: unique email address. Can join multiple organizations.
- MagicLink: 6-character code (A-Z minus O/I/L, 2-9), 15-minute TTL.
- Session: browser session tied to an Identity.
- User: membership record linking Identity to Organization. Roles: owner, admin, member.
- Organization::JoinCode: team invite code in `XXXX-XXXX-XXXX` format (base58).

### Auth flows
1. **Sign in**: enter email, receive 6-char code, enter code, session created
2. **Sign up**: enter email (must be whitelisted domain), verify code, enter name + org name, org created with user as owner
3. **Join via invite**: visit `/join/:code`, enter email, verify code, enter name, user added as member

### Organizations
Multi-tenant. Every user belongs to one or more organizations. Each org has a slug, users, source systems, AI providers, and an Anchor connection. Roles: owner (full control), admin (manage sources + AI), member (basic access).

### Source systems
A connected CRM (Salesforce, HubSpot). Connected via OAuth through Fivetran.

**Lifecycle:** pending -> authorizing (OAuth) -> syncing (first sync) -> active (data flowing) -> paused / error / disconnected / purged

**Disconnect** (soft-delete): pauses connector, preserves data.
**Purge** (hard-delete): deletes Fivetran connector, S3 data, and database record.

### Data pipeline
```
CRM Sources -> Fivetran -> S3 (Delta Lake) -> dbt (DuckDB) -> S3 (Parquet) -> DuckDB counting -> Postgres -> UI
```

1. **Fivetran** extracts CRM data via OAuth. Writes Delta Lake tables to S3.
2. **dbt** normalizes raw tables into staging Parquet files using Fivetran's maintained packages.
3. **DuckDB** counts staged records. Results written to Postgres.
4. **The frontend** reads from Postgres only.

### AI providers (BYOK)
Bring Your Own Key. Organizations configure their own LLM API keys.

| Provider | Models |
|---|---|
| Anthropic | Claude (Opus, Sonnet, Haiku) |
| OpenAI | GPT-4o, o3, ChatGPT |
| xAI | Grok |

One active provider per org at a time. Usage tracked per request (tokens, estimated cost).

### Ontology discovery
AI-powered multi-agent debate protocol. Analyzes CRM data per source and produces a structured ontology (L2) representing concepts, relationships, and meaning.

Uses a Python sidecar (FastAPI + LangGraph) with multiple specialized agents: Explorer, Micro-Critics (8 quality dimensions), Arbiter, Confidence Calibrator. Up to 3 debate rounds.

### Ontology composition
Cross-source alignment. Takes multiple L2 ontologies and produces a unified L3 ontology via a second debate protocol. Discovers where concepts overlap, resolves conflicts, produces unified go-to-market truth.

### Anchor
Ontology registry. Git-like versioning with immutable commits and semantic versioning. Supports YAML DSL, OWL, RDF, Turtle formats. Exposes an MCP server for AI agent integration. Separate product at anchor.syntaxia.com.

### Functions
Role-based workspaces that scope what users see and do:
- **Data Ops**: full source management, sync triggers, ontology discovery. The primary workspace for data teams.
- **Rev Ops**: read-only view of source status and ontology outputs. For revenue operations teams.

### Source card scoping
- **Data Ops**: source cards show action icons (resync, reprocess, ontology discovery, reconnect, disconnect). "Add Source" button visible.
- **All other functions**: source cards are read-only status indicators. No action icons.
- **Within Data Ops**: admins/owners can trigger reprocess + ontology discovery. Members can trigger resync, reconnect, disconnect.

## Related repos
- **Syntaxia webapp**: `../syntaxia` (Rails 8 monolith)
- **Anchor**: `../syntaxia-anchor` (ontology registry, separate Rails app)
- **Python sidecar**: `../syntaxia/sidecar/` (FastAPI + LangGraph, ontology pipeline agents)
# Shaping Debate Engine -- Orchestration Rules

Append this section to the CLAUDE.md in any repo where you want the shaping
debate engine available.

---

## Shaping workflow

When the user says "shape [feature]", "shape up [feature]", or asks to shape
a product feature, activate the debate protocol using three subagents:

- **shaper** -- owns the Shape Up methodology, produces structured documents
- **codebase-advocate** -- owns technical reality, validates against the repo
- **linear-publisher** -- post-convergence, updates the Linear project

### Starting a session

The user may provide the Linear project link in their opening message, like:

> "We're now working on pitching X that's in this project
> https://linear.app/syntaxia-os/project/ontology-visualizer-ab0d642d1966"

If they do, extract and store the project URL for Phase 6.

If they do not, ask: **"Which Linear project is this for? Paste the project
URL or name."** Collect this before starting Phase 1. The project must exist
in Linear already.

### Information to collect before shaping

1. **Linear project URL or name** (required)
2. **Appetite** -- Small Batch (2 weeks) or Big Batch (6 weeks)
3. **Any additional context** -- pain points, user quotes, prior discussions

The team and initiative are already set on the existing Linear project, so
you do not need to ask for those.

### Debate protocol

**Parsing agent signals:**
- The **shaper** ends every response with `VERDICT: NEEDS VALIDATION` or
  `VERDICT: COMPLETE`. If `NEEDS VALIDATION`, route to codebase-advocate.
  If `COMPLETE`, present to user for approval.
- The **codebase-advocate** ends every response with `CONVERGENCE: YES` or
  `CONVERGENCE: NO -- [reason]`. If `YES`, the current phase is stable.
  If `NO`, relay the objections to the shaper for revision.

**Phase 1: Framing**

1. Collect raw input from the user (problem, pain points, context).
2. If Appetite was not provided, ask for it now.
3. Send to shaper: "Produce a Frame (Source, Problem, Outcome) from: [input]"
4. Send Frame to codebase-advocate: "Review this Frame against the codebase."
5. If objections, relay to shaper for revision. Repeat until stable.
6. Present agreed Frame to user. Get approval before proceeding.

**Phase 2: Requirements + Shapes**

1. Send to shaper: "Propose requirements (R0-Rn) and at least two shapes.
   Shape alternatives must differ in their fundamental mechanism, not just
   in detail. If Shape A uses polling, Shape B should use push. If A is
   client-side, B should be server-side. [frame]"
2. Send to codebase-advocate: "Validate these against the codebase. [output]"
3. Relay critique to shaper. Repeat until both stabilize.

**Phase 3: Fit Check + Selection**

1. Send to shaper: "Produce fit check matrix. [R and shapes]"
2. Send to codebase-advocate: "Validate every pass cell. [fit check]"
3. Iterate until stable. Present to user for shape selection.

**Phase 4: Breadboarding**

1. Send to shaper: "Breadboard the selected shape into affordance tables. [shape + R]"
2. Send to codebase-advocate: "Map affordances to actual code. [breadboard]"
3. Iterate until all affordances are mapped or identified as new work.

**Phase 5: Slicing**

1. Send to shaper: "Slice into vertical increments with demo-able UI. [breadboard]"
2. Send to codebase-advocate: "Validate slice ordering and dependencies. [slices]"
3. Iterate until slices are clean.

**Phase 6: Convergence + Output**

Convergence = stable fit check + zero advocate objections + clean slices.

1. Send to shaper: "Produce final Pitch and Big Picture. Appetite: [X]."
2. Write to `.claude/shaped/[feature-name]/pitch.md` and `big-picture.md`.
3. Present to user for final approval.
4. On approval, send to linear-publisher: "Update the Linear project at
   [URL collected at start]. Read pitch from [path] and big picture from
   [path]."
5. Report the updated Linear project URL.

### Orchestration rules

- Every shaper output must be validated by the codebase-advocate before the next phase.
- Track and announce phase transitions.
- Present checkpoints at the end of each phase. Get user approval to advance.
- If the same objection bounces 3+ times, surface it to the user as a decision point.
- **Context management:** For features with 5+ requirements or 3+ shapes,
  persist intermediate artifacts to `.claude/shaped/[feature-name]/` as
  files rather than inlining all content in delegation prompts. Reference
  file paths in delegations: "Read the current requirements from
  `.claude/shaped/[feature-name]/shaping.md`." This prevents context
  window overflow in long shaping sessions.
- When delegating, include all prior artifacts in the prompt (subagents
  have no memory between calls). For large artifacts, reference file paths
  instead of inlining.
- The user is the tiebreaker. Agents debate; the user decides.
# Dev Pipeline -- Orchestration Rules

Append this section to the CLAUDE.md in the Syntaxia repo to enable the
dev pipeline.

---

## Dev pipeline

When the user provides a Linear project URL and says "build this," "implement
this," "start dev on this," or similar, activate the dev pipeline using these
agents:

- **tech-lead** -- reads the project, validates issues, plans execution order
- **developer** -- plans and implements one issue at a time
- **plan-reviewer** -- reviews implementation plans before coding starts
- **code-reviewer** -- reviews completed code on feature branches
- **pr-creator** -- pushes branches and creates GitHub pull requests

### Starting a session

The user provides a Linear project URL:

> "Build https://linear.app/syntaxia-os/project/time-machine-f426e3b51e7c"

Extract the project URL and proceed to Phase 1.

### Resuming a session

If `.claude/dev-pipeline-status.md` exists, read it. For each issue,
determine its current state and pick up from the next step:

| Current state | Next step |
|---------------|-----------|
| `waiting` | Check if dependencies are now `ready`, then start at 2a |
| `planned` | Start at 2b (plan review) |
| `reviewed` | Start at 2c (implement) |
| `implemented` | Start at 2d (code review) |
| `code-reviewed` | Mark as `ready` |
| `ready` | Skip (done, awaiting Phase 3) |
| `failed` | Surface to user for decision |

Do not re-run completed steps. Present the resumed status to the user
before continuing.

### Phase 1: Tech Lead reviews and plans

1. Delegate to **tech-lead**: "Review this project and produce an execution
   plan. Project URL: [url]"
2. The tech lead reads the pitch, big picture, and issues. Validates
   coverage. Creates or edits issues as needed.
3. The tech lead returns an execution plan with groups (parallel and
   sequential).
4. Present the plan to the user. Get approval before proceeding.

### Phase 2: Issue-by-issue execution

Process issues in the order defined by the tech lead's execution plan.
Within a parallel group, process issues sequentially (Option A).

For each issue:

**2a. Developer plans**

Delegate to **developer** in plan mode: "Plan implementation for [ISSUE-KEY].
Issue description: [full description]. Big Picture context: [relevant
section]. Branch: quentin/[issue-key-lowercase]-[slug]."

**2b. Plan review**

Delegate to **plan-reviewer**: "Review this plan for [ISSUE-KEY]. Plan:
[developer's plan]. Issue: [description]. Big Picture: [context]."

Parse the first line of the response:
- `VERDICT: APPROVED` -- proceed to 2c.
- `VERDICT: OBJECTIONS` -- send objections back to the developer for
  revision. Include the objections and the original plan. Repeat until
  approved. Max 3 rounds. If not approved after 3, surface to the user.

**2c. Developer implements**

Delegate to **developer** in implement mode: "Implement [ISSUE-KEY].
Approved plan: [plan]. Branch name: quentin/[issue-key-lowercase]-[slug]."

The developer creates the branch, writes code, writes tests, runs `bin/ci`,
commits, and returns to main.

**2d. Code review**

Delegate to **code-reviewer**: "Review branch
quentin/[issue-key-lowercase]-[slug] for issue [ISSUE-KEY]. Issue
description: [description]."

Parse the first line of the response:
- `VERDICT: APPROVED` -- proceed to 2e.
- `VERDICT: FIXES REQUIRED` -- send fixes back to the developer in
  implement mode with the branch name and the list of fixes. The developer
  checks out the branch, applies fixes, runs `bin/ci`, amends the commit,
  returns to main. Then re-review. Max 3 rounds.

**2e. Move to next issue**

Once approved, note the branch as ready. Update the issue status in Linear
to "In Review" via the tech-lead (or post a comment noting completion).
Proceed to the next issue.

### Phase 3: Pull requests

After all issues in the execution plan are implemented and reviewed:

1. Collect the list of branches and their issue keys.
2. Delegate to **pr-creator**: "Create PRs for these branches: [list]."
3. Report the PR URLs to the user.

### Orchestration rules

- Always start on the `main` branch. Before each developer invocation,
  verify: `git branch --show-current` should return `main` (unless resuming
  work on a branch for fixes).
- Carry full context between delegations. Every developer invocation must
  include: the issue description, the approved plan (in implement mode),
  the branch name, and any reviewer feedback.
- **Persist status after every state transition.** Write to
  `.claude/dev-pipeline-status.md` in this format:
  ```
  # Dev Pipeline Status
  ## [Project Name]
  CORE-80: ready (2026-03-20T14:30)
  CORE-81: code-reviewed (2026-03-20T15:10)
  CORE-82: waiting -- depends on CORE-80
  ```
  Valid states: `waiting`, `planned`, `reviewed`, `implemented`,
  `code-reviewed`, `ready`, `pr-created`, `failed`.
- Present status to the user between groups. At the end of each parallel
  group, show what was completed and what comes next.
- If a developer reports that the plan is wrong mid-implementation, stop
  that issue. Return to plan mode with the developer's findings. Re-review.
- If `bin/ci` fails after 3 developer attempts, escalate to the user with
  the full error output.
- The user can interrupt at any point to adjust the plan, skip an issue,
  or change the order.
