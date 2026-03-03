# General Rules
- Never use `any` for type declarations — always create proper type interfaces
- Never coauthor git commits and never do git commits without explicit permission
- Always give `nvim` commands — never nano, vi, or other editors
- Never run dangerous or destructive commands on the local machine that may damage the system or data (e.g., `rm -rf /`, `mkfs`, `dd`, dropping production databases)

---

# Planning & Architecture

## Planning Before Coding
- Always create a clear plan before writing any code — identify modules, data flow, dependencies, and edge cases upfront
- Break down large tasks into small, well-defined subtasks before implementation
- Identify potential risks, blockers, and unknowns early — address them in the plan
- Document architectural decisions and their rationale (ADRs) for non-trivial choices
- Define acceptance criteria and expected behavior before coding starts
- Consider backward compatibility, migration paths, and rollback strategies in the plan

## Software Design Principles — KISS, DRY, YAGNI, SOLID

### KISS (Keep It Simple, Stupid)
- Prefer the simplest solution that meets requirements — avoid clever or overly abstract code
- Write code that reads like prose — a new developer should understand it without comments
- Avoid premature optimization — profile first, optimize only proven bottlenecks
- Prefer flat code over deeply nested structures — early returns over nested if/else chains
- One function should do one thing — if a function name needs "and" in it, split it
- Avoid unnecessary abstractions — three similar lines of code is better than a premature helper

### DRY (Don't Repeat Yourself)
- Extract repeated logic into reusable functions, utilities, or shared modules
- Centralize configuration, constants, and magic values — never scatter them across files
- Use shared DTOs, interfaces, and types across layers — don't redefine the same shape in multiple places
- Centralize error messages and validation rules — single source of truth for user-facing text
- If you copy-paste code more than twice, refactor it into a shared abstraction
- DRY applies to knowledge, not just code — avoid duplicating business logic across services

### YAGNI (You Aren't Gonna Need It)
- Only implement what is currently required — never code for hypothetical future needs
- Don't add configuration options, feature flags, or extensibility points "just in case"
- Remove dead code, unused imports, and commented-out blocks — they are noise, not insurance
- Don't build frameworks — build features. Extract a framework only when the pattern has proven itself 3+ times
- Avoid gold-plating — if the requirement says "list users," don't add sorting, filtering, and pagination unless asked

### SOLID Principles
- **Single Responsibility (SRP)**: Each class/module should have one reason to change — one responsibility, one owner
- **Open/Closed (OCP)**: Design modules to be extended without modifying existing code — use interfaces, plugins, strategy pattern
- **Liskov Substitution (LSP)**: Subtypes must be substitutable for their base types without breaking behavior — honor contracts
- **Interface Segregation (ISP)**: Prefer many small, focused interfaces over one fat interface — clients should not depend on methods they don't use
- **Dependency Inversion (DIP)**: Depend on abstractions (interfaces), not concrete implementations — inject dependencies, don't instantiate them

---

# Modular Programming & Code Architecture

## Modularity
- Structure code into self-contained, loosely coupled modules with clear boundaries
- Each module should have a well-defined public API — hide internal implementation details
- Follow feature-based or domain-based folder structure — group by business capability, not by technical layer
- Keep module dependencies unidirectional — avoid circular imports
- Use barrel files (`index.ts`) to control what each module exports — explicit public surface area
- Separate concerns: controllers handle HTTP, services handle business logic, repositories handle data access

## ACID Rules (for Transactions & Data Integrity)
- **Atomicity**: Wrap multi-step operations in transactions — either all steps succeed or all roll back
- **Consistency**: Validate data integrity constraints at both application and database level — never leave data in a partial state
- **Isolation**: Use appropriate transaction isolation levels — prevent dirty reads, phantom reads, and lost updates
- **Durability**: Ensure committed data survives crashes — use write-ahead logs, journaling, or database-native durability guarantees
- For MongoDB: use multi-document transactions when atomicity is needed across collections
- For distributed systems: implement saga patterns or event sourcing when ACID across services isn't feasible
- Always handle transaction rollback in error/catch blocks — never leave open transactions

