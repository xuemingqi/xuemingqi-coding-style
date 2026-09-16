# Java Spring and MyBatis-Plus Style

Use these rules for Java Spring modules backed by MyBatis or MyBatis-Plus. Treat neighboring code and enforced repository rules as higher-priority evidence, but do not copy obvious historical inconsistencies.

## 1. Keep the two Service layers distinct

- Put business contracts directly in `service`; name them `<Capability>Service`.
- Put business implementations in `service/impl`; name them `<Capability>ServiceImpl`, annotate them with `@Service`, and implement the matching interface.
- Put database CRUD contracts in `db/service`; name them `<Entity>IService` and extend `IService<Entity>`.
- Put database CRUD implementations in `db/service/impl`; name them `<Entity>IServiceImpl`, annotate them with `@Service`, extend `ServiceImpl<EntityMapper, Entity>`, and implement `<Entity>IService`.
- Keep orchestration, validation, authorization, transactions, conversions, and calls to other systems in the business Service implementation. Keep the database Service thin and persistence-focused.
- Do not omit a business Service interface merely because there is currently one implementation. The interface is the module contract and the implementation belongs under `impl`.
- Inject dependencies through `private final` fields and constructor injection, normally with `@RequiredArgsConstructor`. Do not add new `@Resource` or mutable field injection.

Typical layout:

```text
<module>/
├── controller/
├── service/
│   ├── OrderService.java
│   └── impl/
│       └── OrderServiceImpl.java
└── db/
    ├── entity/
    │   └── Order.java
    ├── enums/
    │   └── OrderStatus.java
    ├── mapper/
    │   └── OrderMapper.java
    └── service/
        ├── OrderIService.java
        └── impl/
            └── OrderIServiceImpl.java
```

Create only the packages the module actually needs.

## 2. Define every `db` file by one responsibility

### `db/entity/<Entity>.java`

- Use a singular domain noun without an `Entity` suffix when that is the module convention.
- Map the table explicitly with `@TableName`, the primary key with `@TableId`, and persistent columns with `@TableField` when explicit mapping improves safety and schema visibility.
- Keep field types aligned with the schema. Use a persistence enum instead of a raw integer or string for a closed coded column.
- Add a concise Chinese Javadoc comment above every persistent field. State units, null meaning, ownership, or coded values when they matter.
- Mark database-managed audit fields such as `create_time` and `update_time` so inserts and updates never overwrite them, for example with `FieldStrategy.NEVER`. Do not apply that rule to business-managed timestamps.
- Keep entities persistence-only: no controller response shape, remote API contract, repository query, or business orchestration.
- Use Lombok consistently with neighboring entities. `@Data` and chain accessors are acceptable; add builders and constructors only when construction patterns require them.

### `db/enums/<Value>.java`

- Store enums tied to database-coded columns next to the entities under `db/enums`.
- Give each constant a stable code and a clear description. Keep fields immutable.
- Mark the persisted value with `@EnumValue`. Add `@JsonValue` only when the same code is intentionally the external JSON representation; persistence mapping alone does not authorize API exposure.
- Provide a deliberate conversion method when external input must be decoded. Reject unknown values through the project error contract instead of silently selecting a default.

### `db/mapper/<Entity>Mapper.java`

- Extend `BaseMapper<Entity>` and rely on package-level `@MapperScan`; do not add repetitive `@Mapper` annotations when scanning already covers the package.
- Keep the Mapper empty for standard CRUD. Add a method only for a query that is genuinely clearer or more efficient as custom SQL, such as a complex join, aggregate, or database-specific bulk operation.
- Do not put business decisions, authorization, DTO conversion, or cross-service orchestration in a Mapper.

### `db/service/<Entity>IService.java`

- Extend `IService<Entity>` and act as the persistence boundary exposed to business Services.
- Keep the inherited CRUD and Lambda API as the default surface. Add custom methods only for reusable, entity-owned persistence behavior that cannot be expressed cleanly at the caller.
- Do not turn the database Service into a second business Service.

### `db/service/impl/<Entity>IServiceImpl.java`

- Extend `ServiceImpl<EntityMapper, Entity>` and implement the exact matching `<Entity>IService` name.
- Keep an empty implementation when inherited behavior is sufficient.
- Use `baseMapper` only here, and only for custom persistence operations declared by the database Service contract. Never expose the Mapper to callers.

### SQL schema or migration files

- Keep hand-maintained DDL or migrations under the repository's resources convention, such as `src/main/resources/db`.
- Use lowercase snake case for tables, columns, indexes, and constraints. Give tables and columns meaningful comments.
- Declare primary keys, unique business constraints, lookup indexes, nullability, defaults, character set, and timestamp ownership explicitly.
- Make Java entity types, enum codes, audit-field strategy, and schema definitions agree. Change them together.
- Do not make application startup silently create or mutate production schema unless the repository explicitly uses a migration tool for that purpose.

