# CampMart V2 — GitHub Copilot Instructions

## 1. Project context

CampMart V2 is a PHP/MySQL(MariaDB) campus marketplace application. The repository is a mature hybrid codebase, not a clean framework-only project. Preserve the existing architecture unless the user explicitly asks for architectural migration.

Observed architecture on the current `main` branch:

- Server-rendered PHP pages at the repository root and in feature directories.
- Shared bootstrap/configuration through `includes/constant.php`, `includes/function.php`, and `includes/controller.php`.
- Shared database connection exposed as the global `$db` mysqli connection.
- Reusable database helpers in `includes/function.php`, including `dbInsert()`, `dbSelect()`, `dbSelectCol()`, `dbSelectOr()`, `dbUpdate()`, and `dbDelete()`.
- A newer REST-style API under `api/v1/` using controllers, middleware, helpers, JWT authentication, and a central router in `api/v1/index.php`.
- Existing legacy/procedural endpoints under `api/` that must not be assumed obsolete simply because `api/v1/` exists.
- AI functionality under `includes/ai/`, including configuration, LLM calls, embeddings, vectors, semantic search, recommendations, chatbot/replies, lost-and-found matching, caching, and search logging.
- Database schema and incremental migrations under `database/`, including `database/campmartv2.sql`, university/listing migrations, AI migrations, payment/rider migrations, and feature-specific table migrations.
- Composer dependency management exists; `composer.json` currently includes Cloudinary.
- Frontend code is primarily server-rendered HTML/CSS/JavaScript with Tailwind utility classes and Material Symbols usage in existing pages.
- Apache/XAMPP compatibility is part of the existing environment.

The repository currently contains both legacy and newer patterns. Do not "clean up" or replace them merely for stylistic reasons.

## 2. Mandatory inspection-first workflow

Before changing code:

1. Inspect the actual repository files relevant to the requested change.
2. Trace how the feature currently works end-to-end:
   - entry page or endpoint
   - includes/imports
   - helper/controller functions
   - database queries/tables
   - frontend JavaScript/AJAX
   - API routes/controllers/middleware where applicable
3. Search the entire codebase for existing implementations of the requested behavior.
4. Search for the relevant database table, column, function, class, endpoint, route, CSS class, icon, or JavaScript handler before creating anything new.
5. Check related migrations/schema files before changing database-dependent code.
6. Check callers/usages before changing a function signature, return value, column name, route, or shared helper.
7. If the current implementation is unclear, inspect more files rather than guessing.

Do not start by writing new code from the request alone.

## 3. Reuse before creating

- Reuse existing functions, helpers, controllers, classes, components, endpoints, and utilities whenever they already provide the needed behavior.
- Do not create a duplicate helper just because a similar function is inconvenient to use.
- Before adding a function, search for equivalent or near-equivalent functions across the repository.
- Before adding a database query helper, inspect `includes/function.php` first.
- Before adding authentication logic to `api/v1`, inspect `AuthMiddleware.php` and the existing JWT/configuration flow.
- Before adding product/service/listing logic, inspect the existing root-page controllers, `includes/controller.php`, and the corresponding `api/v1/controllers/*Controller.php`.
- Prefer extending the existing implementation when that is safer than creating a parallel implementation.

## 4. Preserve the CampMart architecture

Do not introduce a new framework, ORM, router, dependency, frontend build system, or architectural layer unless the user explicitly requests it.

Respect these existing boundaries:

- Root PHP pages: preserve their established include and rendering patterns.
- `includes/constant.php`: central database/site bootstrap.
- `includes/function.php`: shared procedural/database utility layer.
- `includes/controller.php`: existing application controller/profile/listing/image-processing behavior.
- `api/v1/`: REST API controllers, helpers, middleware, JWT, and routing.
- `api/`: existing legacy/procedural endpoints; do not remove or rewrite them as part of an unrelated task.
- `includes/ai/`: existing AI integration and feature-specific AI utilities.
- `database/`: schema plus incremental migrations.
- `vendor/`: third-party Composer dependencies; do not edit vendor source directly.

