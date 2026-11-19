# 📐 Laravel Rules for Cursor

**Give Cursor a shared set of Laravel and PHP coding instructions.**

This repository contains one customizable [`rule.mdc` file](rule.mdc).

- 🗂️ **Organize code:** guidance for project structure and business logic.
- ⚡ **Work with data:** database queries, Eloquent and performance.
- ✅ **Check quality:** security, validation and Pest tests.

Use it as a starting point for your own team conventions.
These are opinionated instructions for the AI assistant, including custom formatting rules.
They do not automatically enforce code quality.

**Use it:** copy `rule.mdc` into your project’s `.cursor/rules/` folder, then edit it to fit your conventions.

<details>
<summary><strong>Read the full coding rules</strong></summary>

```yaml
alwaysApply: true
```

## 1) Core Engineering Standards (Always)

- Write production-grade code as a world-class developer + software architect (long-term thinking, maintainability first).
- Prevent unintended side effects: explicitly handle edge cases, state changes, concurrency, and failure modes.
- Prioritize **readability + low cognitive load**:
  - Code should “tell a story” from start to finish (expressive names, clear flow, minimal surprises).
  - Prefer small, single-purpose methods/classes; keep orchestration explicit.
- **Security is a first-class priority**:
  - Proactively prevent SQL injection, XSS, CSRF, mass-assignment, auth/permission flaws.
  - Auto-fix when safe; otherwise **alert the user** with risks + mitigation.
- **Reuse and consistency**:
  - Apply **SRP + DRY** everywhere possible.
  - “Group code by purpose, not by type” → use meaningful folder structure and domain boundaries.
  - Don’t hard-code roles/types/logic in one place; prefer parameters, enums, abstractions, configuration.
  - Keep logic close to the behavior it belongs to (high cohesion).
- Performance mindset:
  - Optimize loops + DB queries; avoid unnecessary nested loops.
  - Load only necessary dependencies.
  - Write reusable queries (avoid copy/paste query fragments).
- Keep Eloquent models lean: relationships + small atomic behaviors only; avoid orchestration/side-effect hubs.

### SOLID (Must Follow Wherever Possible)
- **Separation of concerns**: Business logic stays out of controllers and (most) models.
- **Scalable architecture**: Extend without breaking existing features.
- **Readable code**: Express *what* and *why*, not just *how*.
- **Testable components**: Small focused classes, clean boundaries.
- **Modular structure**: Work in one domain without touching the rest.

---

## 2) Project Structure (Laravel + DDD)

### Baseline Folders (MVC)
- **Models** → `app/Models` *(Data & DB logic)*
- **Controllers** → `app/Http/Controllers` *(HTTP request handling only)*
- **Views** → `resources/views` *(Blade templates)*

### Domain Driven Design (Primary Structure)
- We use DDD in all Laravel apps whenever possible.
- Organize code by what it means to the business (domains/modules).
- Controllers are always thin:
  - Controllers act as the HTTP layer only.
  - Controllers delegate business logic to domain Actions/classes.

**DDD Root**
- **Domain** → `Src/Domain/{Domain}` *(Business logic)*
- **App** → `Src/App` *(misc/shared app-level code)*

### Other Common Folders
- Middleware → `app/Http/Middleware`
- Requests → `app/Http/Requests`
- Jobs → `app/Jobs`
- Events/Listeners → `app/Events`, `app/Listeners`
- Helpers/Traits → `app/Helpers`, `app/Traits`

### Configuration
- Use `.env`
- General settings & creds → `config/`

---

## 3) Domain Module Conventions (Src/Domain/{ModuleName})

Each folder under `src/Domain/{ModuleName}/` has a purpose:

e.g: src/Domain/Invoices/
├── Actions
├── QueryBuilders
├── Collections
├── DataTransferObjects
├── Events
├── Exceptions
├── Listeners
├── Models
├── Rules
├── Enums
└── States

