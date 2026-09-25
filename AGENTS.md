# AGENTS.md

## Overview

**k** is a minimal coding agent, written in Unison programming language. We try to make **k** simplest, smallest and most readable agent.

## Behaviors

- Never change @README.md and @AGENTS.md unless I requested.
- Never restate the instructions in your response, just follow it strictly.

## Working environment

- You are working inside a Guix shell container.
- Unison's UCM v1.3.0 is available.
- `busybox` is available.
  - When using `mktemp`, use it to create a unique directory by pattern ending with `.XXXXXX`, then put the temporary files inside it.
- `deno@2.8.1` is available, use it as the `python` replacement when you do scripting.
- Never use `rg` because I blocked it, it will never be available.
- Never download other tools, let me know if you need any.

## Key technical decisions

- Use Unison programming language, version 1.3.0 to implement **k**.
- The codebase is stored in UCM database.
- Only use Unison standard library and approved libraries:
  - `@unison/base@7.19.2`.
  - `@unison/http@16.1.0`.
  - `@unison/json@1.4.2`.
- Git and Unison default branch must be `master`. Never use `main`.
- Use mini-swe-agent v2 as the reference.

## Unison coding convention

Use convention described in `./docs/unison-coding-convention.md`.

## Unison codebase

- `k/master`: the first version of k, written as an experiment to understand how coding agent works.
- `k/next`: the careful implementation of k, currently not yet existed.

## Unison namespaces

These are the new namespaces will be used in `k/next` branche. The old namespaces won't changes, and will be used in `k/master`.

- `k.config`:
  - User configuration regarding LLM, credentials and so on...
  - Internal configuration.
- `k.state`: store the mutable state of the system such as selected profile, authentication...
- `k.agent`: the control flow of the coding agent.
- `k.environment`: execute agent actions.
- `k.model`: connect to LLMs.
- `k.shared`: contain shared data models to transfer data between components.

## Unison programming

- Use non-interactive UCM with option `ucm --codebase "$PWD"`, treat Unison codebase as the source of truth, do not base on exported text files or modify database files directly.
- Always consult [language reference](https://www.unison-lang.org/docs/#language-reference) before writing / updating code to program with correct syntax.
- Never guess a library definition or assume it exists under a familiar name. Search by name/type or inspect existing project usage first.
- Always use Unison MCP server for working with Unison codebase, fallback to non-interactive UCM `transcript.in-place` when MCP could not do the expected operation (merge / delete branch, export to `k.usync`...).
- Code editing workflow:
  - Create new project branch from `k/master`.
  - Work on the created branch.
  - Before merging, always present the UCM diff and test results, compile the program to `k.uc`, then wait for explicit approval unless I initially asked you to merge.
  - Merge to `k/master` when approved.
  - Verify the merged result on `k/master`.
  - Delete the merged branch.
  - Export the codebase to `k.usync`.
- `k.usync` must contain the complete exported `k/master` branch. Do not stage or commit it unless explicitly requested. Clean up any text file you produced after finishing your work.
- Common tasks:
  - Compile the codebase, UCM transcript command: `> compile k.main ./k`. `k.main` is the entrypoint, the command will produce `k.uc` file in the current directory.
  - Compare branches: `> diff.branch k/master k/<branch>`.
  - Switch branch: `> switch k/<brach>`.
  - Export codebase: `> sync.to-file <absolute-path-to-k.usync>`.
- Notes:
  - UCM transcript command lines must begin with `> `.
  - Use `transcript.in-place` for operations intended to modify the working codebase. Plain `transcript` runs against a temporary sandbox.
  - When creating a UCM transcript, place a `.md` file inside the temporary directory.
  - Use the Unison MCP server to get the current project and branch, never mis-identify the default branch as active branch.

# Plaintext editing

- Use git patch for small, targeted editing of existing file.
