# PHP Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers plain PHP, Laravel, Symfony and WordPress.

## Authorities
- PHP manual (security section)
- PHP-FIG: PSR-12 and the PER Coding Style
- OWASP PHP Configuration Cheat Sheet
- Laravel and Symfony security docs; WordPress Coding Standards and its plugin security handbook

## Tooling to look for and run
- `composer test` or `vendor/bin/phpunit` / `vendor/bin/pest`
- PHPStan or Psalm
- PHP_CodeSniffer (`phpcs`) or PHP-CS-Fixer
- `composer audit`

## Version awareness
Read `require.php` in `composer.json`. Don't suggest features newer than the target, such as enums and readonly properties (8.1) or property hooks (8.4).

## Security
- `[C]` SQL by concatenation in `mysqli_query`, `PDO::query`, Laravel `DB::raw` / `whereRaw` / `selectRaw`, or WordPress `$wpdb->query` without `$wpdb->prepare`. Use prepared statements or bindings.
- `[C]` `unserialize()` on user-controlled data (object injection). Use `json_decode`, or `['allowed_classes' => false]`.
- `[C]` `eval`, `assert` with strings, `preg_replace` with `/e`, or `include` / `require` with user input (local or remote file inclusion).
- `[C]` `exec`, `shell_exec`, `system`, `passthru` or backticks with input not passed through `escapeshellarg`.
- `[C]` `APP_DEBUG=true`, `display_errors=On` or Symfony's debug mode in production config; `.env` files committed with secrets.
- `[H]` XSS: `echo $_GET[...]` or output without `htmlspecialchars($x, ENT_QUOTES, 'UTF-8')`; Blade `{!! $x !!}`; Twig `|raw`; WordPress output not escaped with `esc_html` / `esc_attr` / `esc_url`.
- `[H]` Passwords hashed with `md5`, `sha1` or `crypt` instead of `password_hash` / `password_verify`; secrets compared with `==` instead of `hash_equals`.
- `[H]` `rand`, `mt_rand` or `uniqid` used for tokens. Use `random_bytes` or `random_int`.
- `[H]` File uploads that trust the client's MIME type or extension, or that are stored in the web root where they can be executed.
- `[H]` Laravel mass assignment (`$guarded = []`, or `$request->all()` passed to `create` or `update`); routes missing `auth` middleware or policy checks; WordPress actions without nonce (`check_admin_referer`) and capability (`current_user_can`) checks.
- `[H]` `extract($_POST)` or `$$var` with user input.
- `[M]` Session not regenerated at login (`session_regenerate_id(true)`); cookies without `Secure`, `HttpOnly` or `SameSite`.

## Correctness
- `[H]` Loose comparison bugs with `==`: `"abc" == 0` (true before PHP 8), magic hashes like `"0e1" == "0e2"`, `in_array` or `array_search` without `strict: true`. Use `===`.
- `[H]` `if (strpos($h, $n))` fails when the match is at position 0. Use `!== false` or `str_contains` (PHP 8+).
- `[H]` Money in `float`. Use integer cents or `bcmath` / `brick/money`.
- `[M]` Missing `declare(strict_types=1);` in a codebase that uses it; untyped parameters or returns in new code.
- `[M]` `DateTime` used without an explicit time zone; mutable `DateTime` changed by accident (prefer `DateTimeImmutable`).
- `[M]` Errors suppressed with `@`.

## Performance
- `[H]` N+1 queries with Eloquent or Doctrine in loops or views. Use `with()` or fetch joins.
- `[M]` Loading whole tables (`all()`, `get()`) where `chunk` or `cursor` would do; queries inside Blade or Twig templates.

## Maintainability
- `[L]` Not following PSR-12 / PER where the project uses it; business logic in controllers or templates; dead code; a missing `final` or `readonly` where the project's convention uses them.
