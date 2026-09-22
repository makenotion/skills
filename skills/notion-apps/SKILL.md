---
name: notion-apps
description: Build Notion Apps with the Notion Apps SDK and `ntn`, including workflows and database syncs. Use when creating, extending, or deploying a Notion App.
---

# Notion Apps

## Confirm alpha access first

Before running commands, inspecting a project, or planning the App, ask the user to
confirm that they are in the Notion Apps alpha. Do not substitute CLI authentication,
an enabled CLI experiment, or a locally working SDK for this confirmation.

If the user says no or cannot confirm, refuse to continue with the App workflow. Explain
briefly that SDK code may build locally, but deployment will fail without alpha access.
Do not scaffold or implement the App.

## What Notion Apps are

After alpha access is confirmed, explain as needed that Notion Apps are packaged
automations for extending Notion and creating resources in it. An App can create
databases, pages, custom agents, and other Notion resources. It can also provide:

- Syncs that bring third-party data into Notion.
- Workflows that run complex, long-running automations in response to any event in
  Notion.

## Scaffold before scoping

After the user confirms alpha access, verify that the Notion CLI is installed with
`ntn --version`. If `ntn` is missing, install it with the command for the user's
platform:

```bash
# macOS and Linux
curl -fsSL https://ntn.dev | bash

# Windows
winget install Notion.ntn
```

Once `ntn` is available, immediately scaffold a fresh project with:

```bash
ntn apps new --install --git <app-directory>
```

Always pass `--git` so the new App is initialized as a Git repository.

Agents cannot use the CLI's interactive directory prompt, so always pass an explicit
destination. Use `.` when the current directory is empty and is an appropriate project
root. Otherwise use a reasonable directory from the user's request or surrounding
context. If no safe destination can be inferred, ask the user where the code should go
before scaffolding. Do not run `ntn apps new` without a destination and wait for a
prompt.

Installing dependencies during scaffolding makes the SDK's bundled guidance available.
If the `apps` command is unavailable, enable it with `ntn experiments enable apps`, then
retry the scaffold.

## Agree on the App design before building

After scaffolding but before proposing or implementing the design, work from the new
App's root and read its `AGENTS.md` completely. Then establish what the user wants the
complete App to do. Clarify the desired outcome, source data, triggers, external
services, and every Notion resource required for the App to work. Recommend a workflow
for most automations; use a sync when the goal is to mirror an external collection into
a Notion database. An App may contain both.

Before proposing the design, read only enough to phrase each open decision as a concrete
option: `AGENTS.md` and the top-level description of each relevant capability (workflow,
sync, connections, notion-as-code). That is enough to know what is possible. Do not read
generated type declarations (`*.generated.d.ts`), full provider API surfaces, or other
implementation-level detail before the user has agreed on a direction — save that
verification for the option actually chosen, during implementation.

Present the proposed design in a concise, easy-to-scan format and get the user's
agreement before implementing it. Include:

- Every Notion resource the App will create, such as databases, pages, and custom
  agents, with its purpose and the capabilities that depend on it.
- Every sync, including its third-party source, destination database, and synchronization
  behavior.
- Every workflow, including its trigger, major actions, resources it reads or changes,
  and external connections.
- Any unresolved decisions or required access.

Adapt this format to the App rather than copying it mechanically:

### Example App design

**Outcome:** Bring support tickets into Notion and escalate urgent tickets to the
support team.

**Notion resources**

| Kind | Name | Purpose | Used by |
| --- | --- | --- | --- |
| Database | Support tickets | Store synchronized tickets and triage status | Ticket sync, escalation workflow |
| Page | Support dashboard | Give the team an operational home and database view | Team members |
| Custom agent | Ticket triage | Classify urgency and summarize a ticket | Escalation workflow |

**Syncs and workflows**

| Kind | Name | Source or trigger | Behavior | Dependencies |
| --- | --- | --- | --- | --- |
| Sync | Ticket sync | Support-system tickets | Upsert tickets by stable external ID | Support-system connection, Support tickets database |
| Workflow | Escalate urgent ticket | A Support tickets page is created or updated | Run triage, update status, and notify the support channel | Ticket triage agent, Support tickets database, messaging connection |

**Open questions:** Which support system and messaging channel should the App use?

Ask the user to confirm or revise the design. Do not begin implementation until they
agree. If implementation reveals a material resource or capability not covered by the
agreed design, update the proposal and confirm the change before adding it.

## Follow the generated project's guidance

Treat the generated project and installed SDK as the source of truth for its version.

Before implementing the agreed design, compare it with everything included by the
scaffold. Remove template workflows, syncs, custom blocks, Notion resource declarations,
sample assets, and supporting code that the App does not need. Do not leave example or
placeholder capabilities in discovered capability directories: if they remain there,
the build can include and deploy them. Preserve shared configuration and infrastructure
that the selected capabilities still require.

Feature-specific skills are installed at
`./node_modules/@notionhq/apps/skills`. Inspect that directory and read every relevant
`SKILL.md` completely before implementing a feature. At minimum, use the `workflow`
skill for workflows or the `sync` skill for syncs, plus any additional skills they route
to, such as connections or Notion as Code. Do not rely on remembered SDK APIs when the
installed declarations or skills can answer the question.

Preserve the template's structure and examples unless the user's App requires a change.
Use the project's documented check and build commands after edits. Deploy with `ntn apps
deploy` only when deployment is part of the user's request, and report local build
success separately from deployment success.

## Hand off a deployed App

When deploying, use `ntn apps deploy --json` (plus any other required arguments) so the
final deployment result includes `worker_url`, `setup_url`, and `is_update`. These are
JSON-only field names. Human output shows the worker page URL without a `worker_url`
label, and non-interactive plain output may show only the worker ID followed by a
`Finish setup:` line. That line is the setup URL, not the worker URL.

For a first deployment (`is_update` is false), show only the onboarding/setup URL from
`setup_url` and tell the user to open it to finish setting up the App.

For a redeployment (`is_update` is true), show both `worker_url` and `setup_url`, clearly
labeled and clickable. Explain that the worker URL opens the deployed App and the setup
URL lets them revisit onboarding. Use the URLs returned by the CLI rather than
constructing or guessing them.