- **Actions/**: Execute one business operation with side effects (persist, notify, dispatch); no raw queries/formatting.
- **QueryBuilders/**: Reusable DB-level query logic for Eloquent; not in-memory filtering or loaded-model behavior.
- **Collections/**: Behavior over already-loaded sets (aggregate/batch actions); never perform DB queries.
- **DataTransferObjects/**: Immutable typed data carriers between layers; no validation/persistence/logic.
- **Events/**: Passive domain occurrences; no behavior—only trigger listeners.
- **Exceptions/**: Domain-specific business failures; not generic framework/HTTP exceptions.
- **Listeners/**: React to events with side effects (email/log/jobs); never mutate core business data.
- **Models/**: Domain entities tied to DB (relationships + small atomic behaviors); not orchestration hubs.
- **Rules/**: Reusable **input validation** rules; not business rules or data access.
- **States/**: Model lifecycle transitions via state logic; avoid if a simple flag suffices.
- **Controllers**: HTTP only; delegate business logic to the domain layer.

---

## 4) PHP Rules

- Prefer `match` over `switch`.
- Use PHP 8 constructor property promotion.
  - Don’t create an empty constructor if there are no parameters.
- Add function return types whenever possible.
- Import all classes with `use` and reference only short names (no fully-qualified names in code).
- Enums:
  - Generate Enums in `app/Enums` (not `app/`), unless instructed otherwise.

---

## 5) Laravel Rules (Performance, Architecture, Style)

### Performance & Data Access
- Cache where it improves performance; **invalidate/clear cache** on create/update/delete where required.
- Aggressively avoid N+1:
  - Eager load relationships.
  - Use `chunk()`/`cursor()` for large datasets.
- Reduce query count:
  - Use `select()` to fetch only required columns.
  - Suggest indexing frequently queried columns.
  - Avoid duplicate queries; fetch required data in as few queries as possible.

### Architecture & Controllers
- Controllers must be slim and delegate to domain Actions/services.
- Use dependency injection (“Let Laravel give you what you need.”) instead of manual instantiation.
- Use structured exception handling when needed: try-catch-finally.

### Type Safety & Language Conventions
- Prefer `?Type` over `Type|null` (e.g., `?string $variable`).
- Use `void` for methods returning nothing.
- Type properties whenever possible (`public string $bar;` not `public $bar;`).
- Enums:
  - Enum values use PascalCase.
  - If a PHP Enum exists for a domain concept, use its cases (or `->value`) everywhere (routes, middleware, migrations, seeds, config, UI defaults).
- If a variable has multiple types, put the most common type first.
- Constructor promotion:
  - Put each promoted property on its own line.
  - Use a comma after the last one.
- Traits:
  - Each applied trait on its own line, with `use` repeated per trait.
- Prefer string interpolation over `sprintf()` and `.` when possible.
- Ternary formatting:
  - Each part on its own line unless very short.
- Control flow:
  - Unhappy path first, happy path last.
  - Prefer separate `if` statements over complex compound conditions.
- Formatting:
  - Add blank lines between statements (unless a tight sequence of equivalent single-line operations).
  - Code should be expressive, professional, and readable.

### Naming & Conventions
- Configuration keys must use `snake_case`.
- Resource controllers:
  - Use plural resource name and standard CRUD methods (`index/create/store/show/edit/update/destroy`).
  - Extract a new controller for non-CRUD actions.
- Form Requests:
  - Validate via Form Requests and use `$request->validated()`.
  - When multiple rules exist for a field, use array notation (avoid `|`).

**Naming specifics**
- Controllers: plural + `Controller` suffix (e.g., `UsersController`, `EventDaysController`).
  - Single-action controllers: action name + `Controller` (invokable).
- Resources/Transformers: plural + `Resource`/`Transformer`.
- Jobs: name describes action (e.g., `CreateUser`).
- Events: tense matters (before: `ApprovingLoan`, after: `LoanApproved`).
- Listeners: action + `Listener` suffix (e.g., `SendInvitationMailListener`).
- Commands: suffix with `Command` (e.g., `PublishScheduledPostsCommand`).
- Mailables: suffix with `Mail` (e.g., `AccountActivatedMail`).
- Enums: no prefix usually (e.g., `OrderStatus`).

### Laravel Guide (Concrete Rules)
- Pivot tables: alphabetical naming (e.g., `project_role`, not `role_project`).
- Local environment: Laravel Herd → assume project URL `http://[folder_name].test`
- Observers:
  - Register using PHP Attributes on the model, not `AppServiceProvider`.
  - Example: `#[ObservedBy([UserObserver::class])]` with `use Illuminate\Database\Eloquent\Attributes\ObservedBy;`
- Validation:
  - Controllers: validation in Form Request; use `$request->validated()`.
- Service classes:
  - Put larger logic pieces in Service classes when appropriate (while still preferring domain Actions for business operations).
- Prefer Laravel helpers over facades when possible:
  - `auth()->id()` not `Auth::id()`
  - `redirect()->route()` not `Redirect::route()`
- Query style:
  - Don’t use `whereKey()` / `whereKeyNot()`; use explicit fields like `id`.
  - Don’t use `::query()` for `create()` (use `User::create()`).
- Livewire:
  - Use Livewire class components only (no Livewire Volt).
- Pivot timestamps:
  - If migration uses `timestamps()`, then add `withTimestamps()` on `BelongsToMany`.
- Migrations:
  - When adding columns, update `$fillable` accordingly.
- Controllers routing:
  - Single-method controllers use `__invoke()`.
  - Multi-method REST controllers use `Route::resource()->only([])`.

---

## 6) Laravel 11+ Skeleton Rules (Must Follow)

- Service Providers:
  - Only `AppServiceProvider` by default.
  - Don’t create new providers unless absolutely necessary.
  - If needed, register in `bootstrap/providers.php` (not `config/app.php`).
- Event Listeners:
  - Auto-discovered when type-hinted correctly (Laravel 11 behavior).
- Scheduler:
  - Put scheduled commands in `routes/console.php` (no `app/Console/Kernel.php`).
- Middleware:
  - Prefer middleware by class name in routes.
  - If alias needed, register in `bootstrap/app.php` (no `app/Http/Kernel.php`).
- Tailwind:
  - Use Tailwind for new Blade pages (not Bootstrap) unless instructed otherwise.
- Factories:
  - Use `fake()` helper (not `$this->faker`).
- Policies:
  - Auto-discovered; do not register manually.

---

### 🚀 Transformers

Transformers are small, focused classes that convert **raw domain entities** into **frontend-ready arrays** for Blade, Vue, or APIs. They handle presentation shaping only, so views receive clean, usable data with zero formatting logic.

- **Input**: Domain entities (Website, Category, SeoStats)
- **Output**: Plain arrays ready to render
- **Responsibility**: Presentation shaping, not business logic
- **Goal**: Keep controllers thin and views dumb
- **Rule**: If it’s for display, it belongs in a transformer

Think: domain produces data, transformers prepare it, views just render.

---

## 7) Frontend Security (Baseline)

- Always validate user inputs to prevent SQL injections, XSS, and CSRF attacks.
- Always escape HTML output to prevent XSS attacks.

---

## 8) Comments, Spacing, and Readability Rules (Mandatory)

This guideline defines strict rules for line spacing, comments, and naming to keep code clean, readable, and visually consistent. These rules are non-standard but mandatory because they improve clarity and speed of understanding.

🚨 Follow strictly. No exceptions.
- 5 blank lines between class methods.
- 2 blank lines between major code blocks inside a method.
- 1 blank line between logical steps within the same block.

- Comments must be minimal but fully explanatory.
- Always explain what the code does and how when needed.

### Comment Quality
- Titles must be clear, short, and action-based.
- Descriptions must explain what happens and when.
- Avoid vague, generic, or non–title-cased comments.

### Variable Naming
- Names must describe purpose, not type.
- Be specific and contextual.
- Use camelCase.
- Avoid vague, generic, or redundant names.
- Prefer clarity over brevity.

Naming Checklist:
- Does the name explain its role?
- Is it understandable without context?
- Is it consistent with the codebase?

### Import Organization
- Group imports logically: controllers → models → services → others.
- PHP Imports should be combined in {example,example-two} format where possible to reduce number of lines and increase clarity.

### Date
- Update date in comment block whenever method defination is updated. Same for other files.

### Class Method Comment Style (Mandatory)
- Comment goes directly above the method and no line break between.
- Start with an UPPERCASE title.
- Explain behavior using short bullet points.
- Keep lines within the top asterisk width.
- Break long lines using `..` to indicate continuation.
- Separate logical sections with one blank line.
- Always document inputs (name + type) and return type.
- JS:
  - Add comments block to group specific related code group
  - Line breaks:
    - Add three line breaks between each code block
    - Add two line breaks in code block when two are relevant but bit different to increase readability.
    - Add one line break else in between same block when make sense.
- Always use this comment style and not genric php styles, example below:

PHP EXAMPLE:
```php
/*****************************************************************
 * COMMENT TITLE IN UPPERCASE - Short, concise, descriptive
 *****************************************************************
 *
 * Define what function is being used for
 * - use bullet list where needed for further details
 *
 * > you can also use > this as well to explain different sections
 *
 * If comment is longer than top* line then break it into next line 
 * .. like where two dots represent that line is part of previous one.
 * 
 * @Author: Developer name (comma separated if more than one, in order)
 * @LastUpdated: 01 Jan 2026
 *
 * @param $variable/object name - return type
 * @parm $title string
 * @return $variable/object name - return type
 * @return $websites - collection / array / model
 *
*****************************************************************/
public function example()
```

JS Example:
```js
/*********************************************************************
 * Axios HTTP Library
 **********************************************************************
 * We'll load the axios HTTP library which allows us to easily issue requests
 * to our Laravel back-end. This library automatically handles sending the
 * CSRF token as a header based on the value of the "XSRF" token cookie.
 *********************************************************************/