## Design Patterns
- Use design patterns where they naturally fit — don't force patterns where a simple function suffices
- **Repository Pattern**: Abstract database access behind repository interfaces — services should not know about query syntax
- **Strategy Pattern**: Use for interchangeable algorithms (e.g., payment processors, notification channels) — swap implementations without changing consumers
- **Factory Pattern**: Use for complex object creation with multiple configurations — centralize instantiation logic
- **Observer/Event Pattern**: Use for decoupled communication between modules — emit events instead of direct calls for cross-cutting concerns
- **Singleton Pattern**: Use sparingly and only for truly global shared state (e.g., database connections, config) — prefer dependency injection
- **Builder Pattern**: Use for constructing complex objects step-by-step — especially useful for test data setup
- **Decorator Pattern**: Use for adding behavior without modifying existing code — logging, caching, retry wrappers
- **Middleware Pattern**: Use for request/response pipelines — authentication, logging, validation, rate limiting
- **Circuit Breaker Pattern**: Use for external service calls — prevent cascade failures when a dependency is down
- **Adapter Pattern**: Use to integrate third-party services — wrap external APIs behind your own interface so you can swap providers

---

# Code Quality & Readability

## Naming Conventions
- Use descriptive, intention-revealing names — `getUsersByRole()` not `getUsers2()` or `fetch()`
- Use consistent casing: `camelCase` for variables/functions, `PascalCase` for classes/interfaces/types, `UPPER_SNAKE_CASE` for constants
- Boolean variables should read as questions: `isActive`, `hasPermission`, `canEdit` — never `active`, `flag`, `check`
- Avoid abbreviations unless universally understood (`id`, `url`, `api` are fine; `usr`, `mgr`, `ctx` are not)
- Name functions as verbs (`createUser`, `validateInput`), name variables as nouns (`userList`, `maxRetryCount`)
- File names should match the primary export — `user.service.ts` exports `UserService`

## Code Structure & Readability
- Keep functions short — ideally under 30 lines; if longer, break into well-named sub-functions
- Keep files focused — ideally under 300 lines; if longer, split into separate modules
- Maximum 3 levels of nesting — use early returns, guard clauses, and extracted functions to flatten logic
- Use meaningful whitespace — group related lines, separate logical blocks with blank lines
- Order code top-down: public methods first, then private helpers — readers should see the "what" before the "how"
- Prefer declarative code over imperative: `.map()/.filter()/.reduce()` over manual for-loops where appropriate
- Use enums or const objects for fixed value sets — never use magic strings or numbers
- Add comments only for "why," not "what" — the code should explain what it does; comments explain non-obvious decisions

## Type Safety
- Never use `any` — create proper interfaces and types for every data shape
- Use strict TypeScript configuration: `strict: true`, `noImplicitAny: true`, `strictNullChecks: true`
- Define explicit return types on public functions and API boundaries
- Use discriminated unions for state modeling — prefer `{ status: 'loading' } | { status: 'success', data: T } | { status: 'error', error: Error }` over loose objects
- Use generics to create reusable, type-safe utilities — avoid duplicating logic for different types
- Prefer `unknown` over `any` when the type is truly unknown — force explicit type narrowing

---

# Error Handling & Exception Management

## General Error Handling
- Every async operation must have proper error handling — never leave unhandled promise rejections
- Use typed/custom error classes with error codes — `throw new AppError('USER_NOT_FOUND', 404)` not `throw new Error('not found')`
- Implement a global error handler as a safety net — catch unhandled exceptions and unhandled rejections at the process level
- Use try/catch at service boundaries — controllers catch service errors and translate to HTTP responses
- Never swallow errors silently (`catch (e) {}`) — at minimum log them
- Distinguish between operational errors (expected: validation, not found, timeout) and programmer errors (unexpected: null reference, type error) — handle them differently
- Return meaningful error responses to clients: consistent error shape with `code`, `message`, and optional `details`
- Never expose internal error details, stack traces, or database errors to the client in production
- Use error boundaries in React/Next.js to prevent entire UI crashes from component errors
- Implement retry logic with exponential backoff for transient failures (network, rate limits) — but set a max retry count
- Always clean up resources in `finally` blocks — close connections, release locks, clear timers

