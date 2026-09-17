# Refined Code Style

## 1. Character and scope

- Prefer minimal, explicit, practical code whose business meaning is visible.
- Keep the happy path compact and linear. Remove redundant checks, queries, conversions, assignments, fields, constants, logs, and return values.
- Validate untrusted input at the boundary; inside the system, rely on established contracts instead of repeating defensive code.
- Before changing a contract or deleting code, trace callers, implementations, models, persistence behavior, and tests.
- Scope changes to the requested behavior, the refactoring needed for a coherent affected flow, and relevant verification. Do not copy nearby defects or restyle unrelated code.

### Feature design before incremental adaptation

- Before adding a feature, review how it changes existing responsibilities, data flow, state ownership, contracts, and resource lifecycle. Consider the resulting design as a whole instead of mechanically adding fields, conditionals, parameters, and adapters to existing code.
- Prefer a clear, reasonable design over preserving the original code shape or achieving the smallest diff. Decide explicitly whether the existing flow still fits; if it does not, refactor the affected logic as part of the feature.
- Let concrete design problems justify refactoring: repeated policy or queries, fragmented state, unclear ownership, inconsistent entry points, or obsolete branches. Consolidate these at the proper boundary and remove superseded paths instead of layering new behavior on top.
- Keep the redesign proportional to the feature. Avoid speculative abstractions, unrelated cleanup, and broad rewrites; preserve unrelated user changes and established behavior, authorization, and lifecycle guarantees.
- Briefly explain the selected design and its reason before substantial changes. Verify both the new behavior and the existing invariants affected by the refactor.

## 2. Reuse without over-design

- Extract common preparation, policy, or transformation instead of duplicating it across entry points.
- Put stable behavior used by multiple scenarios in a focused service, domain component, or utility class.
- Put stable, stateless, business-agnostic capabilities such as encryption primitives, encoding, hashing, ID generation,
  and generic conversions in a focused utility class under the appropriate common or shared module. Keep business
  validation, configuration interpretation, error-code mapping, and scenario orchestration in the caller.
- Keep scenario-specific behavior in that scenario's module even when another scenario looks similar.
- Do not turn one-off expressions, stateful dependencies, or scenario workflows into utility methods merely to shorten a
  class; extract a utility only when the capability has a stable, reusable contract.
- Prefer one precise query or update over multiple calls, full-table reads, or in-memory filtering.
- Pass known data through the call chain instead of querying or converting it again.

## 3. Names, values, and types

- Use full, stable, domain-oriented names: nouns for types and files, verbs for methods, and affirmative predicates for booleans.
- Use one vocabulary for each concept; role suffixes such as `Request`, `Entity`, `Service`, or `Util` must add information.
- Use the precise type at the boundary when the contract is known; do not receive a broad type merely to convert it immediately.
- Avoid magic values. Use uppercase snake case constants for stable literals whose names add meaning.
- Keep scenario-owned constants near that scenario; place genuinely shared literals in a focused common constants class.
- Use an enum when values form a closed, meaningful set. Do not use raw strings or integers where an enum expresses the domain better.
- Keep external input, output, domain, persistence, and configuration models distinct when their trust or lifecycle differs.

### Class structure

- Prefer independent, named top-level classes in separate source files. Avoid inner or nested classes, including static nested configuration, model, and helper classes; having only one caller is not by itself a reason to nest a class.
- Place each class in the package that owns its responsibility and keep dependencies explicit instead of relying on an enclosing instance. Keep a nested class only when a concrete language or framework constraint makes it necessary.

## 4. Framework and dependencies

- Prefer framework annotations for component registration, dependency injection, validation, transactions, mapping, and other standard lifecycle behavior.
- Prefer constructor injection with immutable dependencies, using established annotation support to avoid manual boilerplate; avoid mutable field injection.
- Use builders or fluent construction when they improve scanning. Do not add annotations or abstractions that hide important behavior.
- Keep controllers and other entry points thin: receive, validate, delegate, and translate.

### Database boundary

- In MyBatis-Plus projects, business services inject the project's `IService<Entity>` abstraction and prefer its type-safe Lambda APIs; they do not call Mapper directly.
- Mapper usage stays inside the database service implementation, typically `ServiceImpl<Mapper, Entity>`.
- Let the database manage audit fields such as `create_time` and `update_time`; Java and ORM updates must not overwrite database-managed values.
- Keep business times such as finish time or last-run time under explicit business control.
- Follow [java-spring-mybatis.md](java-spring-mybatis.md) for the concrete `db` tree, Service split, file responsibilities, and query/update rules.

## 5. Control flow, errors, and logging

- Give each method one observable responsibility and keep one abstraction level within it.
- Use only necessary guard clauses, then show the happy path. Avoid deep nesting and hidden fallback behavior.
- Extract a helper for repeated policy or a meaningful operation, not merely to shorten a method.
- Use specific application errors for expected failures and follow the project's central response contract.
- Log actionable context with parameterized fields and exception objects; never log secrets or sensitive payloads.
- Do not catch an exception only to ignore it, wrap it without adding meaning, or return an ambiguous `null`.

## 6. Comments

- Comments explain business intent, constraints, or reasons, not obvious statements.
- In Chinese-first projects, write business comments in Chinese and preserve English technical names.
- Document every Service interface method at the contract source; do not repeat that comment on its `@Override`.
- Add a concise comment above every private helper in a Service implementation.
- Add a field comment above every persistent database Entity field.

## 7. Formatting

- Repository formatters and enforced rules take precedence. Otherwise use four spaces for Java-like languages and two for common web/configuration files.
- Keep ordinary lines near 120 characters, but favor compact, readable grouping over mechanical vertical formatting.
- In method declarations and calls, do not break immediately after `(` and do not put every parameter or argument on its own line.
- Keep the declaration or call on one line when reasonable. When it is long, wrap by logical groups and keep related parameters together on the same line.
- Keep the closing `)` with the last parameter or expression when practical. Break fluent chains by logical step.
- Use blank lines between logical stages, not between tightly related statements.
- Use explicit imports, remove unused imports, and follow the repository's import order.

## 8. Tests and final check

- Name tests after observable behavior and expected outcomes; keep each test deterministic and focused.
- Prefer real objects and narrow unit or slice tests; mock only external or expensive boundaries.
- Add regression coverage when practical. Never add empty tests, arbitrary sleeps, local paths, or private-environment dependencies.
- Verify the narrowest affected scope, then check the diff for broken references, duplication, magic values, excessive splitting, misplaced files, secrets, and unrelated changes.
