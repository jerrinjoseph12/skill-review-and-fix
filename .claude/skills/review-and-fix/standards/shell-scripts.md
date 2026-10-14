# Shell Script Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers Bash, POSIX sh and Zsh scripts (`*.sh`, `*.bash`, `*.zsh`, or files with a shell shebang), plus PowerShell (`*.ps1`, `*.psm1`). Inline scripts in CI pipelines also get [ci-cd.md](ci-cd.md).

## Authorities
- ShellCheck wiki (rule `SCxxxx` explanations)
- Google Shell Style Guide
- POSIX Shell Command Language
- PowerShell: PSScriptAnalyzer rules and Microsoft's PowerShell security guidance

## Tooling to look for and run
- `shellcheck -x script.sh`
- `shfmt -d`
- `bats` tests (if present)
- PowerShell: `Invoke-ScriptAnalyzer -Path . -Recurse`, Pester

## Security and safety
- `[C]` `rm -rf "$DIR/"*` or `rm -rf $DIR/...` where `DIR` may be empty or unset. This can delete `/`. Use `"${DIR:?}"` and `set -u`.
- `[C]` `eval` on user, file or network input; `curl ... | bash` without checksum verification.
- `[C]` PowerShell `Invoke-Expression` (`iex`) on external input.
- `[H]` Unquoted variables and command substitutions (`$var`, `$(cmd)`), which cause word splitting and globbing. Quote them: `"$var"`.
- `[H]` Secrets passed as command-line arguments, which are visible in `ps` and shell history; `set -x` enabled around secret handling.
- `[H]` Fixed temp file names in `/tmp`, which allow symlink attacks. Use `mktemp`, and clean up with `trap`.
- `[H]` PowerShell `-ExecutionPolicy Bypass` baked into scripts; plain-text passwords (`ConvertTo-SecureString -AsPlainText` with literals in the script).

## Correctness
- `[H]` No `set -euo pipefail` (or the project's equivalent) in scripts whose later steps rely on earlier ones succeeding; `cd dir` without `|| exit`. PowerShell: no `$ErrorActionPreference = 'Stop'` where failures must stop the script.
- `[H]` Bash-only syntax (`[[ ]]`, arrays, `==`, `function`, `source`) in scripts with a `#!/bin/sh` shebang. They break on dash or busybox.
- `[H]` Parsing `ls` output, or `for f in $(find ...)`, which breaks on spaces and newlines. Use globs, or `find -print0 | xargs -0` / `while IFS= read -r -d ''`.
- `[M]` `read` without `-r` (backslashes get mangled); `$*` instead of `"$@"`; `[ $x = y ]` with `x` unquoted and possibly empty.
- `[M]` Exit codes ignored in pipelines without `pipefail`; `local var=$(cmd)` hiding the command's exit status. Declare and assign separately.
- `[M]` GNU-only flags (`sed -i ''` vs `sed -i`, `date -d`, `readlink -f`) in scripts that must run on macOS too, or the other way round.

## Maintainability
- `[L]` No usage or help text for scripts that take arguments; magic paths that should be variables; `cat file | grep` (useless use of cat); functions not in `snake_case`; no `main` function in long scripts.
