# CineTrack Project Guide

## 1. Architecture overview
CineTrack follows a layered request flow: `frontend/services/` → `backend/src/routes/` → `backend/src/controllers/` → `backend/src/services/` → `backend/src/data/` → SQLite.

- **`frontend/services/`:** This is the only frontend layer allowed to call the backend API. Keeping network code here separates UI rendering from data access details and supports single-responsibility design.
- **`backend/src/routes/`:** Routes define HTTP endpoints and connect requests to controllers. Limiting them to endpoint wiring keeps transport concerns separate from business logic.
- **`backend/src/controllers/`:** Controllers handle request parsing and response shaping. This layer keeps Express-specific concerns out of the core application rules.
- **`backend/src/services/`:** Services contain business rules such as moving a movie between statuses or enforcing rating rules. Centralizing behavior here makes the system easier to test and aligns with SOLID by isolating decision-making from delivery mechanisms.
- **`backend/src/data/`:** The data layer is the only code allowed to touch SQLite directly. This protects the rest of the app from persistence details and makes future storage changes less disruptive.
- **SQLite database:** SQLite stores the app's local movie data in a file for privacy and simplicity. Keeping storage local fits the team's requirement that user data stays on the user's machine.

## 2. Folder-by-folder guide

| Folder | Purpose | What belongs here | What does NOT belong here | Owner |
| --- | --- | --- | --- | --- |
| `.github/workflows/` | CI automation definitions | GitHub Actions workflow files | App source code or setup docs | Charles Balderas |
| `.github/ISSUE_TEMPLATE/` | Standard issue templates | Bug and feature issue templates | Architecture docs or app code | Shared |
| `frontend/` | React + Vite + TypeScript client app | Frontend source, static assets, frontend tests | Backend logic, database access, direct SQLite code | Yunjong Heo |
| `frontend/public/` | Static assets served as-is | Images, icons, manifest files | Source code or build output | Yunjong Heo |
| `frontend/src/components/` | Reusable UI building blocks | Shared display components such as cards and stars | API calls, page-level routing logic, backend code | Yunjong Heo |
| `frontend/src/pages/` | Screen-level views | Page containers for Want To Watch, Watched, Insights, and Add/Edit Movie | Reusable shared widgets or backend logic | Yunjong Heo |
| `frontend/src/services/` | Frontend API access layer | Fetch wrappers and API request helpers | JSX components or direct database access | Yunjong Heo |
| `frontend/src/hooks/` | Reusable frontend behavior | Custom React hooks and state helpers | API route definitions or database logic | Yunjong Heo |
| `frontend/src/types/` | Shared frontend type definitions | Movie types that mirror the API contract | Business logic or UI styling | Yunjong Heo |
| `frontend/src/styles/` | Styling resources | Global styles, tokens, and CSS modules if used | Business logic or API code | Yunjong Heo |
| `frontend/tests/` | Frontend test files | Component or UI-focused tests | Backend tests or production source files | Yunjong Heo |
| `backend/` | Node.js + Express + TypeScript API | Backend source and backend tests | Frontend UI components or deployment docs | Mia Graham |
| `backend/src/routes/` | HTTP endpoint definitions only | Express route mappings | Business rules, database queries, analytics calculations | Mia Graham |
| `backend/src/controllers/` | Request/response coordination | Input extraction, status codes, response shaping | Direct SQL, heavy business logic, UI code | Mia Graham |
| `backend/src/services/` | Core backend business rules | Status changes, validation orchestration, rating rules | Express route wiring or raw SQLite calls | Mia Graham |
| `backend/src/validation/` | Request validation definitions | Schemas and request validation helpers | Database queries or analytics logic | Mia Graham |
| `backend/src/middleware/` | Cross-cutting Express behavior | Error handling, logging, auth-related middleware if needed later | Business rules or database access | Mia Graham |
| `backend/src/data/` | Persistence/repository layer | SQLite queries, repository interfaces, data mappers | Route handlers or frontend code | Charles Balderas |
| `backend/src/analytics/` | Analytics calculations | Total hours, average rating, top genres logic | Route wiring or direct UI rendering | Charles Balderas |
| `backend/tests/unit/` | Backend unit test suite | Service, validation, and analytics unit tests | Integration-only API tests or app code | Mia Graham |
| `backend/tests/integration/` | Backend integration test suite | Endpoint and data-flow integration tests | Frontend tests or production code | Mia Graham |
| `database/` | SQLite database assets | Schema, migrations, seed data assets | API route code or frontend assets | Charles Balderas |
| `database/migrations/` | Schema evolution scripts | Migration files for database changes | Business logic or UI code | Charles Balderas |
| `database/seed/` | Local seed data support | Seed SQL or seed data files | Production business logic or frontend code | Charles Balderas |
| `docs/` | Project-level documentation | The shared project guide and future top-level docs if approved | Subsystem README sprawl or app source code | Shared |

