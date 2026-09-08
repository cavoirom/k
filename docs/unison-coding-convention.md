# Unison coding convention

**Important: the convention is only applied to the codebase of this project, not the dependencies or external code.**

## Naming

- Namespace: separate the hierarchy by dot `.`, use _snake_case_ for the hierarchy segments, the Type / Ability segment will follow their convention, e.g. `k.models.Codex`.
- Type, data constructor: use _PascalCase_, e.g. `LocalShell`.
- Ability: use _PascalCase_, e.g. `Model`.
- Type variable: use _camelCase_, prefer single letter, e.g. `k`, `v`, `s`, `modelProvider`.
- Function, request operation, value, field, parameter: use _snake_case_, e.g. `submit_message`.
- Test case: use _snake_case_. The name should describe the being tested behavior and expected outcome, e.g. `should_throw_error_when_timeout`.
- Branch: use _snake_case_. Prefer descriptive name. Avoid dots, special characters. E.g. `add_openai_support`.

## Testing

- Place the tests in a nested namespace of the test target, e.g. `k.models.Codex.tests` is the tests of `k.models.Codex`.
- `.tests` contains the pure, deterministic tests, including testing abilities with deterministic handlers. Cached result is acceptable.
- `.io_tests` contains tests which actually use I/O, sub-processes, networking...
- `.live_tests` contains a specific I/O tests that involves real systems that are risky to run automatically, e.g. calling LLM APIs. These tests are optional and will be requested explicitly.
- Prefer TDD approach, we will work to define the tests first, then treat the tests as source of truth to write the implementation.
- The test must be in Arrange / Act / Assert (AAA) format.
