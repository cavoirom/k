# AGENTS.md

## Overview

**k** is a minimal coding agent, written in Unison programming language. We try to make **k** simplest, smallest and most readable agent.

## Behaviors

- Never change @README.md and @AGENTS.md unless I requested.

## Working environment

- You are working inside a Guix shell container.
- `busybox` is available.
- Unison's UCM v1.3.0 is available.
- Never use `rg` because I blocked it, it will never be available.
- Never download other tools, let me know if you need any.

## Key technical decisions

- Use Unison programming language, version 1.3.0 to implement **k**.
- The codebase is stored in UCM database.
- Only use Unison standard library and approved libraries:
  - `@unison/http@16.1.0`.
  - `@unison/json@1.4.2`.
- Prefer TDD approach, we will work to define the tests first, then treat the tests as source of truth to write the implementation. The test must be in Arrange / Act / Assert (AAA) format.
- Git and Unison default branch must be `master`. Never use `main`.
- Use mini-swe-agent v2 as the reference.

## Unison namespaces

- `k.config`:
  - User configuration regarding LLM, credentials and so on...
  - Internal configuration.
- `k.agents`: the control flow of the coding agent.
- `k.environments`: execute agent actions.
- `k.models`: connect to LLMs.

## Unison programming

- Use non-interactive UCM with option `ucm --codebase "$PWD"`, treat Unison codebase as the source of truth, do not base on exported text files or modify database files directly.
- Always consult [language reference](https://www.unison-lang.org/docs/#language-reference) before writing / updating code to program with correct syntax.
- Alawps use Unison MCP server for working with Unison codebase, fallback to non-interactive UCM `transcript.in-place` when MCP could not do the expected operation (merge / delete branch, export to `k.usync`...).
- Code editing workflow:
  - Create new project branch from `k/master` with a descriptive name. Branch name convention: word sparated by hyphen `-`. avoid dot, special characters.
  - Work on the created branch.
  - Present the UCM diff and test results, complie the program, then wait for explicit approval.
  - Merge to `k/master` when approved.
  - Verify the merged result on `k/master`.
  - Delete the merged branch.
  - Export the codebase to `k.usync`.
- `k.usync` must contain the complete exported `k/master` branch. Do not stage or commit it unless explicitly requested. Clean up any text file you produced after finished your work.
- Common tasks:
  - Compile the codebase, UCM transcript command: `compile k.main ./k`. `k.main` is the entrypoint.
