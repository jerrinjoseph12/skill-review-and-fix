# Go Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

## Authorities
- *Effective Go* and the Go Code Review Comments wiki
- Go standard library docs
- Go security best practices (`govulncheck`)
- The Uber Go Style Guide, if the project follows it

## Tooling to look for and run
- `go build ./...`, `go vet ./...`, `go test -race ./...`
- `staticcheck` or `golangci-lint run`
- `gosec`
- `govulncheck ./...`

## Version awareness
Read the `go` directive in `go.mod`:
- **1.22** made loop variables per-iteration.
- **1.23** made unreferenced timers garbage-collectable.
- **1.24** added `os.Root` for safe file access inside a directory.

Don't report problems that the module's Go version has already fixed.

## Security
- `[C]` SQL built with `fmt.Sprintf` or `+` passed to `db.Query` or `db.Exec`. Use `?` / `$1` placeholders.
- `[C]` `exec.Command("sh", "-c", userInput)` or similar. Pass arguments directly.
- `[C]` `tls.Config{InsecureSkipVerify: true}` outside tests.
- `[H]` `text/template` used to render HTML. Use `html/template`. Also flag `template.HTML(userInput)`.
- `[H]` `math/rand` used for tokens or keys. Use `crypto/rand`.
- `[H]` Path traversal: `filepath.Join(base, userInput)` without `filepath.IsLocal` or `os.Root` (1.24+); zip slip when extracting archives.
- `[H]` Unbounded request bodies (no `http.MaxBytesReader`); `http.ListenAndServe` with no read or write timeouts (slowloris).

## Errors
- `[H]` Errors ignored: `_ = f()`, or an unchecked `err` on calls that can fail. Also flag `rows.Err()` not checked after iterating.
- `[H]` `defer resp.Body.Close()` placed **before** the `err` check (panics on nil), or the body never closed, which leaks connections.
- `[M]` Errors wrapped without `%w`, so callers' `errors.Is` / `errors.As` checks fail; comparing errors with `==` instead of `errors.Is`.
- `[M]` `err` shadowed with `:=` in an inner scope, so the outer error is never set; `panic` used for ordinary errors in library code.

## Concurrency
- `[H]` Data races: maps or slices shared between goroutines without a mutex. Writing to a map concurrently is a fatal runtime error.
- `[H]` Goroutine leaks: goroutines blocked forever on channels with no `ctx.Done()` case.
- `[H]` `wg.Add` called inside the goroutine instead of before it starts.
- `[H]` Writing to a nil map (panics); sending on a closed channel (panics).
- `[H]` `http.Client{}` or `http.DefaultClient` with **no timeout** used for outbound calls.
- `[M]` `sync.Mutex` copied by value (`go vet` copylocks); `context.Context` not passed through, or stored in a struct; `context.Background()` used inside request handlers.
- `[M]` Loop variables captured by goroutines or closures when `go.mod` is older than 1.22; `defer` inside a loop holding resources until the function returns.

## Correctness
- `[H]` Integer conversions that overflow (`int64` to `int32`) on external data; slicing past the end without a bounds check.
- `[M]` Nil interface vs. typed nil: returning a typed nil pointer as an `error` gives a non-nil error.
- `[M]` `time.Now()` compared across machines for ordering; `time.Duration` arithmetic with untyped integers (`time.Sleep(5)` is 5 nanoseconds).

## Maintainability
- `[L]` Exported identifiers without doc comments; stuttering names (`user.UserService`); unnecessary `else` after `return`; overly large interfaces (accept interfaces, return structs).
