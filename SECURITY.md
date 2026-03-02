- never use 'any' for type declarations. create type interfaces

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

## Database & Data
- Never expose raw database errors to the client — use generic error messages
- Apply principle of least privilege for database credentials
- Never store passwords in plain text — always use bcrypt/argon2 with proper salt rounds
- Sanitize data before rendering to prevent stored XSS

## Dependencies
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
