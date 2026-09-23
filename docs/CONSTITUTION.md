# TaskMaster Pro Engineering Constitution
**Version:** 1.0.0

*(Assumption: As no project brief or name was provided, this constitution assumes the project is "TaskMaster Pro", a production-grade B2B SaaS task management platform built as a full-stack TypeScript monorepo.)*

## Mission
This document is the supreme engineering authority for TaskMaster Pro. It supersedes all other engineering guidelines, personal preferences, and AI-generated suggestions. Our mission is to build a highly reliable, scalable, and secure task management platform for enterprise teams, ensuring data integrity and seamless collaboration at scale.

## Core Values
1. **Correctness over speed:** It is better to ship late than to ship broken.
2. **Security over convenience:** Security is never compromised for developer or user friction.
3. **Simplicity over cleverness:** Code must be readable and maintainable by junior engineers.
4. **Maintainability over shortcuts:** Technical debt must be explicitly tracked and paid down.
5. **Observability over assumptions:** If it isn't measured, it doesn't exist.
6. **Explicitness over magic:** Avoid implicit behaviors and hidden side-effects.
7. **Automation over manual processes:** CI/CD and scripts must handle repetitive tasks.
8. **Testing over trust:** All code must prove it works through automated tests.

## Technology Stack

### Required Technologies
| Domain | Technology |
| :--- | :--- |
| **Language** | TypeScript (Strict Mode) |
| **Frontend** | React, Next.js, Tailwind CSS |
| **Backend** | Node.js, Express |
| **Database** | PostgreSQL, Prisma ORM |
| **Infrastructure** | AWS (ECS, RDS), Terraform |
| **Package Manager**| pnpm |

### Forbidden Technologies & Practices
* Plain JavaScript in application code (TypeScript is mandatory).
* Unmaintained dependencies (no commits in the last 12 months).
* Experimental libraries in production without explicit architecture approval.
* Direct database mutations from the frontend (must route through API).

## Repository Structure
The project utilizes a Turborepo-powered monorepo structure to share types and configurations across the stack.

```text
taskmaster-pro/
├── apps/
│   ├── web/                # Next.js frontend application
│   └── api/                # Node.js/Express backend API
├── packages/
│   ├── ui/                 # Shared React component library
│   ├── database/           # Prisma schema and generated client
│   ├── config/             # Shared ESLint, Prettier, TS configs
│   └── types/              # Shared TypeScript interfaces
├── infrastructure/         # Terraform configurations
└── .github/                # CI/CD workflows
```

## Language/Code Standards
* **Strict TypeScript:** `strict: true` and `noImplicitAny: true` are mandatory.
* **File Size Limits:** 300 lines target; 500 lines is a mandatory refactor threshold.
* **Naming Conventions:**
  * **Variables/Functions:** `camelCase`
  * **Classes/Components/Interfaces:** `PascalCase`
  * **Files/Directories:** `kebab-case` (e.g., `user-profile.tsx`)
  * **Constants/Enums:** `UPPER_SNAKE_CASE`

## Frontend Standards
* **State Management:** Use React Context for global UI state; use React Query (TanStack Query) for server state and data fetching.
* **Styling:** Tailwind CSS using utility classes. Avoid custom CSS files unless absolutely necessary for complex animations.
* **Components:** Functional components only. No class components.
* **Data Fetching:** Next.js App Router conventions where applicable; otherwise, strict separation of UI components and data-fetching hooks.

## Backend/API & Validation Standards
* **Architecture:** RESTful API design.
* **Validation:** All incoming requests (body, query, params) MUST be validated using Zod before processing.
* **Database Access:** All database interactions must go through the Prisma ORM. Raw SQL is forbidden unless explicitly approved for performance bottlenecks.
* **Response Format:** Standardized JSON responses for both success and error states.

```typescript
// Standard API Response Wrapper
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
}
```

## Error Handling
Errors must be categorized, caught at the boundary layer, and never expose stack traces to the client.

| Category | Description | HTTP Status |
| :--- | :--- | :--- |
| `VALIDATION_ERROR` | Invalid input data from the client. | 400 |
| `AUTHENTICATION_ERROR` | Missing or invalid credentials. | 401 |
| `AUTHORIZATION_ERROR` | Authenticated, but lacks required permissions. | 403 |
| `BUSINESS_ERROR` | Violates business logic (e.g., insufficient funds). | 422 |
| `EXTERNAL_SERVICE_ERROR` | Third-party API failure. | 502 |
| `INFRASTRUCTURE_ERROR` | Database or cache connection failure. | 503 |
| `UNKNOWN_ERROR` | Unhandled exceptions. | 500 |

## Logging
All logs must be structured JSON. `console.log` is forbidden in production code; use the designated logging library (e.g., Pino).

**Required Structured Log Fields:**
* `event`: String identifier for the action (e.g., `USER_LOGIN_FAILED`).
* `timestamp`: ISO 8601 UTC timestamp.
* `requestId`: UUID for distributed tracing.
* `userId`: (Optional) ID of the user performing the action.
* `metadata`: (Optional) Additional contextual JSON data.

## Security
* **Authentication & Authorization:** Must occur server-side. The frontend is an untrusted client.
* **Secrets Handling:** 
  * Loaded exclusively from environment variables at runtime.
  * Managed via AWS Secrets Manager in production.
  * NEVER commit secrets, `.env` files, or credentials to version control.
