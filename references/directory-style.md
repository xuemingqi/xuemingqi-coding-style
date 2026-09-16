# Refined Directory Style

## 1. Respect the repository's organization axis, then responsibility

- First identify whether the repository is organized by technical responsibility, business scenario, or module. Preserve that established axis unless reorganization is explicitly requested.
- In a responsibility-first repository, put controllers in `controller`, Service contracts in `service`, implementations in `service/impl`, configuration in `config` or `properties`, and models in their established model packages. Do not place all files for one feature in a catch-all package such as `authentication`.
- In a scenario-first repository, use the business scenario, domain, or module as the primary boundary, then split only the responsibilities that scenario actually needs.
- Put each file in the narrowest package that accurately owns its behavior.
- Preserve an existing repository's root structure unless reorganization is part of the task.

Typical responsibility names include `api`, `controller`, `service`, `service/impl`, `dto`, `entity`, `db`, `mapper`, `config`, `constants`, `enums`, `exception`, and `util`. Choose one vocabulary for each role.

In Java Spring services that use MyBatis-Plus, keep persistence infrastructure under `db`, with `entity`, `enums`, `mapper`, `service`, and `service/impl` as needed. Keep business contracts in the module-level `service` package and their implementations in `service/impl`; do not mix the two Service layers. See [java-spring-mybatis.md](java-spring-mybatis.md) for file-level rules.

## 2. Keep packages cohesive

- Keep an interface and implementation close; use `impl` only when the interface is a meaningful boundary.
- Separate request, response, domain, and persistence models when their fields or exposure rules differ.
- Keep scenario policy, constants, enums, configuration, and helpers inside the owning scenario.
- Split a package or file only when it contains responsibilities that change independently.
- Do not create a standalone interface, Service, or store for behavior used only by one cohesive Service unless it has an independent contract, lifecycle, or realistic reuse.
- Avoid excessive class extraction, directory depth, repeated module names, and empty template packages.

## 3. Promote shared code carefully

- Move behavior to shared scope when multiple independent scenarios need the same stable implementation, not merely because code looks similar.
- Use a focused utility or shared package name such as `json`, `time`, `validation`, or `filesystem`; do not create vague dumping grounds.
- Put genuinely shared literals in a focused constants class and shared closed value sets in an enum.
- Keep module-specific policy local even if another module has a similar rule.
- Avoid circular sharing between modules.

## 4. Project hygiene

- Mirror production paths in tests where practical.
- Keep generated code, build output, fixtures, and local runtime data outside hand-written production packages.
- Avoid ambiguous buckets such as `misc`, `temp`, `other`, `common2`, or `new`.
