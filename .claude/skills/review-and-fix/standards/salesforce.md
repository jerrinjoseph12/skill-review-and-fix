# Salesforce Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers Apex (`*.cls`, `*.trigger`), SOQL and SOSL, Lightning Web Components (`lwc/`), Aura, Visualforce (`*.page`, `*.component`), Flows (`*.flow-meta.xml`) and metadata in an SFDX project.

## Authorities
- Apex Developer Guide (execution governors and limits, sharing, security)
- Salesforce Secure Coding Guide; Lightning Web Security docs; LWC developer guide
- Salesforce Well-Architected; AppExchange Security Review requirements
- Trailhead and Apex Enterprise Patterns (fflib), if the project uses them

## Tooling to look for and run
- `sf code-analyzer run`: Salesforce Code Analyzer v5, which bundles PMD for Apex, ESLint for LWC, RetireJS, regex rules and Flow scanning. The older v4 command was `sf scanner run`.
- `npm run test:unit` (`sfdx-lwc-jest`) and `npm run lint` for LWC.
- `sf apex run test --test-level RunLocalTests --code-coverage` needs an org. **Ask before running it, and never deploy (`sf project deploy start`) without explicit permission.**

## Version awareness
Read `sourceApiVersion` in `sfdx-project.json` and `<apiVersion>` in each `-meta.xml`. Behavior depends on the API version: `WITH USER_MODE`, the `Assert` class, and Lightning Web Security vs. Locker. Flag very old API versions on changed files as `[L]`.

## Security
- `[C]` Dynamic SOQL or SOSL built by concatenating user input: `Database.query('... ' + input)`. Use bind variables, `Database.queryWithBinds`, or `String.escapeSingleQuotes` as a last resort.
- `[C]` Secrets, passwords or tokens in Apex, custom labels, unprotected custom settings or static resources. Use **Named Credentials / External Credentials**.
- `[H]` CRUD and field-level security not enforced on user-facing data access. Apex runs in **system mode** by default. Use `WITH USER_MODE` / `AccessLevel.USER_MODE` (preferred over `WITH SECURITY_ENFORCED`, and don't mix the two), or `Security.stripInaccessible`.
- `[H]` Classes with no sharing keyword, or with `without sharing` and no documented reason, when they serve `@AuraEnabled`, REST or Visualforce callers. Default to `with sharing` or `inherited sharing`.
- `[H]` `@AuraEnabled` or `@RestResource` methods that take record IDs without checking access (IDOR).
- `[H]` XSS: Visualforce `escape="false"`; merge fields used in JavaScript without `JSENCODE`; LWC `lwc:dom="manual"` with `innerHTML` set from data; Aura `$A.util` HTML injection.
- `[M]` `@AuraEnabled(cacheable=true)` on methods that change data, or that return user-specific sensitive data cached on the client.

## Governor limits and bulkification
- `[C]` SOQL or DML inside loops in triggers or batch code. One bulk insert of 200 records hits the 100 SOQL or 150 DML statement limits.
- `[H]` Triggers written for one record (`Trigger.new[0]`); more than one trigger per object; logic in the trigger body instead of a handler class.
- `[H]` Trigger recursion with no guard; recursive record-triggered Flows and triggers updating each other.
- `[H]` Non-selective SOQL on large objects (no indexed filter; leading `%` LIKE; `!= null` filters); unbounded queries without `LIMIT` in user-facing code.
- `[H]` Mixed DML (setup and non-setup objects) in one transaction without `@future` or Queueable separation.
- `[M]` `@future` called from batch or future contexts, which isn't allowed; heavy work done synchronously when it should be Queueable or Batch Apex; callouts after uncommitted DML.

## Correctness
- `[H]` Hardcoded record IDs, profile names, org URLs or record type IDs. Use custom metadata, `Schema.SObjectType...getRecordTypeInfosByDeveloperName()` or labels.
- `[H]` `catch (Exception e) {}` that swallows DML errors; partial-success `Database.insert(records, false)` with the `SaveResult` errors ignored.
- `[M]` `Decimal` or `Double` rounding mistakes with currency; `Date` vs `Datetime` time-zone confusion between users and GMT.

## Flows
- `[H]` Get, Create, Update or Delete Records elements inside loops.
- `[H]` No **fault paths** on elements that can fail; hardcoded IDs.
- `[M]` Before-save logic placed in an after-save flow, which adds extra DML; a flow running in system context without sharing where user context was intended.

## LWC and Aura
- `[H]` `@wire` results used without handling `error`; imperative Apex calls without `catch`.
- `[M]` Mutating `@api` properties inside the component; DOM queried outside `this.template`; event listeners added in `connectedCallback` without being removed in `disconnectedCallback`.

## Tests
- `[H]` `@isTest(SeeAllData=true)` without a reason; tests with no assertions written only to reach the **75% coverage** deploy threshold.
- `[H]` Bulk paths not tested with 200+ records; no `Test.startTest()` / `stopTest()` around async code; callouts without an `HttpCalloutMock`.
- `[M]` Permission and sharing paths not tested with `System.runAs`; `System.assert(true)`-style checks instead of `Assert.areEqual` with a message.

## Maintainability
- `[M]` Business logic in triggers or controllers instead of service or domain classes, where the project uses a layered pattern such as fflib.
- `[L]` Debug statements (`System.debug`) left in hot paths; naming that doesn't match the org's conventions.