## Validation Errors
- Validate at system boundaries: API endpoints, form submissions, external data ingestion
- Return all validation errors at once — don't make users fix one field at a time
- Use schema validation libraries (class-validator, Zod, Joi) — don't write manual if-checks for validation
- Validate both shape (type, required fields) and semantics (range, format, business rules)

---

# Logging & Observability

## Logging Levels
- **ERROR**: Unrecoverable failures that need immediate attention — failed transactions, data corruption, service crashes
- **WARN**: Recoverable issues that indicate something is wrong — deprecated API usage, fallback triggered, retry attempts, approaching rate limits
- **INFO**: Key business events and lifecycle milestones — user registered, order placed, service started, job completed
- **DEBUG**: Detailed diagnostic information for development — request/response payloads, intermediate state, query details (disabled in production)
- **TRACE/VERBOSE**: Ultra-detailed step-by-step execution flow — only used during active debugging sessions (never in production)

## Logging Best Practices
- Use structured logging (JSON format) — enables parsing, searching, and alerting in log aggregation tools
- Include correlation/request IDs in every log entry — trace a request across services
- Log at function entry/exit points for critical operations — but not for every function (avoid log spam)
- Never log sensitive data: passwords, tokens, PII, credit card numbers — mask or redact them
- Include context in log messages: user ID, request ID, operation name, relevant entity IDs — logs without context are useless
- Use a centralized logging library configured once — never use raw `console.log` in production code
- Configure log levels per environment: `DEBUG` in dev, `INFO` in staging, `WARN`+`ERROR` in production
- Implement log rotation and retention policies — unbounded logs will fill disks
- Add request duration logging for API endpoints — identify slow endpoints without dedicated APM
- Log the outcome of retries and circuit breaker state changes — critical for diagnosing intermittent failures

---

# Edge Cases & Defensive Programming

## Edge Case Handling
- Handle null, undefined, empty strings, empty arrays, and zero values explicitly — don't rely on truthy/falsy coercion for business logic
- Handle boundary conditions: first item, last item, single item, maximum size, empty collection
- Handle concurrent access: race conditions, duplicate submissions, optimistic locking conflicts
- Handle time-related edge cases: timezone differences, DST transitions, leap years, date boundaries
- Handle unicode and special characters in user input — emoji, RTL text, zero-width characters
- Handle pagination edge cases: page beyond total, page size of 0, negative page numbers
- Handle network edge cases: partial responses, connection resets, DNS failures, SSL certificate errors
- Test with extreme inputs: very large strings, deeply nested objects, maximum integer values
- Handle graceful degradation: if a non-critical feature fails, the core application should continue working

## Defensive Programming
- Validate function arguments at public API boundaries — fail fast with clear error messages
- Use TypeScript strict mode and linting to catch issues at compile time rather than runtime
- Prefer immutable data structures — use `readonly`, `const`, `Object.freeze()` where appropriate
- Use exhaustive switch statements with `never` type for discriminated unions — catch unhandled cases at compile time
- Set sensible defaults for optional parameters — don't assume callers will always provide them
- Use assertion functions for invariants that should never be violated — `assertNonNull()`, `assertValidState()`

---

# Database — MongoDB

## Schema & Data Modeling
- Use denormalized collections for optimized read queries — accept duplicate data across collections when it avoids expensive joins/lookups
- Embed frequently accessed related data within the parent document — optimize for the most common query patterns
- Use references (ObjectId) only when the related data is large, changes independently, or is accessed separately
- Design schemas around query patterns, not around entity relationships — MongoDB is not a relational database
- Define Mongoose schemas with strict validation: required fields, types, enums, min/max constraints
- Use indexes on all fields that appear in query filters, sort, or unique constraints — monitor slow queries and add compound indexes as needed
- Implement soft deletes (`deletedAt` timestamp) for important business data — hard deletes are irreversible
- Use `timestamps: true` on all schemas — `createdAt` and `updatedAt` are invaluable for debugging and auditing
- Add `versionKey` (`__v`) for optimistic concurrency control on documents with concurrent updates

