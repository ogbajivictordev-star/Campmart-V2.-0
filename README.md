# CampMart V2

> **Your Campus. Your Marketplace.**

CampMart V2 is a campus-focused online marketplace built for students and campus communities. It provides a web application for discovering, buying, selling, saving, messaging about, and managing products and services, together with campus/community features and a REST API.

This README describes the **current repository structure and implementation**. It replaces the older README that described CampMart as an earlier static/database prototype.

---

## Overview

CampMart V2 is a PHP/MySQL(MariaDB) application designed to run primarily on Apache/XAMPP.

The current codebase is a mature, feature-rich PHP application containing:

- Product marketplace
- Student services marketplace
- User accounts and profiles
- University/campus scoping
- Categories
- Search and suggestions
- Bookmarks
- Cart and orders
- Messaging/conversations
- Notifications
- Reviews and ratings
- Lost & Found
- Free-item/community listings
- Sponsored/advertising features
- Flash sales
- Affiliate functionality
- QR/QR-store functionality
- Rider/delivery functionality
- Admin management
- Email verification
- Password reset functionality
- SEO-related settings
- AI-powered search/recommendations/chat/lost-and-found assistance
- A versioned REST API under `api/v1/`
- OpenAPI documentation for the v1 API

The repository also contains legacy/procedural API endpoints alongside the newer v1 API. These are both part of the current codebase and should not be assumed interchangeable.

---

## Technology Stack

| Area | Current implementation |
|---|---|
| Backend | PHP |
| Database | MySQL / MariaDB |
| Web server | Apache |
| Local development | XAMPP-compatible |
| Database driver | MySQLi |
| API | PHP REST-style API under `api/v1/` |
| Authentication | Session-based web authentication plus JWT for API v1 |
| Frontend | Server-rendered HTML, CSS, JavaScript |
| UI utilities | Tailwind utility classes and existing Material Symbols usage |
| Image/media | Existing upload/image-processing code and Cloudinary dependency |
| AI | Mistral-based configuration in `.env.example` |
| API documentation | OpenAPI 3.0.3 |
| Dependency manager | Composer |

The repository currently contains Composer's `vendor/` directory. Third-party source inside `vendor/` should not normally be edited manually.

---

## Current Architecture

CampMart is not a single-framework application. It uses several established layers.

### Main PHP application

Important shared files include:

```
includes/
├── constant.php
├── controller.php
└── function.php
```

### `includes/constant.php`

Provides core database/site configuration and creates the shared MySQLi connection.

The current repository configuration uses:

```
DB_SERVER = localhost
DB_USER   = root
DB_PASS   = ""
DB_NAME   = campmartv2
```

**Important:** the repository currently references the database as `campmartv2`. Do not rename the database or change this configuration without also checking the SQL schema, migrations, local database, and every code reference.

### `includes/function.php`

Contains the shared procedural utility/database layer.

Existing helpers include functions such as:

- `dbInsert()`
- `dbSelect()`
- `dbSelectCol()`
- `dbSelectOr()`
- `dbUpdate()`
- `dbDelete()`
- `countRows()`
- `countRowsOr()`
- `tableRowItem()`
- `tableRowItemOr()`
- `sanitize()`
- password/time and other application utilities

When adding functionality, inspect this file first and reuse an existing helper when appropriate.

### `includes/controller.php`

Contains substantial application-level behavior including profile/user handling, listing operations, image processing, lost-and-found posting, role restrictions, and other shared application logic.

Do not duplicate functionality from this file without first searching its existing functions.

---

## REST API v1

The newer REST API lives under:

```
api/v1/
```

Its main entry point is:

```
api/v1/index.php
```

The API currently uses:

- Configuration
- CORS handling
- Database bootstrap
- JWT authentication
- Authentication middleware
- Request validation
- Standard response helpers
- Pagination
- Controllers
- OpenAPI documentation

### Current controller areas

The repository currently contains controllers for:

- Authentication
- Products
- Categories
- Users
- Bookmarks
- Cart
- Orders
- Services
- Reviews
- Notifications
- Messages/chats
- Search

Representative structure:

```
api/v1/
├── config/
│   ├── cors.php
│   ├── database.php
│   └── jwt.php
├── controllers/
│   ├── AuthController.php
│   ├── BookmarkController.php
│   ├── CartController.php
│   ├── CategoryController.php
│   ├── MessageController.php
│   ├── NotificationController.php
│   ├── OrderController.php
│   ├── ProductController.php
│   ├── ReviewController.php
│   ├── SearchController.php
│   ├── ServiceController.php
│   └── UserController.php
├── helpers/
│   ├── pagination.php
│   ├── response.php
│   └── validator.php
├── middleware/
│   └── AuthMiddleware.php
├── docs/
│   ├── index.html
│   └── openapi.json
└── index.php
```

### API authentication

API v1 uses bearer JWT authentication.

Protected routes use the existing `AuthMiddleware` implementation. Public and optional-authentication routes use the existing patterns where applicable.

