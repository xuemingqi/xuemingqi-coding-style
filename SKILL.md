---
name: xuemingqi-coding-style
description: Use only when writing, modifying, or refactoring source code, including test code, in any project for this user. Do not trigger for read-only code reading, analysis, explanation, review, running existing tests, or directory-only organization.
---

# Xuemingqi Coding Style

## When to use

Load this skill when the task requires writing, modifying, or refactoring source code, including test code. Do not load it for read-only reading, analysis, explanation, code review, running existing tests, or directory-only organization. If a read-only task later requires code changes, load the skill before writing those changes.

## Core principle

Prioritize a coherent design and clear logic when adding a feature. Assess whether the new behavior calls for refactoring the affected flow before extending the existing code; preserving its current shape or minimizing changed lines is not the goal. Keep the resulting code minimal, explicit, and extensible without over-abstraction.

## Resolve precedence first

Apply guidance in this order:

1. The user's current explicit request.
2. Repository instructions, formatters, linters, and generated-code rules.
3. Consistent conventions in neighboring files.
4. This personal style as the default and tie-breaker.

Do not restyle unrelated code. In a new project, use this skill as the starting convention.

## Load the relevant guidance

- Read [references/code-style.md](references/code-style.md) before writing or modifying code.
- Also read [references/directory-style.md](references/directory-style.md) when a code-writing task also requires creating, moving, renaming, or grouping files and directories.
- Also read [references/java-spring-web.md](references/java-spring-web.md) before changing Java Spring Web code, especially authentication, request identity, HTTP clients, configuration, DTOs, or token handling.
- Also read [references/java-spring-mybatis.md](references/java-spring-mybatis.md) before changing Java Spring code that uses MyBatis or MyBatis-Plus, especially database access, `db` packages, or Service layers.

## Working sequence

1. Inspect a small representative set of nearby files, tests, and enforced tooling.
2. Trace callers, implementations, data models, and tests before changing a contract or deleting code.
3. Evaluate responsibilities, data flow, state ownership, and lifecycle across the affected flow. Decide whether to extend or refactor, and briefly explain substantial design choices before editing. Apply the feature-design guidance in [references/code-style.md](references/code-style.md).
4. Keep the happy path compact; remove repeated checks, queries, conversions, and assignments.
5. Extract stable, stateless, business-agnostic capabilities into focused utility classes in the appropriate common or
   shared module; keep scenario policy and orchestration inside the owning module.
6. Add or update focused tests and run the narrowest relevant verification.
7. Review the diff for broken references, magic values, excessive splitting, misplaced files, and unrelated changes.

## Quick check

| Concern | Expected shape |
|---|---|
| Design | Evaluate the affected flow before extending it; refactor when it makes the resulting responsibilities and logic clearer |
| Names | Full, stable, role-revealing words |
| Flow | Necessary guards followed by a compact, linear happy path |
| Reuse | Shared behavior is extracted once; one-off policy stays local |
| Utilities | Stable business-agnostic capabilities live in focused utility classes; callers retain business policy and error mapping |
| Values | Constants for stable literals; enums for closed value sets |
| Boundaries | Input, output, domain, persistence, and configuration data stay distinct |
| Web identity | Authenticate at the boundary, place trusted identity in context, and let Service read it without controller plumbing |
| HTTP clients | Prefer declarative OpenFeign clients for stable Spring service integrations |
| Class structure | Prefer independent top-level classes in separate files; avoid inner and nested classes, including static nested models |
| Java models | Prefer chainable POJOs for mutable transport/state models; use records only for deliberate immutable values |
| Dependencies | Annotation-driven, constructor-injected, and immutable where practical |
| Persistence | Business services inject `IService` and prefer type-safe Lambda APIs; Mapper stays behind the database service implementation |
| Java services | Contracts live in `service`; implementations live in `service/impl` and contain the orchestration |
| Comments | Explain business intent, constraints, or reasons |
| Service docs | Document service contracts and private helpers; do not repeat interface docs on overrides |
| Entity docs | Add a field comment above every database Entity field |
| Formatting | Do not break immediately after `(` or put every argument on its own line |
| Directories | Follow the repository's organization axis and place every file under its actual responsibility; never use a feature catch-all |
| Tests | Describe observable behavior and remain environment-independent |

## Avoid false imitation

Do not reproduce historical artifacts such as mixed naming, duplicated defensive code, manual framework boilerplate, field injection, magic strings, vague utility buckets, excessive package depth, empty tests, local paths, hard-coded credentials, or obvious comments.