## Query Optimization
- Use `.lean()` for read-only queries — skips Mongoose hydration and returns plain objects (much faster)
- Use `.select()` to project only needed fields — don't fetch entire documents when you need two fields
- Use aggregation pipelines for complex queries — prefer `$match` early in the pipeline to reduce documents processed
- Implement pagination with cursor-based approach (`_id` > lastId) for large collections — avoid `skip/limit` on large offsets
- Use `bulkWrite()` for batch operations — never loop with individual `save()` calls
- Monitor and profile slow queries — add explain plans for queries taking over 100ms

---

# Elasticsearch

## Usage
- Use Elasticsearch for any search requiring full-text search across multiple fields or on text content
- Use Elasticsearch as vector storage for semantic search — store embeddings alongside documents
- Keep Elasticsearch in sync with the primary database — use change streams, events, or scheduled sync jobs
- Define explicit mappings for all indices — never rely on dynamic mapping in production
- Use proper analyzers for text fields: standard for general text, keyword for exact match, custom analyzers for language-specific needs
- Implement index aliases for zero-downtime reindexing — never query indices directly in application code

## Search Best Practices
- Use `bool` queries to combine `must`, `should`, `filter`, and `must_not` clauses — `filter` context for non-scoring filters (faster, cached)
- Use `multi_match` for searching across multiple fields — configure field boosting for relevance
- Implement search suggestions with completion suggesters or edge n-grams
- Use `knn` search for vector/semantic queries — configure appropriate `num_candidates` and `k` values
- Paginate with `search_after` for deep pagination — avoid `from/size` beyond 10,000 results
- Set appropriate `index.refresh_interval` — real-time search needs 1s, analytics can use 30s+

---

# Caching Strategy

## Smart Caching
- Implement caching at every expensive boundary: 3rd-party APIs, LLM calls, database queries, computed results
- Use Redis for distributed caching across service instances — in-memory cache (e.g., `node-cache`) for single-instance data
- Define TTL (time-to-live) based on data volatility: static config = hours/days, user data = minutes, real-time data = seconds
- Use cache-aside (lazy loading) pattern as default — populate cache on miss, return from cache on hit
- Implement cache invalidation on data mutation — stale data is worse than no cache
- Use cache key namespacing: `{service}:{entity}:{id}:{version}` — avoid key collisions across modules
- Cache entire API responses for idempotent GET requests — use `ETag` or `Last-Modified` headers for HTTP caching
- For LLM calls: cache prompt+response pairs keyed by a hash of the input — identical prompts should return cached results
- Implement cache warming for critical data on service startup — avoid cold-start latency spikes
- Set memory limits on Redis — use eviction policies (`allkeys-lru`) to prevent OOM
- Log cache hit/miss ratios — a low hit ratio means your caching strategy needs adjustment
- Use multi-level caching where appropriate: L1 (in-memory, fastest) → L2 (Redis, shared) → L3 (database, source of truth)

---

# Third-Party API Integration

## Timeouts & Resilience
- ALWAYS set explicit timeouts on every 3rd-party API call — never allow unbounded waiting
- Use reasonable timeout values: connect timeout (5s), read timeout (10-30s depending on the API)
- Implement circuit breaker pattern for external service calls — open circuit after N consecutive failures, retry after cooldown
- Use exponential backoff with jitter for retries — `baseDelay * 2^attempt + randomJitter`
- Set maximum retry attempts (typically 3) — don't retry indefinitely
- Implement bulkhead pattern: isolate external service calls so one slow API doesn't exhaust all connections
- Use connection pooling for HTTP clients — reuse TCP connections for repeated calls to the same service
- Log all external API call durations — identify degradations before they become outages
- Implement fallback strategies: cached response, default value, degraded functionality — don't let an external failure crash your service
- Wrap all 3rd-party SDKs behind your own adapter interface — swap providers without changing business logic

## Rate Limiting & Quotas
- Track and respect rate limits of all external APIs — implement local rate limiting to stay under quotas
- Use token bucket or sliding window algorithms for outgoing request rate limiting
- Queue and batch API calls where possible — reduce total call count
- Implement request deduplication — don't call the same API twice for the same data in the same request cycle

---

# Frontend / UI