import axios from 'axios';
```

### Comments Inside Methods
- Separate blocks with 2 blank lines.
- Each block starts with `// -----------------------`
- Follow with a short Title Case description.
- Optional extra lines for context.
- Inline comments allowed for special cases.
- JS:
  - separate each code block by using `// ===` comment structure

Example 1:
```php
// -----------------------
// Filters
$search = $request->input('search');
```

Example 2:
```php
// -----------------------
// Apply Search Query
// search filter: match term against website domain or site title or contact email.
// @param string $search The search term to filter websites by
if (! empty($search)) {
```

JS Example:
```js
// ==========================================================
// Imports
// ==========================================================
import { example } from 'vue';



//==========================================================
// Props & Data
//==========================================================
const props = defineProps({
```

### Top Of The Code: PHP + JS
- Use an UPPERCASE title + 2–3 line summary.
- Summarize code in few sentences and place it at top of file or class declaration.
- Placement:
  - **PHP**: ace above class declaration when context is needed.
  - **JS:** Always at the very top of the file.

PHP Example:
```php
/*********************************************************************
 * COMMAND: UPDATE WEBSITE SEO METRICS
 *********************************************************************
 * Update SEO metrics for all active marketplace websites perodically.
 * Dispatches jobs to fetch SEO stats for each active website in the 
 * marketplace based on last updated date.
**********************************************************************/
class example extends example{}
```