### Optional Mapper XML

- Create XML only when custom SQL is justified. Match its namespace to the Mapper interface and method IDs to Mapper method names.
- Keep result mappings explicit when names or types do not map safely. Bind values with parameters; never concatenate untrusted input into SQL.
- Keep SQL concerned with data retrieval or mutation, not presentation or business workflow.

## 3. Call persistence through `IService` Lambda APIs

- Inject `<Entity>IService` into business Service implementations. Do not inject or call `<Entity>Mapper` from controllers, business Services, schedulers, consumers, or tools.
- Use `getById`, `save`, `saveBatch`, `updateById`, and `removeById` for primary-key and simple CRUD operations.
- Prefer `lambdaQuery()` and `lambdaUpdate()` for conditional operations so columns are referenced by entity method references rather than strings.
- Use conditional overloads such as `eq(condition, Entity::getField, value)` and `set(condition, Entity::getField, value)` to keep optional filters and patch updates linear.
- Finish fluent operations with the narrowest meaningful terminal operation: `one`, `list`, `exists`, `count`, `page`, `update`, or `remove`. Use `one` only when uniqueness is guaranteed.
- Perform filtering, sorting, projection, pagination, existence checks, and bulk mutation in the database. Avoid full-table reads, repeated queries, N+1 calls, and in-memory filtering when one precise operation suffices.
- Include tenant, user ownership, scope, and status predicates in the database operation when they define access. Do not fetch broadly and authorize after mutation.
- Use `lambdaUpdate()` for targeted conditional or patch updates. Use `updateById()` when updating a loaded entity is clearer. Update only intended fields.
- Check boolean or affected-row results when failure changes business behavior; do not add ceremonial checks when the surrounding contract already handles the outcome.
- Avoid raw SQL fragments such as `last(...)` in business code. If a database-specific query is necessary, keep the fragment constant and safe or move a meaningful custom operation behind the database Service.

Preferred shape:

```java
return orderIService.lambdaQuery()
        .eq(Order::getUserId, userId)
        .eq(status != null, Order::getStatus, status)
        .orderByDesc(Order::getUpdateTime)
        .page(new Page<>(page, pageSize));
```

```java
orderIService.lambdaUpdate()
        .eq(Order::getId, orderId)
        .eq(Order::getUserId, userId)
        .set(name != null, Order::getName, name)
        .update();
```

## 4. Keep business Service contracts clean

- Document every public method on the Service interface with its business effect, important constraints, parameters, and result. Do not repeat the same Javadoc on `@Override` methods.
- Give each implementation method one business operation. Use necessary guard clauses followed by a compact happy path.
- Add a concise comment above every private helper explaining its business role or constraint. Do not comment mechanics already visible from the method name and code.
- Place `@Transactional` on the business Service operation that owns multiple related database writes. Keep transactions short and remember that remote calls, files, messages, and schedulers are not rolled back by a database transaction.
- Convert Entity objects to domain or response models at the business boundary. Do not return persistent entities from controllers or remote APIs.
- Return empty collections rather than `null`. Use the project's typed exception and centralized response-code contract for expected failures.

## 5. Preserve model and entry-point boundaries

- Use DTO or request models for untrusted transport input and validate them at the controller or adapter boundary.
- Use domain or response models for business output. Use internal parameter or context models when a Service contract should not depend directly on a transport DTO.
- Keep database Entity, API DTO, domain output, configuration properties, and runtime state separate when exposure, trust, or lifecycle differs.
- Keep controllers thin: bind and validate input, obtain request context, delegate to a business Service, and wrap the result in the project response type.
- Do not place database calls, Entity mutation, transactions, or multi-step business policy in controllers.
- Put stable cross-scenario conversion in a focused converter; keep one-off conversion near the owning Service rather than creating a generic mapping bucket.

## 6. Review checklist

- Does the package distinguish business `service/impl` from `db/service/impl`?
- Does every implementation exactly match its interface name?
- Are business callers using `IService` instead of Mapper?
- Are conditional database operations using Lambda method references rather than column strings?
- Are Entity fields documented and audit timestamps protected from ORM writes?
- Are authorization predicates, filters, sorting, and pagination executed in the database?
- Are transaction boundaries located at the owning business operation?
- Are transport, domain, persistence, configuration, and runtime models kept distinct?
- Are historical field injection, inconsistent suffixes, raw SQL fragments, and duplicated comments avoided?