When both a legacy path and a v1 API path implement related functionality, determine which one the requested feature actually uses before modifying either.

## 5. Database safety is mandatory

The database is tightly coupled to the application. Before changing database-related code:

- Inspect the relevant table definition in `database/campmartv2.sql` and all relevant migration files.
- Search every repository reference to the table/column being changed.
- Confirm whether the column exists in the schema/migrations before using it.
- Check joins, WHERE clauses, INSERT/UPDATE arrays, API responses, admin pages, and frontend consumers.
- Never invent a column name because it seems logical.
- Never rename/remove a column without tracing all references first.
- Never assume the README's database instructions are current when the actual schema/configuration differs.
- Treat the actual schema and current code as the source of truth; document discrepancies rather than silently "fixing" unrelated documentation.
- If a schema change is required, create/update the appropriate migration rather than relying only on a code change.
- Avoid destructive database operations unless explicitly requested.

The current repository contains university-scoped listing logic using `university_id` in products/services/lost-and-found/sponsored content, so database-dependent changes must verify those relationships against the current schema/migrations.

## 6. Database/query conventions

- Prefer the existing prepared-statement helpers where they fit the operation.
- Existing shared helpers include `dbInsert`, `dbSelect`, `dbSelectCol`, `dbSelectOr`, `dbUpdate`, and `dbDelete`; inspect their exact behavior before bypassing them.
- In API controllers, follow the established mysqli prepared-statement style already used in `api/v1/controllers/`.
- Do not mix query styles unnecessarily inside one feature.
- Validate and sanitize input consistently with the surrounding implementation.
- Preserve existing response/error conventions.
- Be especially careful with dynamically constructed table/column names in helper functions.
- Do not change shared database helpers for a single feature unless the change is genuinely reusable and all callers have been checked.

## 7. API conventions

For `api/v1/`:

- Follow the existing central routing pattern in `api/v1/index.php`.
- Reuse `AuthMiddleware::authenticate()`, `AuthMiddleware::optional()`, and `requireRole()` where appropriate.
- Reuse existing response helpers, validation helpers, and pagination helpers.
- Preserve the existing controller naming convention: `ProductController`, `UserController`, `AuthController`, etc.
- Keep authentication/authorization checks at the established middleware/controller boundaries.
- Do not create a second authentication mechanism for the same API.
- Preserve existing HTTP status and JSON response conventions.

For legacy `api/` endpoints:

- Inspect the endpoint and its callers before modifying it.
- Do not migrate it to v1 unless migration is explicitly part of the request.

## 8. PHP conventions

- Preserve existing PHP 7.4+/PHP 8.x-compatible syntax unless the actual target environment is verified otherwise.
- Match the naming and formatting style of the surrounding file.
- Use strict, explicit checks when modifying security-sensitive or database-sensitive code.
- Preserve existing session/authentication behavior unless the task specifically changes it.
- Do not remove existing validation, CSRF protection, authorization, or account-status checks merely to simplify code.
- Do not expose secrets, API keys, database credentials, or tokens in source files.
- Keep environment-specific secrets in the existing environment/configuration mechanism.

## 9. Frontend conventions

- Inspect existing HTML, Tailwind classes, JavaScript, and shared includes before introducing new UI patterns.
- Reuse existing CSS utility classes and UI patterns where possible.
- Preserve existing responsive behavior.
- Do not replace existing icon infrastructure globally for an unrelated feature.
- When working with icons, inspect how the surrounding page loads and renders Material Symbols before changing icon markup or adding another icon library.
- Do not add a large frontend dependency for a small UI requirement.
- For AJAX/fetch changes, inspect the existing endpoint, request format, response format, authentication/session behavior, and error handling first.

## 10. AI integration conventions

AI functionality already exists under `includes/ai/` and is connected to product/service/lost-found functionality.

Before adding or changing AI behavior:

