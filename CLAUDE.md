# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A maintained **fork** of `laravelcollective/html` — the HTML and Form builders for Laravel (the `Form::` and `Html::` facades, `link_to*` / `form_*` helpers). The upstream package was retired; this fork keeps the same API working on current Laravel. It targets **Laravel 12 and 13 only**, on **PHP 8.3+**. Work here is maintenance: preserving the existing public API while keeping it compatible with modern Laravel/PHP.

## Commands

```bash
composer install        # install deps (must run before tests; vendor/ is gitignored)
vendor/bin/phpunit      # run the full test suite
vendor/bin/phpunit --filter testMethodName   # run a single test
vendor/bin/phpunit tests/FormBuilderTest.php # run one test file
```

There is no linter or build step. (`.travis.yml` is stale legacy CI for the old PHP 7.2 line — ignore it.) Tests run on PHPUnit 12; `phpunit.xml` uses the PHPUnit 10+ schema (`<source>`, not the old `<filter>/<whitelist>`).

PHP 8.4+ deprecates implicitly-nullable typed parameters (`Type $x = null`), so type-hinted optional params must be written `?Type $x = null`. The two builder constructors already follow this — keep it when touching signatures.

## Test bootstrap

`phpunit.php` (the PHPUnit bootstrap, not a config) spins up an in-memory SQLite database via Eloquent's `Capsule\Manager` and creates a `models` table (`id`, `string`, `email`, timestamps). Tests that exercise model binding / `FormAccessible` rely on this schema, so a new test needing extra columns means editing `phpunit.php`.

## Architecture

Two singletons registered by `HtmlServiceProvider` and bound into the container as `html` and `form`:

- **`HtmlBuilder`** (`src/HtmlBuilder.php`) — links, assets, lists, meta/entity helpers. Depends on `UrlGenerator` and the view `Factory`. Facade: `HtmlFacade` (`Html`).
- **`FormBuilder`** (`src/FormBuilder.php`) — form open/close, inputs, selects, model binding, CSRF/method spoofing. Depends on the `HtmlBuilder`, `UrlGenerator`, view `Factory`, the CSRF token, and the `Request`; the session store is injected after construction via `setSessionStore`. Facade: `FormFacade` (`Form`).

`FormBuilder` is constructed from `app['html']`, so the form builder always wraps the same html builder instance.

### Extension points (shared traits)

Both builders `use Macroable` and `Componentable`, with the trait `__call` methods aliased so both can coexist (`Macroable::__call as macroCall`, `Componentable::__call as componentCall`). When adding/changing a builder method that overlaps with these traits, preserve that aliasing pattern.

- **`Componentable`** (`src/Componentable.php`) — lets users register reusable view-backed components (`Form::component($name, $view, $signature)`) that resolve through `__call`.
- **`Eloquent/FormAccessible`** (trait for the user's models) — provides `getFormValue()` so a model can expose a `formFooAttribute()` "form mutator" that transforms a value only when rendered in a form (distinct from Eloquent's normal accessors). Supports dot-notation into related models via `isNestedModel`.

### Blade directives

`HtmlServiceProvider::registerBladeDirectives()` auto-generates Blade directives from a hardcoded whitelist (`$directives`). For every public method on `HtmlBuilder`/`FormBuilder` whose name is in that list, it registers `@html_snake_case` / `@form_snake_case` directives. **A new builder method is NOT available as a Blade directive unless its name is added to the `$directives` array.**

### Helpers

`src/helpers.php` (autoloaded via composer `files`) defines global snake_case functions (`link_to`, `form_open`, etc.) that are thin wrappers delegating to `app('html')` / `app('form')`. Each is guarded by `function_exists()`.

## Conventions

- Builder methods return `Illuminate\Support\HtmlString` (already-escaped HTML), not plain strings.
- This is a deferred service provider (`implements DeferrableProvider` + `provides()`), so registration must stay lazy — don't add eager boot-time work that would defeat deferral.
- Match the existing PSR-2-ish style (4-space indent, doc-blocked methods) already present in the files.
