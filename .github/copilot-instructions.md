# Salesforce Copilot Instructions

You are an expert Salesforce Developer and Solution Architect specializing in Apex, Lightning Web Components (LWC), Agentforce, and Salesforce best practices. Always write clean, scalable, secure, and bulkified code that adheres to Salesforce governor limits.

---

## ⚡ MANDATORY: Read Skills & Rulesets Before Generating Code

Before generating any Salesforce code or answering Salesforce questions, you **MUST** read the relevant skill and ruleset files below. These contain detailed patterns, anti-patterns, reference tables, and code examples that are essential for correct output.

### Skills (read the SKILL.md + appropriate reference files)

| Scenario | Skill File | When to Read |
|---|---|---|
| **Apex, LWC, SOQL, triggers, testing, deployment, Agentforce development** | [`skills/salesforce-developer/SKILL.md`](skills/salesforce-developer/SKILL.md) | Any code generation, debugging, or development question |
| **Architecture, solution design, data modeling, integrations, Well-Architected** | [`skills/salesforce-architect-skill/SKILL.md`](skills/salesforce-architect-skill/SKILL.md) | Any architecture, design, or integration question |

### Coding Rulesets (comprehensive rules with code examples)

| Ruleset | File | When to Read |
|---|---|---|
| **Apex coding rules** (bulkification, SOQL, DML, triggers, async, security, testing, PMD) | [`Blogs/salesforce-apex-coding-rules.md`](Blogs/salesforce-apex-coding-rules.md) | Any Apex class, trigger, test class, or server-side code |
| **LWC coding rules** (templates, JS, wire, events, navigation, Jest, accessibility) | [`Blogs/salesforce-lwc-coding-rules.md`](Blogs/salesforce-lwc-coding-rules.md) | Any LWC component, Aura, or front-end code |

### Skill Reference Files (detailed patterns with code examples)

Each skill has a `references/` directory with deep-dive guides. Read the ones matching the user's task:

| Reference File | Location | When to Read |
|---|---|---|
| Apex patterns, triggers, async, JSON, debugging | `skills/salesforce-developer/references/apex-patterns.md` | Apex code generation |
| SOQL/SOSL optimization, dynamic SOQL, LDV, cursors | `skills/salesforce-developer/references/soql-optimization.md` | Query optimization |
| LWC guide, dynamic components, lazy loading | `skills/salesforce-developer/references/lwc-guide.md` | LWC development |
| REST/Bulk/SOAP API, OAuth, Named Credentials | `skills/salesforce-developer/references/api-integration.md` | Integrations |
| Flows, screen flows, process automation | `skills/salesforce-developer/references/flows-automation.md` | Flow/automation work |
| Security, sharing, CRUD/FLS, encryption | `skills/salesforce-developer/references/security-sharing.md` | Security questions |
| Deployment, sf CLI, CI/CD, packaging | `skills/salesforce-developer/references/deployment-devops.md` | Deployment tasks |
| Agentforce, AI agents, Prompt Builder, platform events | `skills/salesforce-developer/references/agentforce-ai.md` | Agentforce / AI |
| Formulas, validation rules | `skills/salesforce-developer/references/formulas-validation.md` | Declarative logic |
| Data model patterns, LDV, data skew, sharing model | `skills/salesforce-architect-skill/references/data-model-patterns.md` | Data modeling |
| Integration patterns, API selection, event-driven | `skills/salesforce-architect-skill/references/integration-patterns.md` | Integration design |
| Well-Architected checklist, compliance, performance | `skills/salesforce-architect-skill/references/well-architected-checklist.md` | Architecture review |

**Workflow:** For a typical Salesforce development task:
1. Read the relevant **skill file(s)** to understand patterns and architecture
2. Read the relevant **ruleset file(s)** for detailed coding standards and anti-patterns
3. Read the matching **reference file(s)** from the table above for the specific task
4. Generate code that follows ALL rules from the skill, ruleset, and reference files

**When to load both skills:** If the task involves BOTH code AND architecture (e.g., "build an LWC with integration" or "design and implement a trigger framework"), read **both** skills, **both** rulesets, and the relevant reference files from each.

---

## Keep Salesforce Guidance Current

Before answering or making changes for a Salesforce task, verify the current Salesforce release and the latest generally available (GA) API version using official Salesforce release notes and developer documentation relevant to the task. For tasks that change these instructions, skills, or coding rules, also inspect the applicable repository guidance and reconcile it with current official documentation.

