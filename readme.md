![LaravelCollective HTML](LaravelCollectiveHTML-banner.png)

# HTML and Form Builders for Laravel

A maintained fork of [`laravelcollective/html`](https://github.com/LaravelCollective/html) that keeps the `Form::` and `Html::` builders working on current Laravel.

This fork targets **Laravel 12 and 13** on **PHP 8.3+**. The original package has been retired; if you do not need to keep the legacy API and are starting fresh, [`spatie/laravel-html`](https://github.com/spatie/laravel-html) is a good alternative.

## Installation

Add the fork as a repository in your application's `composer.json`:

```json
"repositories": [
    {
        "type": "vcs",
        "url": "https://github.com/jago86/html"
    }
]
```

Then require it:

```bash
composer require laravelcollective/html:^12.0
```

The service provider and the `Form` / `Html` facades are auto-discovered.

## Documentation

The original Forms & HTML API is unchanged. Refer to the [legacy LaravelCollective documentation](https://laravelcollective.com/docs/6.0/html) for usage.

## Upgrade notes (Laravel 12/13)

### `Form::selectMonth()` format string

The `$format` argument of `selectMonth()` now uses a PHP [`date()`](https://www.php.net/manual/en/datetime.format.php) format string instead of the old `strftime()` format (the underlying `strftime()` is deprecated as of PHP 8.1 and removed in PHP 9). The default changed from `'%B'` to `'F'`, and month names are now localized through Carbon (the app locale) rather than the system locale.

```php
// Before (strftime format)
Form::selectMonth('month', null, [], '%B'); // full month name
Form::selectMonth('month', null, [], '%b'); // abbreviated month name

// After (date() format)
Form::selectMonth('month');                 // default, full month name ('F')
Form::selectMonth('month', null, [], 'M');  // abbreviated month name
```

Calls that rely on the default argument require no changes.

## License

This package is open-sourced software licensed under the [MIT license](LICENSE.txt).