---

## 9) 🌍 Translation Guide (Vue + Blade) — Mandatory for Public UI Text

All **public-facing text must use translations**. No hardcoded strings. Ever.  
These rules must be followed automatically when editing or creating **Vue or Blade files**.

### Blade Translation Handling
- Every user-visible string goes through translations.
- Always use the `__()` helper.
- Do **not** use `@lang` in Blade views.
- Reason: `__()` works in **both Blade and PHP**, keeping things consistent.
consume translations **from Laravel**, never call `__()` directly
- **All public-facing text must be translated**
- **No hardcoded UI strings** in Vue, Blade, or PHP
- Always use **Laravel translations**

### Vue Translation Handling
- Do NOT hardcode strings
- Do NOT call __() in Vue
- Always read from shared translations
- Use Inertia to share full translation files
- Example: ```php
    Inertia::share('i18n', fn () => [
        'newsletter' => trans('newsletter'),
    ]);
    ```
- Translation Example: `<h2>{{ $page.props.i18n.newsletter.form.title }}</h2>`
- Translation file → Inertia share → Vue consumes → View renders

### Guidance
- Organize by **feature/page**, not by component. If file does not exist, **create it**.
- Add translations to lang files (create if missing).
- Translation must be passed to Vue from Laravel (not handled directly in Vue/JS).
- Auto-create translation keys and professional translations for major langs when possible. English is required.
- Location: `resources/lang/{lang}/`