## 3. API contract (draft)
**DRAFT: agree as a team before building.**

### Planned endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/movies` | List movies with optional search and filtering by title, genre, and status |
| GET | `/api/movies/:id` | Get one movie by id |
| POST | `/api/movies` | Create a new movie entry |
| PUT | `/api/movies/:id` | Update an existing movie entry |
| DELETE | `/api/movies/:id` | Delete a movie entry |
| PATCH | `/api/movies/:id/status` | Change a movie's status, such as moving it from want-to-watch to watched |
| PATCH | `/api/movies/:id/rating` | Set or update a movie's rating |
| GET | `/api/insights` | Get analytics such as total hours watched, average rating, and top genres |

### Movie data shape in plain words
A Movie should include an id, title, genre, runtime in minutes, watch status, rating, created date, updated date, and optional watched date. The team should also agree on whether status values are a fixed set such as `want-to-watch` and `watched`, and whether rating is nullable until a movie is watched.

## 4. Integration points
The team's main integration points are the API contract, the shared Movie type, and the data layer interface. Yunjong's frontend depends on stable request and response shapes, Mia's backend depends on service and controller expectations, and Charles's data layer must satisfy the backend contract without leaking SQLite details upward.

Any contract change affecting endpoint shape, the shared Movie type, or the data layer interface requires a pull request reviewed by all three teammates before merging. This rule protects integration work by making interface changes explicit and coordinated.

## 5. Git workflow
- `main` is a protected branch.
- Create feature branches using the format `feature/<name>-<short-desc>`.
- Keep pull requests small and focused.
- Require at least one teammate review before merging.
- CI must pass before merge.
- Use commit prefixes such as `feat:`, `fix:`, `test:`, and `docs:`.

## 6. Testing expectations
Yunjong should cover key UI flows with component tests, especially important forms and page interactions. Mia should write unit tests for backend services and validation logic plus integration tests for API routes using Jest and Supertest.

Charles should add tests for analytics calculations and any data-layer behavior that benefits from verification, and analytics calculations must have tests before related work is considered complete. Across the team, testing should focus on behavior and contract correctness rather than implementation details.

## 7. Environment and setup
1. Copy `.env.example` to a local `.env` file for each app as needed.
2. Install frontend dependencies in `frontend/`.
3. Install backend dependencies in `backend/`.
4. Start the frontend locally with the Vite development command.
5. Start the backend locally with the Express development command.
6. Point the frontend API base URL at the local backend.
7. Use the SQLite file path from `.env` so local data stays on the developer machine.

## 8. Deployment
The frontend will deploy to Vercel and the backend will deploy to Render. Required environment variables should include the frontend API base URL, backend port, allowed frontend origin, and SQLite database path or any deployment-specific storage path used by the backend service.

Charles Balderas owns deployment setup, integration checks, and triggering production-oriented deploy decisions. Team members should coordinate with Charles before changing deployment configuration or environment variable requirements.

## 9. Roadmap
- **Week 1:** Finalize schema, confirm architecture, and build the basic UI shell.
- **Week 2:** Implement core tracking features for creating, editing, listing, filtering, and changing movie status.
- **Week 3:** Add analytics, complete core testing coverage, and tighten integration behavior.
- **Week 4:** Polish UX, finish documentation, verify deployment, and prepare the final handoff.
- **Optional features:** Only start optional enhancements after the core workflow is stable and working end to end.

## 10. Definition of done
- [ ] Feature matches the agreed issue or user story.
- [ ] Folder ownership and architecture rules are respected.
- [ ] API contract changes, if any, were reviewed by all three teammates.
- [ ] Relevant tests were added or updated.
- [ ] Analytics logic has tests when analytics behavior changed.
- [ ] Local manual verification was completed.
- [ ] CI passed.
- [ ] Documentation was updated if the change affected team workflow or contracts.
