# Python Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

## Authorities
- PEP 8 (style), PEP 257 (docstrings), PEP 484 and later (typing)
- The "Security Considerations" sections of the Python standard library docs
- Bandit rules and OWASP cheat sheets
- Framework docs: the Django deployment checklist, Flask, FastAPI

## Tooling to look for and run
- `pytest`
- `ruff check` or `flake8`
- `ruff format --check` or `black --check`
- `mypy` or `pyright`
- `bandit -r`
- `pip-audit`

Use whatever `pyproject.toml`, `setup.cfg`, `tox.ini` or `Makefile` configures.

## Version awareness
Read `requires-python`. Don't suggest syntax newer than the target: `match` needs 3.10, `X | Y` type unions need 3.10, and `typing.Self` needs 3.11.

## Security
- `[C]` `eval` or `exec` on external data; `pickle`, `marshal`, `shelve` or `joblib.load` on untrusted bytes; `yaml.load` without `SafeLoader` (use `yaml.safe_load`).
- `[C]` `subprocess` with `shell=True` and interpolated input; `os.system` or `os.popen` with input. Pass an argument list.
- `[C]` SQL via f-strings, `%` or `.format()` in `cursor.execute`. Use parameter placeholders. Also flag Django `raw()` or `extra()` with interpolation.
- `[C]` `app.run(debug=True)` in Flask or Werkzeug reachable in production (the debugger allows remote code execution); Django `DEBUG = True` or a committed `SECRET_KEY`.
- `[H]` `assert` used for authorization or validation. Asserts are removed with `python -O`.
- `[H]` `requests` or `httpx` with `verify=False`, or **without `timeout`**, which hangs workers forever.
- `[H]` `random` used for tokens or passwords. Use `secrets`.
- `[H]` `tarfile.extractall` or `zipfile` without path checks (zip slip). Use `filter="data"` (3.12+, the default from 3.14).
- `[H]` `xml.etree`, `minidom` or `lxml` on untrusted XML. Use `defusedxml` (see [xml.md](xml.md)).
- `[H]` Django `mark_safe` or `|safe`, or Jinja `Markup` or `autoescape=False`, on user data.
- `[M]` `tempfile.mktemp` (race condition); world-writable `os.chmod`; secrets read from code instead of the environment or a vault.

## Correctness
- `[H]` Mutable default arguments (`def f(x=[])`).
- `[H]` Late-binding closures in loops (`lambda: i` captures the last `i`).
- `[H]` Bare `except:` or `except Exception: pass` that hides errors on business paths. Also catching `BaseException` by accident, which swallows `KeyboardInterrupt` and `SystemExit`.
- `[H]` `is` or `is not` used to compare values (`x is 1000`, `s is "abc"`) instead of `None` or sentinels.
- `[H]` Naive and timezone-aware `datetime` objects mixed; `datetime.utcnow()` (deprecated in 3.12) where an aware datetime is needed. Use `datetime.now(timezone.utc)`.
- `[H]` Money in `float`. Use `decimal.Decimal` built from strings.
- `[H]` Modifying a list or dict while iterating over it; reusing an exhausted generator or iterator.
- `[M]` Files, sockets or locks opened without a `with` block; class-level mutable attributes shared between instances by mistake.

## Async
- `[H]` Blocking calls inside `async def` (`requests`, `time.sleep`, synchronous DB drivers), which stall the event loop.
- `[H]` A coroutine called without `await`, so it never runs; tasks created with `asyncio.create_task` and not referenced (they can be garbage-collected).
- `[M]` `asyncio.gather` over an unbounded list with no semaphore.

## Performance
- `[H]` Django or SQLAlchemy N+1 queries in loops or templates. Use `select_related`, `prefetch_related`, `joinedload` or `selectinload`.
- `[M]` pandas `iterrows` or `apply` where a vectorized operation exists; whole files or querysets loaded into memory where iteration or streaming would do; building strings with `+=` in large loops.

## Maintainability
- `[M]` Public functions without type hints in a codebase that uses typed APIs; new behavior without tests.
- `[L]` PEP 8 or naming deviations not caught by the configured linter, `print` used instead of `logging` in library code, star imports, dead code.
