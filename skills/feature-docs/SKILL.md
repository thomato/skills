---
name: feature-docs
description: Document a codebase feature as a multi-chapter markdown "book" for onboarding. Use when the user wants to explain, onboard someone to, or audit a feature in a Spring Boot / Kotlin BFF.
---

Write a **book** for a junior engineer six months into the codebase: technical, approachable, no jargon left unexplained. One feature, one book — an `index.md` table of contents plus numbered chapter files (`01-overview.md`, `02-api-layer.md`, …) under `docs/features/{feature-name}/`, where `{feature-name}` is kebab-case from the feature identifier. Each chapter opens with a one-paragraph summary.

## Getting the feature

If the feature isn't named, ask: *"What feature should I document? Give me a branch name, PR number, description (e.g. 'the payment retry mechanism'), or entry point (e.g. 'PaymentController')."*

## Scope

- Focus on code.
- Skip tests.

## Secrets — redact, never reproduce

Config files and downstream calls routinely carry credentials. Document the **shape and purpose**, never the value. Operators read docs assuming literal values are real, so even a placeholder, dev default, or `${ENV_VAR}` becomes a leak.

- **Treat as secret** any property whose key matches (case-insensitive) `password`, `secret`, `token`, `key`, `credential`, `apikey`, `api[-_]?key`, `client[-_]?secret`, `auth`, `private`, `cert`, `dsn`, `connection[-_]?string`, or which holds a JWT, PEM block, or other high-entropy string.
- **Treat as secret** the values of `Authorization`, `Proxy-Authorization`, `Cookie`, `Set-Cookie`, `X-Api-Key`, `X-Auth-Token`, and any header whose name contains `token`, `key`, `secret`, or `auth`.
- Replace the value with `<REDACTED>` in tables and code snippets. Keep the key/name, purpose, and shape (e.g. "JWT", "32-char hex").
- When in doubt, redact.

These rules apply everywhere in the output — header tables, config chapters, inline snippets, all of it.

## Explore

Trace outward from the feature identifier along the call chain: entry points (controllers/routes) → service layer → downstream clients → configuration. Done when nothing reachable from what you've found is missing from your inventory — every file, every public function, every config property, every downstream contract accounted for.

## Chapters

Split the book into as many chapters as the feature's complexity warrants. Cover **every** item below — leave nothing implicit, and don't hold back on Mermaid diagrams where they aid understanding.

### Big picture
- What does this feature do, in plain language? Why does it exist?
- Mermaid diagram of the major components and how they relate.

### Architecture and data flow
- Mermaid **sequence diagram** of the end-to-end request/response flow .
- Mermaid **data flow diagram** when data is transformed between layers.

### Entry points
- Every controller/route handler in the feature: HTTP method, path, parameters, return shape.

### File and function inventory
- Every file in the feature, with: one-sentence responsibility; every public function (name, what it does, takes, returns); how it connects to its neighbours.
- Mermaid **component or class diagram** of the relationships between key classes/files.

### Downstream services and contracts

For every external/downstream service the feature calls:
- Which service, which endpoint(s).
- Request shape: method, path, headers, body. Redact per the secrets rules.
- Response shape: status codes, body.
- Error handling.
- The relevant DTOs/models.

### Configuration

- Properties in `application.yml` (or similar) relevant to this feature: key, purpose, default, profile/environment overrides.
- Feature flags and how they're checked in code.
- Redact per the secrets rules.

## Formatting

- Code blocks tagged by language (` ```kotlin `, ` ```yaml `, ` ```mermaid `).
- File references relative to the project root; function references include the file and class.
- If you can't find or can't confirm something, say so — don't guess.
