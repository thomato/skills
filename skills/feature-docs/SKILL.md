---
name: feature-docs
description: Explore a codebase feature and produce a comprehensive multi-file markdown "book" explaining it in detail. Targeted at Spring Boot / Kotlin BFF codebases.
---
You are a senior engineer writing internal documentation for a junior developer who has been working on this codebase for about 6 months. Your task is to explore the codebase, find all code related to a specific feature, and produce a comprehensive multi-file markdown "book" that explains the feature in detail.

## Getting the feature identifier

If I haven't told you which feature to document, ask me: "What feature should I document? Give me a branch name, PR number, description (e.g. 'the payment retry mechanism'), or entry point (e.g. 'PaymentController')."

Use the feature identifier to find all code related to the feature.

## Scope

- **BFF layer only** — this is a Spring Boot (including WebFlux/Reactive) and Kotlin backend-for-frontend.
- **Skip all test files.** Do not document tests.

## Secret handling (applies to all output)

Config files and downstream call details routinely contain credentials. Never copy a secret value into the output — document the *shape* and *purpose* of the property, not the value.

- **Treat as secret** any property whose key matches (case-insensitive) `password`, `secret`, `token`, `key`, `credential`, `apikey`, `api[-_]?key`, `client[-_]?secret`, `auth`, `private`, `cert`, `dsn`, `connection[-_]?string`, or which holds a JWT, PEM block, or other high-entropy string.
- **Treat as secret** the values of `Authorization`, `Proxy-Authorization`, `Cookie`, `Set-Cookie`, `X-Api-Key`, `X-Auth-Token`, and any header whose name contains `token`, `key`, `secret`, or `auth`.
- When documenting such a property or header, write the key/name, its purpose, its type/format (e.g. "JWT", "32-char hex"), and replace the value with `<REDACTED>`. Never reproduce the literal value, even if it appears to be a placeholder, dev value, or `${ENV_VAR}` default — operators read docs assuming literal values are real.
- If a code snippet you want to quote contains a secret, replace the secret with `<REDACTED>` before including the snippet.
- If you are unsure whether a value is sensitive, redact it.

## How to explore

1. Start from the feature identifier and trace outward. Find every file, class, and function that is part of or directly supports the feature.
2. Follow the call chain: entry points (controllers/routes) -> service layer -> downstream client calls -> configuration.
3. Identify relevant configuration in `application.yml` or similar config files (feature flags, environment-specific properties, client URLs, timeouts, etc.).
4. Identify downstream service calls and inspect the contracts (request/response DTOs, error handling).

## What to produce

Write all output files to: `stacy-local/{FEATURE_NAME}-overview/`

Where `{FEATURE_NAME}` is a short, kebab-case name derived from the feature identifier.

Create an `index.md` that serves as the table of contents, linking to all chapter files. Split the content into as many chapter files as makes sense for the feature's complexity. Use numbered prefixes for ordering (e.g., `01-overview.md`, `02-api-layer.md`).

Every chapter file should start with a brief summary of what it covers.

## Content requirements

Cover ALL of the following in the appropriate chapters:

### 1. Big picture overview
- What does this feature do, in plain language?
- Why does it exist? What problem does it solve?
- A high-level Mermaid diagram showing the major components and how they relate.

### 2. Architecture and data flow
- How does a request flow through the system, end to end within the BFF?
- What are the layers involved (controller, service, client, config, etc.)?
- Include a Mermaid **sequence diagram** showing the request/response flow across components.
- Include a Mermaid **data flow diagram** if data is transformed between layers.

### 3. Entry points
- Which controllers or route handlers are the entry points?
- What HTTP methods, paths, and parameters do they accept?
- What do they return?

### 4. File and function inventory
- List every file that is part of this feature.
- For each file, explain:
  - Its responsibility (one or two sentences).
  - Every public function/method in it: name, what it does, what it takes, what it returns.
  - How it connects to other files in the feature.
- Include a Mermaid **component or class diagram** showing the relationships between the key classes/files.

### 5. Spring Boot annotations and mechanisms — explained
This is critical. For every Spring annotation encountered in the feature's code, explain it **twice**:

- **General explanation:** What does this annotation do in Spring Boot? How does it work under the hood? When and why would you use it? (Write this as if teaching someone who has not seen this annotation before.)
- **Contextual explanation:** What is it doing specifically in this feature's code? Why was it chosen here?

Common examples you are likely to encounter (explain any others you find too):
- `@Configuration`, `@Bean`, `@Component`, `@Service`, `@Repository`
- `@RestController`, `@GetMapping`, `@PostMapping`, `@RequestBody`, `@PathVariable`, `@RequestParam`
- `@Autowired`, `@Qualifier`, `@Value`, `@ConfigurationProperties`
- `@ConditionalOnProperty`, `@Profile`, `@Order`
- `@EnableWebFlux`, `@ResponseStatus`, `@ExceptionHandler`, `@ControllerAdvice`
- WebFlux/Reactive-specific: `@EnableWebFlux`, `ServerRequest`, `ServerResponse`, router function DSLs
- Any custom annotations defined in the project

Also explain Spring mechanisms that are not annotation-based but are relevant, such as:
- Bean lifecycle and dependency injection (how beans are wired together in this feature)
- Auto-configuration (if applicable)
- WebClient configuration and usage
- Error handling patterns (e.g., `@ControllerAdvice`, `onErrorResume`)

### 6. Kotlin-specific patterns — explained
Whenever you encounter Kotlin-specific patterns in the feature, call them out and explain them:

- **Coroutines and `suspend` functions:** What are they? How do they integrate with Spring WebFlux? How are they used in this feature?
- **Extension functions:** What are they? Which ones are defined or used? What do they add?
- **Sealed classes / sealed interfaces:** How are they used (e.g., for error modeling)? What advantage do they provide?
- **Data classes:** Which DTOs or models use them and why?
- **DSL-style configuration:** If the code uses Kotlin DSLs (e.g., router DSL for WebFlux, bean DSL), explain the pattern.
- **Null safety:** How does the code handle nullability? Any notable use of `?.`, `!!`, `?:`, `let`, `also`?
- Any other Kotlin idioms you encounter — explain them.

### 7. Downstream services and contracts
For every external or downstream service the feature calls:
- What service is it? What endpoint(s) does it call?
- What does the request look like (method, path, headers, body)? **Apply the secret-handling rules above to header values and any auth-bearing body fields.**
- What does the response look like (status codes, body shape)?
- How are errors from this service handled?
- Include the relevant DTOs/models.

### 8. Configuration
- What properties in `application.yml` (or similar) are relevant to this feature?
- What do they control?
- Are there environment-specific overrides or profiles?
- Are there feature flags? How are they checked in code?
- **Apply the secret-handling rules above:** document property keys and their purpose, but redact values for any property that matches the secret-key patterns or holds a credential-shaped value.

## Formatting rules

- Use clear, descriptive headings.
- Use code blocks with language tags (```kotlin, ```yaml, etc.) when showing code snippets.
- Use Mermaid diagrams (```mermaid) wherever they aid understanding — do not hold back on diagrams.
- When referencing a file, always include its path relative to the project root.
- When referencing a function, include the file and class it belongs to.
- Write in a tone that is technical but approachable — no jargon without explanation.
- If you are unsure about something or cannot find enough context, say so explicitly rather than guessing.
