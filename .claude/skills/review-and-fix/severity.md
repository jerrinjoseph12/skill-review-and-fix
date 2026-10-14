# Severity Rubric

Assign severity by **impact × likelihood** in realistic use, not by category. A security issue that is unreachable may be Low. A small logic slip on the checkout path may be Critical.

When unsure between two levels, choose the lower one and say why in the finding's Scenario. Overstated severity erodes trust in the whole report.

## Critical
Ship-blocking. Causes serious harm in normal use, or is easy to exploit.

- Remote code execution, SQL, command or path injection reachable from untrusted input
- Authentication or authorization bypass; one user can read or change another user's data
- Secrets, credentials or private keys committed to source
- Data loss or corruption on a common path (for example, delete without a WHERE clause, a migration that drops data)
- Crash or hang of the whole service on ordinary input
- Money, billing or inventory computed wrongly in normal flows

## High
Must fix before merge. Real defect with significant impact, or a Critical-class issue that requires unusual conditions.

- Logic error producing wrong results for a plausible subset of inputs
- Unhandled error or null on a reachable path that crashes a request or job
- Race condition or lost update under normal concurrency
- XSS, SSRF, open redirect, or sensitive data (PII, tokens) written to logs
- Resource leak (connections, file handles, listeners) that builds up over time
- Breaking change to a public API or contract without versioning
- N+1 or unbounded query on a hot path that will degrade in production

## Medium
Should fix. Correct today but fragile, or wrong only in edge cases.

- Edge cases mishandled: empty collections, zero, boundaries, unicode, timezones, very large inputs
- Missing input validation where downstream code happens to tolerate it
- Inefficient algorithm or repeated work off the hot path
- Error swallowed or logged without context, which makes failures hard to diagnose
- New behavior with no tests
- Duplicating an existing helper or utility in the codebase

## Low
Nice to fix. No behavioral impact.

- Misleading names or comments, dead code, unused imports or variables
- Minor departures from project conventions not caught by the linter
- Small readability improvements, such as simplifying a condition or removing a needless temporary

## Not a finding

Leave these out of the report entirely:
- Pure formatting the project's formatter would handle
- Personal style preferences with no convention in the codebase to back them
- Speculative issues with no concrete failure scenario
- Problems in code outside the review scope that the change doesn't touch or make worse
