# Appian Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

## Authorities
- Appian documentation: SAIL functions, design best practices, the Health Check, and Designer recommendations
- Appian's accessibility checklist
- **The local `appian` skill, if installed** (or `appian-26-3` for 26.3 environments). Load it for function and component signatures, `queryRecordType` patterns, null-safety rules and accessibility rule IDs. Don't guess function names or parameters. Unknown or invented functions are a common source of defects.

## Where Appian code lives
- **Exported application XML** (`content/*.xml`, with SAIL inside `<definition>` or CDATA): review the SAIL. **Don't hand-edit UUIDs, version info or object references.** Mark fixes that need structural XML changes as **needs-decision**, and recommend making them in Appian Designer.
- **Live environment via an Appian MCP server:** reading for review is fine. **Writing fixes changes a shared environment.** List the exact objects and changes, and get the user's confirmation before calling any create, update or delete tool. Validate expressions (`validateExpression`) before updating objects, and run the object's test cases afterwards if it has them.
- **Plug-ins** (`appian-plugin.xml` plus Java): apply [java.md](java.md) as well. Check that only supported Appian SDK APIs are used.

Check the Appian version. Function behavior changes between versions, for example the 25.2 `a!queryRecordByIdentifier` change described below.

## Security
- `[C]` Access is enforced only by `showWhen` or visibility in the interface, while the underlying query, record action, Web API or process model is open to everyone. Hiding UI isn't access control.
- `[C]` Credentials or API keys hardcoded in integrations, constants or expressions. Use a Connected System with secured credential fields.
- `[H]` Record types where different roles need different data but no record-level security is configured. By default, all Viewers see all records.
- `[H]` A Web API exposing data or actions without a group restriction suited to the caller; process models startable by "All Users" when they shouldn't be.
- `[H]` User input concatenated into `a!queryEntity` or `a!queryRecordType` filters as text expressions, or into SQL in a Query Database node or stored procedure (injection).
- `[M]` Individual users in role maps instead of groups; group names or IDs hardcoded instead of held in constants; overuse of **Deny** permissions.
- `[M]` Sensitive data in process variables that aren't hidden, or written to logs or notification emails.

## Correctness (SAIL)
- `[H]` **Functions that reject null** called on values that may be null:
  - `user()`, `group()`, `text()` with date or number formats, `upper()`, `lower()`, `left()`, `right()`, `find()`, `not()`
  - Wrapping the value in `a!defaultValue()` does **not** prevent the error. Guard with `if(a!isNotNullOrEmpty(...), ..., fallback)`.
- `[H]` `and()` and `or()` **don't short-circuit**. Using them to guard property access (`and(a!isNotNullOrEmpty(x), x.field = 1)`) still evaluates `x.field`. Use nested `if()`.
- `[H]` Indexing into a possibly empty array (`local!items[1]`, `.field` on empty results). Use `index(array, i, default)` or a guard.
- `[H]` `a!writeRecords` or `a!startProcess` without `onError`, so failures are silent. Also flag `submit: true` with no `saveInto` effect.
- `[H]` Record writes: an update without the primary key creates a duplicate, and a create that sets the primary key causes an error or overwrites.
- `[H]` `a!queryRecordByIdentifier` (25.2 and later) without `fields` returns **only the primary key**. Other fields come back null.
- `[H]` One-to-many related fields without `a!relatedRecordData(..., limit: n)`. The default is **10**, and the extra records are silently cut off.
- `[H]` Query filters that use variables but have no `applyWhen` guard. A null value then filters incorrectly.
- `[H]` Process models: XOR or OR gateways with no default path (the process stalls); no error handling or alerts on integration or Write Records nodes; MNI with no upper bound.
- `[M]` Functions that don't exist or are obsolete, such as `regexmatch()`, `property()`, `a!recordQuery()`. Legacy `load()` and `with()` in new code should be `a!localVariables()`.
- `[M]` `a!recordData()` stored in a `local!` variable. It's meant only for a grid or chart `data` parameter.

## Performance
- `[H]` Queries inside `a!forEach`, grid column expressions, or rules called once per row (N+1). Query once, then index in memory.
- `[H]` `pagingInfo` with `batchSize: -1`, or no paging, on tables that can grow.
- `[H]` Rules or interfaces that run slowly. The Appian Health Check flags rules averaging over **2000 ms** as high risk.
- `[M]` `a!queryRecordType` without `fields` returns **all** fields. Select only what's needed.
- `[M]` `a!queryRecordType` used for a single known ID. Use `a!queryRecordByIdentifier`.
- `[M]` The same query repeated across an interface and its child rules. Pass the data down as a rule input.
- `[M]` Heavy data loaded synchronously on dashboards. Consider `a!asyncVariable` or deferred loading.
- `[M]` Large data or documents kept in process variables; long unattended activity chains (chaining stops after 50 activities).
- `[L]` `refreshAlways: true` or `refreshInterval` where a referenced-variable refresh would do.

## Accessibility and UX (Appian accessibility checklist)
- `[H]` Inputs with no `label` (use `labelPosition: "COLLAPSED"` to hide it visually; don't leave it null); grids without a `label` or a `rowHeader`; icon-only buttons or links with no `altText` or `accessibilityText`.
- `[M]` Required fields not marked `required: true`; format hints put in rich text instead of `instructions`; headings not semantic (use `a!headingField` or `labelHeadingTag`); expandable sections without a label.
- `[M]` A Form used for read-only detail pages (use a Summary View); separate create and edit forms where one form with an `isUpdate` flag would do.
- Organization rule, from the local `appian` skill's accessibility rules: `a!dateTimeField` is prohibited. Use separate date and time inputs.

## Maintainability
- `[M]` Hardcoded values (statuses, group names, URLs, thresholds) that should be constants; duplicated logic that should be a reusable expression rule.
- `[M]` New expression rules with no test cases, when the application uses them.
- `[L]` Naming that breaks the app-prefix convention (`ABC_RuleName`); objects missing a description; rule inputs not documented; very large interfaces that should be split into sub-interfaces.