Do not create a second authentication mechanism for API v1 without an explicit architectural decision.

### API documentation

The OpenAPI specification is located at:

```
api/v1/docs/openapi.json
```

The documentation page is:

```
api/v1/docs/index.html
```

When changing an API endpoint, check whether the OpenAPI documentation also needs to be updated.

---

## Legacy API

The repository also contains procedural endpoints directly under:

```
api/
```

Examples include endpoints for:

- Login/signup
- Cart operations
- Orders
- Bookmarks
- Search suggestions/logging
- View/ad tracking
- QR functionality
- AI functionality
- Admin operations
- Dashboard operations

There is also an `api/admin/`, `api/ai/`, `api/dashboard/`, and `api/qrstore/` structure.

Do not remove or migrate legacy endpoints simply because `api/v1/` exists. First inspect their callers and determine which implementation the requested feature actually uses.

---

## AI Features

AI configuration is provided through:

```
.env.example
```

The current example configuration identifies Mistral as the AI provider and includes configuration for:

- LLM requests
- Embeddings
- Vector/search functionality
- Request timeout
- AI logging
- AI enable/disable state

AI-related functionality exists under:

```
includes/ai/
api/ai/
ai-reindex.php
```

The codebase includes AI functionality related to areas such as:

- Semantic search
- Recommendations
- Chatbot functionality
- Smart replies
- Lost-and-found matching
- Embedding/indexing
- Search logging/caching

### AI environment configuration

Copy the example configuration into your local environment/configuration as appropriate and provide your own secret values.

Never commit real API keys or other secrets.

---

## Database

The primary SQL dump currently tracked in the repository is:

```
database/campmartv2.sql
```

The dump was generated from MariaDB and is associated with the `campmartv2` database.

The database directory also contains incremental feature migrations, including migrations for areas such as:

- AI features
- Email verification
- SEO settings
- University/listing relationships
- University advertising relationships
- Cart
- Orders
- Password resets
- Featured subscriptions
- Payments
- Riders
- Campus submissions
- Guide videos

### Important database rule

The SQL dump and migrations represent the database side of the application. Before changing a database-dependent feature:

1. Inspect the relevant table in `database/campmartv2.sql`.
2. Check related migration files.
3. Search the entire repository for the table/column.
4. Check PHP queries and API controllers.
5. Check frontend consumers.
6. Make the smallest compatible change.

Do not assume that a column exists merely because its name seems logical.

---

## Local Development with XAMPP

The application is structured to work in an Apache/XAMPP environment.

A typical local setup is:

```
C:\xampp\htdocs\campmart
```

or another Apache document-root location.

### Basic setup

1. Install/start XAMPP.
2. Start **Apache**.
3. Start **MySQL**.
4. Clone/copy the repository into the Apache document root.
5. Create/import the `campmartv2` database.
6. Import `database/campmartv2.sql` or apply the required migrations.
7. Check `includes/constant.php` and make sure its database settings match the local database.
8. Install Composer dependencies if the environment does not already contain `vendor/`.
9. Open the application through Apache.

Example local URL:

```
http://localhost/campmart/
```

The exact URL depends on the folder name used in your XAMPP `htdocs` directory.

### Composer

The repository currently declares Cloudinary in `composer.json`.

If dependencies need to be installed:

```bash
composer install
```

Do not edit third-party packages directly inside `vendor/`.

---

## Main Feature Areas

The current repository contains implementation for a broad set of marketplace/community functionality.

### Marketplace

- Product listings
- Product categories
- Product images
- Product detail pages
- Seller/store pages
- Product views
- Bookmarks
- Search
- Cart
- Checkout/order flows
- Reviews
- Flash sales
- Sponsored content

### Services

- Service listings
- Service categories
- Provider profiles
- Service pricing
- Portfolio/media
- Reviews and ratings
- Service management

### Users

- Registration
- Login/logout
- Profiles
- Profile images
- University information
- Account status
- Email verification
- Password reset
- Role-based access

### Community

- Lost & Found
- Free/community listings
- Campus/university-specific content
- User messaging
- Notifications
- Search history/logging

### Business/monetization features

The repository contains implementation for areas including:

- Sponsored content
- Featured subscriptions
- Affiliate functionality
- Payments
- Flash sales
- Rider/delivery functionality

### Admin

The repository includes admin-facing pages and endpoints for areas such as:

- Users
- Products
- Services
- Transactions
- Ads
- Lost & Found
- Videos
- Categories
- Plans/subscriptions
- Analytics/management functionality

---

## Important Directories

A simplified view of the current project:

```
CampMart V2
│
├── .github/
│   └── copilot-instructions.md
│
├── api/
│   ├── admin/
│   ├── ai/
│   ├── dashboard/
│   ├── qrstore/
│   └── v1/
│
├── database/
│   ├── campmartv2.sql
│   └── feature migrations
│
├── includes/
│   ├── ai/
│   ├── constant.php
│   ├── controller.php
│   └── function.php
│
├── vendor/
│   └── Composer dependencies
│
├── admin*.php
├── rider/
├── *.php application pages
├── .env.example
├── .htaccess
├── composer.json
└── README.md
```

