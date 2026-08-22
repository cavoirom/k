# k

**k** is my personal coding agent, its name was inspired by the protagonist of Blade Runner 2049.

## Why Unison?

- Unison is _content-addressed_, pure functional programming language. Its codebase accepts only code that typechecks, while content addressing tracks exactly what has changed. Unchanged definitions are not recompiled, and deterministic pure tests re-run only when their dependencies change. and the tests are deterministic. Test with side effects are separated and always re-run. This lead to a very fast development feedback loop by avoiding unnecessary recompilation and repeated test runs.