- Inspect `includes/ai/config.php`, `dotenv.php`, `llm.php`, `embeddings.php`, `vectors.php`, `search.php`, `recommend.php`, `lostfound.php`, `chatbot.php`, `replies.php`, caching, and search logging as relevant.
- Reuse existing AI wrappers and configuration.
- Do not introduce a second provider abstraction unless explicitly requested.
- Do not hard-code API keys.
- Preserve fallback behavior when AI is disabled/unavailable.
- Check the corresponding AI database migration before changing embedding/search behavior.

## 11. Naming and file conventions

Follow existing names rather than inventing a new naming system.

Observed conventions include:

- Root PHP pages use descriptive lowercase/hyphenated names such as `product.php`, `my-products.php`, etc.
- API v1 controllers use PascalCase class names ending in `Controller`.
- Shared procedural helpers live in `includes/function.php`.
- AI utilities use descriptive lowercase filenames under `includes/ai/`.
- SQL migrations use descriptive lowercase names with underscores.

When adding a file, choose a name consistent with the directory and neighboring files.

## 12. Scope control

For every requested change:

- Make the smallest complete change that solves the problem.
- Do not refactor unrelated code.
- Do not rename unrelated files/functions/variables.
- Do not reformat entire files unless required.
- Do not update documentation, migrations, dependencies, or unrelated UI merely because you notice an imperfection.
- Do not "modernize" legacy code unless it is necessary for the requested change.
- Do not delete existing functionality unless explicitly requested.
- If you discover unrelated bugs, mention them separately instead of silently changing them.

## 13. Avoid breaking existing features

Before modifying shared code, identify its callers.

After changes, verify:

- PHP syntax for changed PHP files.
- Includes/requires resolve to real files.
- Functions/classes referenced by changed code still exist.
- Database tables and columns referenced by changed queries exist in the current schema/migrations.
- API routes still map to real controllers/methods.
- Frontend AJAX/fetch URLs still point to real endpoints.
- Response fields expected by frontend code are still returned.
- Authentication/authorization behavior is preserved.
- Existing feature paths are not accidentally redirected or blocked.
- File paths work under the project's Apache/XAMPP deployment structure.

When possible, run the narrowest relevant test or static check first, then broader checks if practical.

## 14. Error and reference verification

After making a change, search the repository again for:

- old function names
- renamed variables
- old database column names
- old table names
- old endpoint URLs
- old file paths
- removed classes/functions
- stale references to the changed feature

Do not report a change as complete until broken references introduced by the change have been checked.

If a runtime test is unavailable, explicitly state what was statically verified and what could not be executed.

## 15. Git/change discipline

- Keep changes reviewable.
- Do not modify generated/vendor files unless specifically required.
- Do not commit secrets.
- Do not alter unrelated files.
- Prefer one focused change over a broad cleanup.
- Before finishing, inspect the final diff conceptually and confirm every changed file is necessary for the user's request.

## 16. Important current-repository discrepancies

The current README contains older setup/architecture information that does not fully describe the present repository. For example, it references older database/setup paths and older project phases, while the actual repository contains a much larger application, `api/v1/`, AI modules, migrations, and feature directories.

Therefore:

**Do not treat README claims as authoritative when they conflict with actual code, schema, or current file structure. Inspect the repository.**

Likewise, the current database configuration in `includes/constant.php` and SQL dump/migration files must be inspected before changing the database name or schema. Never infer the intended database name from the project/repository name alone.

## 17. Default Copilot behavior

When the user asks for a code change, follow this sequence:

**Inspect → Search → Trace → Plan minimal change → Modify → Re-scan references → Validate → Summarize.**

If you cannot verify an assumption from the repository, say so and inspect further instead of guessing.

When proposing code, prefer an existing CampMart pattern over a new abstraction.

When a requested change could affect shared functionality, explicitly check its callers and related database/API/frontend references before editing.

The priority order is:

1. Preserve existing working behavior.
2. Follow the actual CampMart architecture.
3. Reuse existing code.
4. Make the smallest necessary change.
5. Verify references and errors.
6. Improve code quality only when it directly supports the requested change.