## Cold Start & Mock Data
- To solve the cold start problem in UI, show mock data when the project is running in dev mode
- Add a `USE_MOCK_DATA` flag (or equivalent) in `.env` to switch between mock and live data
- Mock data should be realistic and representative — use factories or fixtures, not `{ name: 'test' }`
- Keep mock data files co-located with the features they serve — `__mocks__/` directory next to the module
- Mock data should cover edge cases: empty states, error states, maximum-length content, special characters
- Never ship mock data to production — use environment-based conditional loading
- Use MSW (Mock Service Worker) or similar for intercepting API calls in development — mock at the network level, not inside components

## UI Best Practices
- Implement loading states, error states, and empty states for every data-fetching component
- Use skeleton screens instead of spinners for better perceived performance
- Implement optimistic updates for user actions — update UI immediately, reconcile with server response
- Handle offline/slow network gracefully — queue actions, show indicators, retry on reconnection
- Implement proper form validation with real-time feedback — don't wait for submission to show errors

---

# Testing

## Testing Strategy
- Write unit tests for business logic and utility functions — test pure functions with known inputs/outputs
- Write integration tests for API endpoints — test the full request/response cycle including validation, auth, and database
- Write e2e tests for critical user flows — login, checkout, data creation workflows
- Use descriptive test names that document behavior: `should return 403 when user lacks admin role` not `test auth`
- Follow AAA pattern: Arrange (setup), Act (execute), Assert (verify) — one act per test
- Use factories/builders for test data — never hardcode test data inline
- Mock external dependencies (APIs, databases) in unit tests — use real instances in integration tests
- Aim for meaningful coverage, not 100% — focus on business logic, edge cases, and error paths
- Run tests in CI on every PR — never merge with failing tests

---

# Dependencies & Package Management

## Package Selection
- Only use open-source packages from NPM or GitHub that are safe and don't have known vulnerabilities
- Verify packages have a good number of downloads (1k+ weekly for niche, 10k+ for core dependencies)
- Check package maintenance: last publish date, open issues count, number of contributors
- Prefer packages backed by organizations or well-known maintainers over anonymous single-dev packages
- Read the package's `package.json` and check for postinstall scripts — malicious packages use these as attack vectors
- Use `npm audit` or `yarn audit` before adding new dependencies — fix critical/high vulnerabilities immediately
- Pin exact versions in `package.json` for critical dependencies — use lockfiles (`package-lock.json` / `yarn.lock`) always
- Minimize dependency count — every dependency is a supply chain risk. If the functionality is 10 lines of code, write it yourself

---

# Security Rules

## Secrets & Environment Variables
- NEVER commit `.env`, `.env.*`, or any file containing secrets (API keys, tokens, passwords, connection strings)
- NEVER hardcode secrets, credentials, or API keys in source code — always use environment variables
- NEVER log secrets, tokens, or passwords — even in debug mode
- Before any git add/commit, verify no `.env` or secret files are staged (`git status`)
- If a secret is accidentally committed, treat it as compromised — rotate it immediately, don't just remove the file

## Authentication & Authorization
- Always validate and verify JWT tokens server-side — never trust client-side token claims
- Check JWT expiration (`exp`), issuer (`iss`), and audience (`aud`) on every protected route
- Use short-lived access tokens with refresh token rotation
- Never store JWTs in localStorage — prefer httpOnly secure cookies
- Always implement proper role-based access control (RBAC) — check permissions on every API endpoint, not just the frontend
- Never expose user IDs or internal object IDs in URLs without authorization checks (IDOR prevention)

## Input Validation & Injection Prevention
- Sanitize and validate ALL user input on the server side — never trust frontend validation alone
- Use parameterized queries / ORM methods — never concatenate user input into SQL queries
- Escape output to prevent XSS — sanitize HTML, use Content-Security-Policy headers
- Validate file uploads: check MIME type, file size, and extension — never trust the client-provided filename
- Rate-limit authentication endpoints and sensitive APIs to prevent brute force

## Build & Deploy
- Always run `npm run build` (or equivalent) and verify it passes before committing
- Always run `npm run lint` before committing — fix all warnings and errors
- Never commit with `--no-verify` — pre-commit hooks exist for a reason
- Never push directly to `main`/`master` without a PR review
- Never include `node_modules`, `dist`, `build`, or generated artifacts in commits
- Check for known vulnerabilities with `npm audit` periodically

