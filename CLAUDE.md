# CLAUDE.md - Memic Python SDK

Guidelines for AI assistants working on this codebase.

## Golden Rules

1. **NO BREAKING CHANGES** - Never break existing functionality
2. **ASK FIRST** - Always ask the developer before making changes
3. **VERSION OVER BREAKING** - Add new versions/methods instead of modifying existing ones
4. **KISS** - Keep it simple, stupid

## Package Overview

This is the official Python SDK for the Memic Context Engineering API. It provides:
- File uploads with presigned URLs
- Semantic search with metadata filters
- Simple, KISS-focused design

## Project Structure

```
memic-python/
├── src/memic/
│   ├── __init__.py      # Public exports
│   ├── _version.py      # Version string
│   ├── client.py        # Main Memic class
│   ├── types.py         # Pydantic models
│   └── exceptions.py    # Exception classes
└── tests/
    └── test_client.py   # Unit tests
```

## No Breaking Changes Policy

**NEVER do any of the following:**
- Remove or rename public functions, classes, or methods
- Change function signatures (removing params, changing required/optional)
- Change return types
- Change exception types thrown by a function
- Remove or rename dataclass/Pydantic model fields
- Change default values that alter behavior

**Instead, always:**
- Add new optional parameters with defaults
- Add new methods alongside existing ones (e.g., `search_v2()`)
- Add new fields, never remove
- Deprecate with warnings, remove only after 2+ minor versions

### Deprecation Process

```python
import warnings
warnings.warn(
    "old_method() is deprecated, use new_method() instead. "
    "Will be removed in v0.3.0",
    DeprecationWarning,
    stacklevel=2
)
```

## Design Principles (KISS)

- Single `Memic` class, no nested resource patterns
- Use `requests` for HTTP (sync only in V0.x)
- Pydantic models for types (runtime validation, IDE support)
- 4 exception classes only (MemicError, AuthenticationError, NotFoundError, APIError)
- Env var fallback for API key (MEMIC_API_KEY)

## Testing

```bash
# Run tests
pytest tests/

# Run with coverage
pytest --cov=memic tests/

# Type check
mypy src/memic/
```

## Common Tasks

### Adding a New API Method

1. Add method to `client.py` in the `Memic` class
2. Add any new types to `types.py`
3. Add tests to `test_client.py`
4. Update `__init__.py` if new types need exporting
5. Update CHANGELOG.md

## Build & Deploy

### Local Development

```bash
# Install in editable mode with dev dependencies
pip install -e ".[dev]"

# Run tests
pytest tests/

# Run linter
ruff check src/

# Run type checker
mypy src/memic/
```

### Build Package

```bash
# Clean previous builds
rm -rf dist/ build/ *.egg-info

# Build sdist and wheel
python -m build

# Verify the build
ls dist/
# Should show: memic-X.X.X.tar.gz and memic-X.X.X-py3-none-any.whl
```

### Publish to PyPI

**Pre-release checklist:**
1. All tests pass: `pytest tests/`
2. Type check passes: `mypy src/memic/`
3. Lint passes: `ruff check src/`
4. Version bumped in `src/memic/_version.py`
5. CHANGELOG.md updated

**Publish steps:**

```bash
# 1. Build fresh
rm -rf dist/ && python -m build

# 2. Upload to TestPyPI first (optional but recommended)
twine upload --repository testpypi dist/*

# 3. Test install from TestPyPI
pip install --index-url https://test.pypi.org/simple/ memic

# 4. Upload to production PyPI
twine upload dist/*

# 5. Tag the release
git tag v0.X.X
git push origin v0.X.X
```

**Required credentials:**
- PyPI API token in `~/.pypirc` or via `TWINE_USERNAME`/`TWINE_PASSWORD` env vars