This is intentionally a simplified map. The repository contains many additional feature-specific files.

---

## Configuration and Secrets

The repository includes:

```
.env.example
```

This file documents AI-related environment settings.

Do not place real API keys, passwords, database credentials, JWT secrets, payment credentials, or other private secrets into Git.

Before deploying publicly:

- Use production secrets outside source control.
- Review database credentials.
- Review API authentication secrets.
- Review upload permissions.
- Enable HTTPS.
- Review error reporting.
- Restrict sensitive admin functionality.
- Verify CORS and API configuration.

---

## Development Rules

CampMart V2 is a mature codebase. New changes should follow the existing architecture.

### Before changing code

**Inspect → Search → Trace → Plan → Modify → Re-scan → Validate**

Always:

- Search for existing implementations.
- Reuse existing helpers.
- Inspect database references.
- Inspect API callers.
- Inspect frontend consumers.
- Check shared functions before changing them.
- Avoid creating duplicate functionality.

### Keep changes focused

Do not:

- Refactor unrelated code.
- Rename unrelated files.
- Replace the architecture without a specific reason.
- Add a framework just for one feature.
- Remove legacy functionality without tracing its usage.
- Modify vendor code unnecessarily.
- Change database tables without checking all references.

The project-specific Copilot rules are maintained in:

```
.github/copilot-instructions.md
```

---

## Security Notes

The application contains authentication, authorization, user data, database operations, uploads, API endpoints, and payment-related functionality.

When developing:

- Use prepared statements where the surrounding implementation supports them.
- Validate user input.
- Preserve authentication and authorization checks.
- Protect file uploads.
- Never commit secrets.
- Do not expose private configuration.
- Check permissions before changing admin/rider functionality.
- Preserve existing CSRF/security mechanisms.
- Review database and API changes for unintended access.

Security-related changes should be tested against both authenticated and unauthenticated behavior where applicable.

---

## API Endpoint Summary

The v1 API currently exposes resource areas including:

```
/auth
/products
/categories
/user
/bookmarks
/cart
/orders
/services
/service-categories
/reviews
/notifications
/chats
/search
/users
```

The exact method, authentication requirement, parameters, and response structure are defined by the implementation in:

```
api/v1/index.php
api/v1/controllers/
api/v1/docs/openapi.json
```

Use those files as the source of truth rather than copying an endpoint description from this README.

---

## Database and Architecture Notes

There are intentionally multiple generations of code in the repository.

For example:

- The web application uses shared procedural helpers.
- The v1 API uses controller/middleware/helper organization.
- Legacy procedural API endpoints remain in `api/`.
- AI functionality has its own module structure.
- Feature migrations have accumulated as the application has grown.

This is expected in the current repository.

**Do not treat older code as automatically disposable.** Determine what is currently used before replacing or removing it.

---

## Documentation Files

Other documentation in the repository includes:

- `CLEAN_URL_GUIDE.md`
- `EMAIL_VERIFICATION.md`
- `api/v1/docs/openapi.json`
- `api/v1/docs/index.html`
- `.github/copilot-instructions.md`

When functionality changes, update the relevant documentation instead of allowing it to become stale.

---

## Troubleshooting

### Database connection error

Check:

```
includes/constant.php
```

Verify:

- MySQL/MariaDB is running.
- Database name matches `DB_NAME`.
- Username/password are correct.
- The required tables exist.

The current repository configuration uses:

```
campmartv2
```

### Unknown database/unknown column errors

Do not immediately change the PHP query.

First:

1. Check the SQL schema.
2. Check migrations.
3. Search for all references to the table/column.
4. Determine whether the local database is behind the repository schema.
5. Apply the appropriate migration or make the smallest compatible code change.

### API errors

Check:

```
api/v1/index.php
api/v1/config/
api/v1/middleware/
api/v1/controllers/
```

Also check the request method, URL, authentication header, request body, and expected response format.

### AI errors

Check:

```
.env
.env.example
includes/ai/
api/ai/
```

Verify that AI is enabled and the required provider credentials/configuration are available.

---

## Current Repository Status

This repository is an actively developed CampMart V2 codebase rather than the earlier database-only prototype described by the old README.

The codebase already contains substantial marketplace, community, admin, API, AI, payment, rider, affiliate, and account functionality.

Therefore, feature requests should normally be implemented by **extending and connecting existing functionality**, not by rebuilding the feature from scratch.

---

## License / Project Use

Refer to the repository's current project ownership and distribution requirements before redistributing or deploying the application.

---

## Maintainer Guidance

When working on CampMart V2:

> **Inspect the actual code. Search before creating. Reuse before duplicating. Check the database before changing queries. Keep changes focused. Verify references after every change.**

For AI-assisted development, read:

```
.github/copilot-instructions.md
```

before making substantial changes.

---

**CampMart V2 — Your Campus. Your Marketplace.**