## API & Network Security
- Always use HTTPS — never HTTP for any external communication
- Set proper CORS configuration — never use `origin: '*'` in production
- Add security headers: `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`
- Never expose stack traces, internal errors, or debug info in API responses in production
- Implement request size limits to prevent payload-based DoS

## Database & Data Security
- Never expose raw database errors to the client — use generic error messages
- Apply principle of least privilege for database credentials
- Never store passwords in plain text — always use bcrypt/argon2 with proper salt rounds
- Sanitize data before rendering to prevent stored XSS

## Dependencies Security
- Never install packages from untrusted or typo-squatted sources — verify package names
- Review new dependencies before adding — check download counts, maintainers, and last update
- Keep dependencies updated — outdated packages are a common attack vector
- Prefer well-maintained packages with active security disclosure processes

## NestJS Security
- Always use `ValidationPipe` globally with `whitelist: true` and `forbidNonWhitelisted: true` — strip unknown properties automatically
- Use class-validator decorators (`@IsString()`, `@IsEmail()`, `@MaxLength()`, etc.) on every DTO — never accept raw request bodies
- Always use Guards (`@UseGuards()`) for authentication/authorization — never check auth manually inside controllers
- Use `@Roles()` decorator + `RolesGuard` for RBAC — never rely on frontend role checks alone
- Always apply `ThrottlerGuard` on auth routes (`/login`, `/register`, `/forgot-password`) — configure globally via `ThrottlerModule`
- Use `helmet` middleware (`app.use(helmet())`) — sets security headers automatically
- Enable CSRF protection for cookie-based auth sessions
- Use `@Exclude()` from class-transformer on sensitive entity fields (password, tokens) — never return them in responses
- Always use `ConfigService` to access env vars — never use `process.env` directly in services/controllers
- Set `app.enableCors()` with explicit `origin`, `methods`, and `credentials` — never leave defaults
- Use `ParseIntPipe`, `ParseUUIDPipe`, etc. on route params — never trust raw param types
- Implement global exception filters to catch and sanitize all unhandled errors — never leak stack traces
- Use `@SerializeOptions()` or interceptors to control what gets sent in responses
- Always scope database queries to the authenticated user — never trust user-supplied IDs without ownership checks
- Use `bcrypt` (10+ salt rounds) or `argon2` in auth services — never roll custom hashing

## Next.js Security
- NEVER expose secrets in client code — only use `NEXT_PUBLIC_` prefix for truly public values
- Sensitive keys (DB, API secrets, JWT secrets) must ONLY be in server-side code (`getServerSideProps`, API routes, Server Components, Route Handlers)
- Always validate and sanitize inputs in API routes (`/app/api/` or `/pages/api/`) — they are public endpoints
- Use `next-auth` or similar for auth — never roll custom session management from scratch
- Protect API routes and Server Actions with auth checks — middleware alone is not enough, validate in each handler
- Always use `NextResponse` with proper status codes and generic error messages — never expose internal errors
- Set security headers in `next.config.js` using the `headers()` config — CSP, X-Frame-Options, HSTS, etc.
- Use `middleware.ts` for route protection but ALWAYS re-validate auth server-side — middleware can be bypassed
- Never use `dangerouslySetInnerHTML` unless absolutely necessary — if needed, sanitize with DOMPurify first
- Always use Next.js `<Image>` component — prevents image-based XSS and enforces domains via `remotePatterns`
- Configure `remotePatterns` in `next.config.js` — never allow arbitrary external image domains
- Use Server Components by default — minimize `'use client'` to reduce client-side attack surface
- In Server Actions, always re-authenticate and re-authorize — never trust the caller
- Validate `redirect()` targets — never redirect to user-supplied URLs without allowlisting
- Use `nonce`-based CSP for inline scripts when needed — configure in middleware
- Never commit `.next/` folder — it may contain build-time secrets in server bundles
- Set `poweredByHeader: false` in `next.config.js` — don't advertise the framework
- Use `httpOnly`, `secure`, `sameSite: 'strict'` flags on all cookies
