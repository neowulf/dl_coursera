---
sessionId: session-261022-144400-1h5d
---

# Requirements

### Python Libraries for `dl_coursera`

Based on the project's `pyproject.toml`, here are the pip packages needed:

**Core dependencies (required to run the tool):**

| Package | Minimum Version | Purpose |
|---|---|---|
| `beautifulsoup4` | >= 4.13.4 | HTML parsing for extracting course content |
| `Jinja2` | >= 3.1.6 | Template engine |
| `lxml` | >= 6.0.0 | XML/HTML parser (used by beautifulsoup4) |
| `requests` | >= 2.32.4 | HTTP client for downloading content |
| `tqdm` | >= 4.67.1 | Progress bar for downloads |

**Quick install:**
```
pip install beautifulsoup4 Jinja2 lxml requests tqdm
```

Or simply:
```
pip install -e .
```
This installs the package itself along with all core dependencies.

**Optional dev dependencies:**
```
pip install pre-commit black
```

**Optional deploy dependencies:**
```
pip install build cryptography Nuitka setuptools twine wheel
```

**Python version requirement:** Python >= 3.10

The `requirements.txt` file also pins transitive dependencies (like `certifi`, `urllib3`, `charset-normalizer`, `idna`, `soupsieve`, `MarkupSafe`, `typing_extensions`), but these are installed automatically as sub-dependencies — you only need to install the core packages listed above.

# Delivery Steps

###   Step 1: Review project dependencies
Identify all Python pip packages needed for the dl_coursera project by examining pyproject.toml and requirements.txt.