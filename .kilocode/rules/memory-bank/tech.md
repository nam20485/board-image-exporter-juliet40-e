# Tech

## Technologies Used

### Core Stack
- **Python 3.11+**: Implementation language with modern features (pattern matching, improved error messages, performance gains)
- **Pydantic v2**: Data modeling, validation, and serialization with JSON schema generation
- **Matplotlib 3.7+**: Rendering engine supporting SVG, PNG, and PDF via single `savefig()` API
- **NumPy 1.24+**: Optional acceleration for geometric transforms and batch operations

### Package Management
- **uv**: Fast Python package manager with lockfile support for reproducible builds

### Development Tools
- **pytest 7.4+**: Unit testing framework
- **pytest-cov**: Coverage reporting
- **hypothesis 6.82+**: Property-based testing
- **syrupy 4.0+**: Snapshot testing for visual regression
- **ruff 0.1+**: Linting and formatting
- **pyright 1.1+**: Static type checking (strict mode)

### Containerization
- **Docker**: Multi-stage builds with python:3.11-slim base image
- **Docker Compose**: Service orchestration with volume mounts

### Security
- **TruffleHog**: Secret scanning in CI and pre-commit hooks

## Development Setup

### Prerequisites
- Python 3.11 or higher
- uv package manager (`pip install uv`)
- Docker (optional, for containerized execution)

### Environment Setup
```bash
# Clone repository
git clone <repository-url>
cd pcb-renderer

# Install dependencies
uv sync

# Run tests
uv run pytest

# Run type checking
uv run pyright

# Run linting
uv run ruff check .
```

### Docker Usage
```bash
# Build image
docker build -t pcb-renderer:latest -f docker/Dockerfile .

# Run CLI
docker run --rm -v $(pwd):/workdir pcb-renderer:latest validate /workdir/board.json
```

## Technical Constraints

### 20-Minute Review Constraint
- Codebase must be reviewable within 20 minutes
- Clear separation of concerns
- Explicit over implicit
- Comprehensive docstrings

### Correctness Requirements
- All geometric transforms must be mathematically correct
- Validation must detect all 14 error classes
- Rendering must be deterministic (same input → same output)
- No external service dependencies (fully offline)

### Performance Considerations
- Pydantic v2 for fast validation
- Optional NumPy acceleration
- Matplotlib's optimized rendering
- Lazy loading where appropriate

## Dependencies

### Production Dependencies
```toml
[project]
dependencies = [
    "pydantic>=2.0.0",
    "matplotlib>=3.7.0",
    "numpy>=1.24.0",
]
```

### Development Dependencies
```toml
[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "hypothesis>=6.82.0",
    "syrupy>=4.0.0",
    "ruff>=0.1.0",
    "pyright>=1.1.0",
]
```

## Tool Usage Patterns

### Running Tests
```bash
# All tests
uv run pytest

# With coverage
uv run pytest --cov=pcb_renderer

# Specific test file
uv run pytest tests/test_parse.py

# Property-based tests only
uv run pytest -m hypothesis

# Snapshot tests
uv run pytest --snapshot-update
```

### Code Quality
```bash
# Format code
uv run ruff format .

# Check linting
uv run ruff check .

# Type checking
uv run pyright

# All checks
uv run ruff check . && uv run pyright && uv run pytest
```

### CLI Usage
```bash
# Install in development mode
uv pip install -e .

# Run CLI commands
pcb-render validate board.json
pcb-render render board.json -o output.svg
pcb-render info board.json
```

### Security Scanning
```bash
# Run TruffleHog locally
./scripts/security/run-trufflehog.sh
# or
./scripts/security/run-trufflehog.ps1
```

## Configuration Files

### pyproject.toml
- Project metadata
- Dependencies
- Tool configurations (ruff, pyright, pytest)
- Entry points

### uv.lock
- Reproducible dependency resolution
- Commit to version control

### .github/workflows/
- CI/CD pipelines
- Secret scanning
- Automated testing

### docker/
- Dockerfile for containerized execution
- docker-compose.yml for service orchestration
- .env.example for environment configuration