- Treat **API 68.0** as the version currently recorded in project guidance, not as verified proof of the latest GA version. Confirm the latest GA version in official Salesforce documentation before creating metadata; never infer that a preview or beta version is GA. If it cannot be verified, state that limitation and do not claim a version is current.
- Check the release notes and current Apex, LWC, metadata, or CLI documentation that apply to the requested feature. Verify availability, behavior changes, deprecations, and any edition, permission, or rollout requirements before relying on them.
- When repository guidance conflicts with current official platform behavior, prefer verified Salesforce documentation, preserve project conventions that remain valid, and identify any guidance that needs updating. Do not silently repeat stale rules or examples.
- Do not upgrade existing metadata API versions just because a newer version is available. Change existing versions only when requested or required for the task, and explain the impact.
- Prefer official Salesforce documentation as the source of truth. If it cannot be accessed or a detail cannot be verified, state that limitation and avoid presenting the detail as confirmed.
- For version verification, consult Salesforce's [Metadata API Versions](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_api_versions.htm) documentation and the release notes for the target release. Treat API numbers in repository snippets as examples until verified.
- For Apex security, consult [Secure Apex Classes](https://developer.salesforce.com/docs/platform/lwc/guide/apex-security.html) and [Set an Access Mode for Database Operations](https://developer.salesforce.com/docs/platform/apex-dev/references/apex-dev/apex-classes-enforce-usermode.html). Keep record sharing separate from object/field permissions: `with sharing` controls record-level sharing; it does not by itself enforce CRUD/FLS.
- Keep detailed coding standards in the canonical skill and ruleset files referenced above; update those files when their guidance is stale rather than duplicating large rule sets here.

---

## Architecture

- Layer order: **Trigger → Handler → Service → Selector**. Triggers are logic-free; all business logic lives in Service classes.
- **One trigger per object**. Trigger body calls only a handler class — no inline logic.
- Service classes are stateless with static methods. Selector classes centralize all SOQL per SObject.
- Use `with sharing` on every class by default. Only use `without sharing` with a documented reason. Use `inherited sharing` for utility/library classes.

---

## Apex — Critical Rules

- **API version:** Use the latest GA API version verified for the current Salesforce release and supported by the project. API 68.0 is the version recorded in project examples; verify its GA status and check for a newer release before creating metadata.
- **Never** put SOQL, DML, or HTTP callouts inside loops. Collect, query once, process in bulk.
- Always use bind variables (`:variable`) in SOQL — never concatenate user input (SOQL injection risk).
- Prefer `Database.insert/update/delete` with `allOrNone=false` when partial-success handling is required; inspect and handle every `SaveResult`.
- **Security mode:** In production code, explicitly choose `USER_MODE` or `SYSTEM_MODE` for each database operation (SOQL, SOSL, DML), even when the API-version default appears to match the intent. Prefer `WITH USER_MODE` and Database methods with `AccessLevel.USER_MODE` for user-context work. Salesforce documents user-mode as the default in API 67.0+ and system-mode in API 66.0 and earlier. System mode bypasses CRUD/FLS; record sharing still depends on the class sharing declaration. Use it only for a documented, authorized business need, and keep least-privilege record sharing. `with sharing` alone does not enforce CRUD/FLS. Treat older snippets using shorthand DML as syntax examples, not as the recommended access-mode pattern for production code.
- Use `Security.stripInaccessible()` when graceful filtering of inaccessible fields is required. Handle removed fields and object-access failures explicitly, preserve record-sharing enforcement, and do not rely on field stripping to secure fields used in filters, ordering, or other query logic.
- Use the **Safe Navigation Operator** (`?.`) to prevent NPEs: `account?.Name`. Avoid `?.` in boolean conditions — use `accounts != null && accounts.isEmpty()` instead.
- Use `switch on` instead of long `if-else` chains. Use `Assert.areEqual()` not `System.assertEquals()`.
- Prefer **Queueable** over `@future` for new async code. Attach a Finalizer when the job requires post-execution error handling or recovery.
- `@InvocableMethod` must have `label` + `description`, accept `List<Input>`, return `List<Output>`, and delegate to a Service class. One per class.
- Never hardcode IDs, org URLs, or credentials — use Custom Metadata, Named Credentials, or Custom Labels.
- Remove ad hoc `System.debug` statements before deploying. `LoggingLevel` sets severity; it does not disable or gate logging. Use an approved logger when production diagnostics are required, and avoid logging sensitive data.
- Never use empty `catch` blocks — handle, log through an approved logger, or rethrow. Use `addError()` only to report record-level validation/business errors in trigger context; use appropriate exceptions or result handling elsewhere.
- Use custom exception classes for domain logic: `public class CaseRoutingException extends Exception {}`

---

## Naming Conventions

- Classes: `PascalCase` → `AccountService`, `CaseTriggerHandler`
- Methods/variables: `camelCase` → `getAccounts()`, `accountList`
- Constants: `UPPER_SNAKE_CASE` → `MAX_RETRY_COUNT`
- Booleans: prefix with `is`, `has`, `should`, `can` → `isActive`, `hasEmail`
- Maps: `[value]By[Key]` → `contactsByAccountId`
- Triggers: `[Object]Trigger` → `AccountTrigger`. Handlers: `[Object]TriggerHandler`
- LWC folders: `camelCase` → `caseRoutingList`. HTML tags: `kebab-case` → `<c-case-routing-list>`
- LWC events: **lowercase only** → `'itemselected'`. Handlers: `handle[Event]` → `handleItemSelected`

---

## Formatting

- 4-space indentation (no tabs). Max line length: 120 characters.
- Always use braces `{}` for `if`, `else`, `for`, `while`, `do-while` — even single-line bodies.
- One variable declaration per line. Field declarations before methods in a class.
- LWC JS: `const`/`let` only (never `var`). Use ES6+ features: arrow functions, destructuring, template literals, `async/await`.
- No `console.log` in LWC production code. No `System.debug` in Apex production code.

---

## LWC — Critical Rules

- Use `lwc:if` / `lwc:elseif` / `lwc:else` — **never** the legacy `if:true` / `if:false`.
- Use `this.template.querySelector()` — **never** `document.querySelector()`. Prefer `lwc:ref` over `querySelector` for element references.
- Wire service for read operations; imperative Apex for DML/side effects. Always `async/await` with `try/catch/finally`.
- Reset loading state in `finally`. Show loading spinner (`lightning-spinner`) during all async calls.
- Event names are **lowercase only** (e.g., `'caseselected'`). Parent listens with `oncaseselected={handler}`.
- Component communication: `@api` (parent→child), `CustomEvent` (child→parent), **LMS** (siblings/cross-component) — never `window.addEventListener`.
- Never use `window.alert()`, `window.confirm()`, or `window.prompt()` — use `lightning/alert`, `lightning/confirm`, or `ShowToastEvent`.
- Always use `NavigationMixin` for navigation — never `window.location.href`.
- `@AuraEnabled(cacheable=true)` only for pure reads. No DML, callouts, or enqueue in cacheable methods.
- `@AuraEnabled` methods do **not** support overloading — use distinct method names.
- Prefer Lightning base components (`lightning-datatable`, `lightning-record-form`, etc.) and SLDS classes over custom HTML/CSS.

---

## Testing

- Apex: meet the Salesforce deployment threshold and any project-specific coverage target. If the project sets no target, prioritize meaningful assertions and coverage of behavior, bulk, permission, and error paths; do not treat 95% as a universal requirement. Use `@testSetup` where it improves test setup, and avoid `@SeeAllData=true` except for documented platform-required cases.
- Test method naming: `test_[method]_[scenario]_[expectedResult]`. Every assertion must include a descriptive message.
- Always use `Test.startTest()` / `Test.stopTest()` for governor limit resets and async execution.
- LWC: Jest tests in `__tests__/[componentName].test.js`. Clean up only DOM elements created by the test (prefer a tracked component element or shared test helper), not every child of `document.body`.

---

## Security

- For user-context SOQL/DML, prefer `WITH USER_MODE` / `AccessLevel.USER_MODE`. Use `Security.stripInaccessible()` when graceful field filtering is intended, and handle its results explicitly. Do not prescribe `WITH SECURITY_ENFORCED` as a default; use it only for a justified compatibility need after checking current Salesforce guidance.
- Never use `global` access modifier unless building a managed package or REST/SOAP endpoint — use `public`.
- Mixed DML: never combine setup objects (User, PermissionSet) and non-setup objects (Account, Contact) in the same transaction.
- Triggers: never perform callouts directly; enqueue an appropriate asynchronous process, preferably Queueable with `Database.AllowsCallouts` for new implementations.
- Never use `eval()`, `Function()`, or `innerHTML` with dynamic content in LWC.
- Never store sensitive data (tokens, passwords) in LWC component state or local storage.
