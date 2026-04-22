---
name: notion-cli
description: >-
  Use the Notion CLI (`ntn`) to interact with the Notion API, manage workers,
  and upload files. Use when the user asks to "call the Notion API", "deploy a
  worker", "upload a file to Notion", "create a page", "query a database", or
  any task involving the `ntn` command.
---

# Notion CLI

## Look things up before answering

The CLI is self-documenting. Always prefer running these commands over guessing
syntax or relying on memorized knowledge:

- `ntn api ls` — list every public API endpoint.
- `ntn api <path> --help` — show methods, doc links, and usage for an endpoint.
- `ntn api <path> --docs` — print the full official docs for an endpoint.
- `ntn api <path> --spec` — print a reduced OpenAPI fragment (useful for
  understanding request/response schemas).
- `ntn <command> --help` — help for any command or subcommand.

## Install

```bash
curl -fsSL https://ntn.dev | bash
```

## Authentication

- The CLI automatically uses `NOTION_API_TOKEN` when it is set.
- Check `NOTION_API_TOKEN` first. If it is already set, prefer using it instead
  of telling the user to run `ntn login`.
- `ntn login` / `ntn logout` — log the CLI in or out (only use if not using
  `NOTION_API_TOKEN`). `ntn login` requires the user to visit a URL in a web
  browser.

## `ntn api`

Run `ntn api --help` for full syntax. Quick summary:

```bash
# GET with query param
ntn api v1/users page_size==100

# POST with inline body fields
ntn api v1/pages parent[page_id]=abc123

# POST with JSON body
ntn api v1/pages -d '{"parent":{"page_id":"abc123"}}'
```

The method is inferred (GET by default, POST when a body is present). Override
with `-X METHOD`.

When creating pages or comments, Markdown is available as an input method; use Markdown by default unless you need more advanced formatting.

## `ntn files`

Convenience wrapper around the File Uploads API.

```bash
ntn files create < image.png
ntn files create --external-url https://example.com/photo.png
ntn files list
ntn files get <upload-id>
```

## `ntn workers`

Manage Notion workers (deploy, list, execute, etc.). Run `ntn workers --help`
for subcommands.

```bash
ntn workers new my-worker        # scaffold a new project
ntn workers deploy               # deploy from current directory
ntn workers ls                   # list workers
ntn workers exec <capability>    # execute a capability
```