When encountering UI text in Vue or Blade:
1. Generate a translation key using feature-based dot notation
2. Check `resources/lang/en/{feature}.php`
3. If file/key does not exist → create it
4. Add English translation using professional wording
5. Replace UI string with the correct translation call
6. For Vue, ensure translation is available via shared `i18n`

Blade Usage:
```blade
<h2>{{ __('newsletter.form.title') }}</h2>
{!! __('newsletter.form.description') !!}
```

Organise by feature example:
```txt
resources/lang/en/newsletter.php
```

---

## 10) Code Optimisation (When Asked)

When asked for optimisation, optimize for **performance + readability** without changing behavior.

- Reduce DB queries as much as possible.
- Cache where possible and ensure correct invalidation.
- Store config variables in the project config (not scattered constants/classes). Group config using comments + line breaks.
- Refactor:
  - Break long methods into smaller methods where it improves clarity without disrupting existing usage.
- Improve variable names within the same file; ensure renames are updated everywhere used.
  - Do **not** rename variables that form external contracts (e.g., JSON keys) if that risks breaking dependencies.
- Security review:
  - Auto-fix safe issues; otherwise report problem + solutions clearly.
- After changes:
  - Tell the developer what changed and suggestions for improvement in simple English (clear enough for ~100 IQ).

---

## 11) Testing (Mandatory Defaults)

- Use **Pest**, not PHPUnit.
- Run tests with: `php artisan test`.
- Every test uses **Arrange – Act – Assert**.

Arrange:
- Use factories with meaningful values + names (improve debugging clarity).
  - Bad: `$user1 = User::factory()->create();`
  - Better: `$adminUser = User::factory()->create(['email' => 'admin@admin.com']);`

Assert (when applicable):
- HTTP: `assertStatus()`
- Response structure/data: `assertViewHas()`, `assertSee()`, `assertDontSee()`, `assertJsonContains()`
- Redirect + flash: `assertRedirect()`, `assertSessionHas()`
- DB: `assertDatabaseHas()`, `assertDatabaseMissing()`, `expect($variable)->toBe()` etc.

---

## 12) Execution Tips (Expected Model Behavior)

- Always suggest improvements and optimization opportunities beyond the immediate request.
- Focus on: low cognitive load, readability, reuse, SOLID/SoC, fewer queries, less code without sacrificing clarity.
- Maintain modular, reusable components and meaningful naming.
- Follow industry style guides (PSR-12 for PHP, AirBnB for JS) unless these rules override.
- Use braces for control structures where they make sense.
- Avoid packing multiple instructions into one line of code.
- Always validate user input using Form Requests.
- Auto-create or update tests whenever code is written/optimized.

</details>
