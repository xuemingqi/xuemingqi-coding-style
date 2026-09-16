# Java Spring Web, Authentication, and Configuration Style

## 1. Keep trusted request identity out of controller plumbing

- Authenticate in an interceptor, filter, gateway, or equivalent trusted boundary, then place the authenticated user in the project's request or thread context.
- Keep controllers thin. Pass business input to the Service; do not extract `userId` merely to add it to every Service method.
- Let the business Service read the authenticated user through the established context utility such as `ServerUtil`.
- Do not repeat null checks after a mandatory authentication boundary has guaranteed that the user context exists.
- Keep an explicit identity parameter only when identity is genuine business input, such as an administrator acting on another user, an asynchronous job, or a boundary that does not participate in the normal interceptor chain.
- Authenticate explicitly at raw Servlet, proxy, streaming, or other entry points that bypass the MVC interceptor before reading user-scoped data.

## 2. Use JWT and shared Redis state for centrally managed distributed login

- For a multi-node service that needs revocation and shared user state, prefer a signed JWT plus Redis-backed token state over an in-memory HTTP Session.
- Use standard JWT claims with their standard semantics: `jti` for the token identifier, `iss` for issuer, `aud` for audience, `sub` for subject, and `iat`/`exp` for issued and expiry times. Do not replace standardized claim names merely to avoid abbreviations.
- Keep the JWT minimal. A signed JWT is not encrypted, so do not embed unnecessary personal data or the complete user object in it.
- Store the authenticated user information and revocation state in Redis under a key derived from `jti`. On each request, verify the JWT first, then load the authoritative user state from Redis.
- Align the Redis TTL with the token expiry. Logout deletes or invalidates the Redis state so a structurally valid JWT can no longer authenticate.
- Keep issuer, audience, signing configuration, Redis key prefixes, and header conventions consistent across every service that creates or verifies the token.

## 3. Prefer declarative HTTP clients

- Prefer OpenFeign for stable Spring service integrations instead of wrapping `RestClient` or manually repeating form submission, serialization, and error handling.
- Give the Feign interface a narrow remote contract and keep remote DTOs distinct from internal domain models.
- Translate remote transport and protocol failures once at the integration boundary. Do not leak Feign-specific exceptions into business code.
- Avoid dependency cycles between authentication, MVC configuration, interceptors, and Feign infrastructure. Keep remote-client creation independent from request authentication whenever possible.

## 4. Keep configuration under its owning domain

- Put ordinary project defaults directly in `application.yml` and profile-specific YAML unless repository deployment or secret-management rules require external configuration.
- Group configuration under the domain that owns it. For example, authentication configuration belongs under `authentication`, not under an unrelated `agent` namespace.
- Do not hard-code environment-dependent service addresses, redirect URLs, application identifiers, token validity, key prefixes, or protocol paths inside Service methods.
- Store a complete externally governed URL in configuration when composing it in code would duplicate fixed protocol policy. Bind it through a typed `@ConfigurationProperties` class.
- Keep real production secrets out of committed configuration even when non-secret local defaults are stored in YAML.

## 5. Prefer chainable POJOs for mutable models

- For mutable DTO, DO, configuration, and runtime-state models, prefer ordinary Lombok POJOs with `@Data` and `@Accessors(chain = true)` when the repository already uses Lombok.
- Construct these models with named chain setters so field meaning remains visible. Avoid positional construction such as `new AccessTokenState(userInfo, ticket, expiresAt)`.
- Use a Java `record` only when the type is intentionally immutable, has transparent value semantics, and benefits from a stable positional contract. Do not make records the default for external request/response models or mutable state.

## 6. Make time semantics explicit

- Follow the project's preference for `LocalDateTime` in business, persistence, and stored-state models when the value is interpreted in one documented zone.
- Keep `Instant` or epoch time at protocols that define an absolute moment, including standard JWT timestamps.
- Convert explicitly at the boundary using one declared zone, preferably UTC for token state. Never rely on the host's implicit default time zone for authentication expiry.

## 7. Remove authentication magic values

- Put shared authentication and protocol literals such as token type, authorization prefix, standard header names, Redis key prefixes, and shared parameter names in a focused common constants class.
- Derive related constants from one source, for example build the authorization prefix from the shared Bearer token type instead of repeating `"Bearer "`.
- Keep literals that belong to only one integration local to that integration. Do not turn a common constants class into a dumping ground.

## 8. Provide a local substitute for unreachable identity systems

- When the real SSO cannot be reached during local development, create a small isolated mock SSO application that implements only the required protocol endpoints and callback flow.
- A mock login page may accept any non-empty development credentials when its purpose is exercising redirects and token exchange, not testing identity validation.
- Keep mock behavior, ports, credentials, and permissive login strictly in development configuration. Make the production client switch by configuration rather than conditional business code.
- Verify the complete local flow: unauthenticated redirect, mock login, callback, token issuance, Redis user lookup, authenticated request, expiry, and logout.
