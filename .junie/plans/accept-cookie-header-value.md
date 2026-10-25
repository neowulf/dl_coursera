---
sessionId: session-261022-143819-18hl
---

# Technical Design

### Current Implementation

- **`dl_coursera_run.py`**: CLI entry point. `--cookies` (required) accepts a file path to a Netscape cookies file. The `crawl()` function takes `cookies_file` and passes it to `login()`.
- **`dl_coursera/Crawler.py`**: `login(sess, cookies_file=None)` loads cookies either from a Netscape file (`cookies_file`) or from `DL_COURSERA_COOKIES_BASE64` env var (base64-encoded Netscape file). Uses `MozillaCookieJar` to parse the Netscape format.
- **`cookie-to-netscape.sh`**: Shell script that converts a raw `Cookie` header string to Netscape format. Already exists but requires an extra step.

### Proposed Changes

1. **`dl_coursera/Crawler.py` — `login()` function**:
   - Add optional parameter `cookie_string=None`.
   - When `cookie_string` is provided, parse the semicolon-separated `name=value` pairs and populate `sess.cookies` directly (using `requests.cookies.RequestsCookieJar`).
   - Strip optional `Cookie: ` prefix if present (matching the shell script behavior).
   - Priority: `cookie_string` > `cookies_file` > `DL_COURSERA_COOKIES_BASE64` env var.
   - Also add support for `DL_COURSERA_COOKIE` env var as a fallback (raw cookie string, not base64).

2. **`dl_coursera/Crawler.py` — `Crawler.__init__()`**:
   - Add optional `cookie_string=None` parameter.
   - Pass to `login()` in the `login()` method.

3. **`dl_coursera_run.py` — `crawl()` function**:
   - Add optional `cookie_string=None` parameter.
   - Pass to `login()`.

4. **`dl_coursera_run.py` — CLI**:
   - Make `--cookies` optional (not required).
   - Add new optional `--cookie` argument for a raw cookie string value.
   - At least one of `--cookies` or `--cookie` must be provided (validation).

### Data Flow

```
graph LR
  A[CLI --cookie value] --> B[crawl()]
  C[CLI --cookies file] --> B
  B --> D[login()]
  D --> E[Parse cookie string / load Netscape file]
  E --> F[session.cookies]
```

### Key Signatures

```python

# Crawler.py

def login(sess, cookies_file=None, cookie_string=None):
    ...

class Crawler:
    def __init__(self, *, ts, sess, cookies_file=None, cookie_string=None, include_quiz=False):
        ...

# dl_coursera_run.py

def crawl(cookies_file=None, cookie_string=None, slug, outdir, is_spec, include_quiz=False):
    ...
```

# Delivery Steps

### ✓ Step 1: Update login() to accept raw cookie strings
Modify `login()` in `dl_coursera/Crawler.py` to accept a `cookie_string` parameter.
- Add `cookie_string=None` parameter to `login()`.
- When `cookie_string` is provided, parse semicolon-separated `name=value` pairs into `sess.cookies` using `RequestsCookieJar`.
- Strip optional `Cookie: ` prefix from the string.
- Add support for `DL_COURSERA_COOKIE` env var as a fallback for raw cookie strings.
- Update `Crawler.__init__()` to accept and store `cookie_string`.
- Update `Crawler.login()` method to pass `cookie_string` to the `login()` function.

### ✓ Step 2: Update crawl() and CLI to support --cookie flag
Modify `dl_coursera_run.py` to expose the new cookie string option.
- Add `cookie_string=None` parameter to `crawl()`.
- Pass `cookie_string` to `login()` inside `crawl()`.
- Pass `cookie_string` to `Crawler()` constructor.
- Make `--cookies` optional in the CLI argument parser.
- Add new `--cookie` argument for a raw cookie header value.
- Add validation that at least one of `--cookies` or `--cookie` is provided.
- Update the `main()` function to pass the new argument through the call chain.