* **Data Protection:** PII must be encrypted at rest and in transit (TLS 1.2+).

## Accessibility
* **Standard:** All frontend interfaces must comply with WCAG 2.1 AA standards.
* **Implementation:** Semantic HTML is mandatory. Use ARIA attributes only when native HTML elements are insufficient.
* **Validation:** Automated accessibility checks (e.g., `eslint-plugin-jsx-a11y`, axe-core) must pass in CI.

## Performance
* **Frontend:** Core Web Vitals must remain in the "Good" threshold (LCP < 2.5s, FID < 100ms, CLS < 0.1).
* **Backend:** 95th percentile API response time must be under 200ms.
* **Database:** Queries taking longer than 50ms must be logged as warnings and optimized.

## Testing
* **Minimum Coverage:** 80% minimum overall; 95% for critical business logic (auth, billing, permissions).
* **Required Test Types:**
  * **Unit Tests:** (Jest) For pure functions, utilities, and isolated components.
  * **Integration Tests:** (Supertest) For API endpoints and database interactions.
  * **E2E Tests:** (Playwright) For critical user journeys (login, task creation, checkout).

## CI/CD
All code must pass through automated gates before merging. Direct pushes to `main` are blocked.

**CI Gates per PR:**
1. Linting (`eslint`, `prettier`)
2. Typechecking (`tsc --noEmit`)
3. Unit Tests
4. Integration Tests
5. Security Scan (Dependabot / Snyk)

## Documentation
* **READMEs:** Every package and app must have a `README.md` explaining setup, scripts, and purpose.
* **API Docs:** All backend endpoints must be documented using OpenAPI/Swagger.
* **Architecture:** Significant architectural changes must be documented using Architecture Decision Records (ADRs) in `docs/adrs/`.

## Observability
* **APM & Metrics:** Datadog is used for application performance monitoring and infrastructure metrics.
* **Tracing:** Distributed tracing must be implemented across the frontend and backend using `requestId`.
* **Alerting:** Critical errors (`INFRASTRUCTURE_ERROR`, `UNKNOWN_ERROR`) must trigger PagerDuty alerts.

## AI Development Rules
* **AI-generated code policy:** AI code is UNTRUSTED. It must be reviewed, tested, and validated by a human engineer before merging.
* **Agent restrictions (without human approval):**
  * May NOT deploy to production.
  * May NOT rotate credentials.
  * May NOT modify infrastructure state.
  * May NOT approve pull requests.

## Prompt/MCP/RAG Standards
* **Prompts:** Must be version-controlled, documented, and tested like code. Changes require peer review.
* **MCP Integrations:** Must operate on the principle of least-privilege, be fully auditable, and be instantly revocable.
* **RAG Sources:** Must be trusted, versioned, and include source-attribution in outputs.

## Code Review Standards
Every Pull Request must explicitly answer the following questions in its description:
1. **What changed?** (Brief summary of the technical changes)
2. **Why?** (Link to Jira ticket or business justification)
3. **Risks?** (Potential side effects, performance impacts)
4. **Rollback plan?** (How to revert if things go wrong)
5. **Testing evidence?** (Screenshots, test output, or explanation of how it was verified)

## Git Standards
* **Branch Conventions:** `feature/*`, `bugfix/*`, `hotfix/*`, `chore/*`
* **Commit Conventions:** Conventional Commits are mandatory.
  * `feat:` New feature
  * `fix:` Bug fix
  * `refactor:` Code change that neither fixes a bug nor adds a feature
  * `test:` Adding or correcting tests
  * `docs:` Documentation only changes
  * `perf:` Code change that improves performance
  * `chore:` Build process or auxiliary tool changes

## Dependency Rules
* **Policy:** 
  * Must pass automated security scans.
  * Must pass license review (MIT, Apache 2.0, BSD preferred; GPL is forbidden).
  * Must be actively maintained.
  * Prefer building over adding a dependency when the implementation is trivial (under 50 lines).

## Definition of Done
A task is only "Done" when:
- [ ] Requirements are fully implemented.
- [ ] Tests are written and passing.
- [ ] Typecheck and Linting are passing.
- [ ] Security review is completed.
- [ ] Documentation (API, ADRs, READMEs) is updated.
- [ ] Accessibility is validated (Frontend).
- [ ] Performance is validated.
- [ ] Code is reviewed and approved by at least one peer.

## Non-Negotiable Rules (NEVER / ALWAYS)
* **NEVER** commit secrets, API keys, or `.env` files.
* **NEVER** bypass CI checks or force-push to `main`.
* **NEVER** swallow errors silently (e.g., `catch (e) {}`).
* **ALWAYS** validate external input at the system boundary.
* **ALWAYS** write tests for bug fixes to prevent regressions.

## Amendment Process
This constitution is a living document but requires rigorous consensus to change:
1. **Written Proposal:** Submit a PR modifying this document with a detailed rationale.
2. **Architecture Review:** The proposal must be reviewed by the lead architect/engineering manager.
3. **Team Approval:** Requires a majority approval from the core engineering team.
4. **Version Increment:** Upon merge, the version number at the top of this document must be incremented (SemVer).