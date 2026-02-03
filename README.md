# Pipeline Project Template

A modern project template for VFX/Animation pipeline tools with:
- **Python** — DCC tools, APIs, CLIs
- **Rust** — Performance-critical core logic (with PyO3 bindings)
- **C++** — Maya/Houdini plugins
- **Infrastructure** — Terraform modules for AWS deployment
- **CI/CD** — GitHub Actions for all the things

## Quick Start

### Using Copier (recommended)

```bash
# Install copier
pip install copier

# Create new project from template
copier copy gh:voidreamer/pipeline-project-template my-new-project

# Or from local clone
copier copy ./pipeline-project-template my-new-project
```

### Manual Setup

1. Clone/fork this repo
2. Run the init script:
   ```bash
   ./scripts/init-project.sh my-project-name
   ```

## Project Structure

```
my-project/
├── python/                  # Python package
│   ├── src/
│   │   └── my_project/
│   ├── tests/
│   └── pyproject.toml
├── rust/                    # Rust crate(s)
│   ├── core/                # Core library
│   ├── cli/                 # CLI binary
│   └── pyo3/                # Python bindings
├── cpp/                     # C++ DCC plugins
│   ├── maya/
│   └── houdini/
├── terraform/               # Infrastructure
│   ├── modules/
│   └── environments/
├── .github/workflows/       # CI/CD
└── docs/                    # Documentation
```

## GitHub Actions Workflows

| Workflow | Trigger | Description |
|----------|---------|-------------|
| `python-ci.yml` | Push to python/ | Lint, typecheck, test |
| `rust-ci.yml` | Push to rust/ | Format, clippy, test, miri |
| `maya-plugin.yml` | Push to cpp/maya/ | Build for Maya 2022-2025 |
| `houdini-plugin.yml` | Push to cpp/houdini/ | Build for Houdini 19.5-20.5 |
| `release.yml` | Tag v* | Build & publish all artifacts |

### Maya Plugin Matrix

Builds automatically for:
- **Maya versions**: 2022, 2023, 2024, 2025
- **Platforms**: Linux, Windows

### Rust + Python (PyO3)

The template supports building Rust libraries with Python bindings:

```rust
// rust/pyo3/src/lib.rs
use pyo3::prelude::*;

#[pyfunction]
fn fast_validate(data: &str) -> PyResult<bool> {
    Ok(my_project_core::validate(data))
}

#[pymodule]
fn my_project_rs(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(fast_validate, m)?)?;
    Ok(())
}
```

## Terraform Modules

### lambda-api

Deploy a Python Lambda with Function URL:

```hcl
module "api" {
  source = "./modules/lambda-api"
  
  project_name    = "my-project"
  environment     = "prod"
  lambda_zip_path = "../dist/lambda.zip"
}
```

### s3-frontend + cloudfront

Deploy a static frontend with CDN:

```hcl
module "frontend" {
  source = "./modules/s3-frontend"
  
  project_name = "my-project"
  environment  = "prod"
}

module "cdn" {
  source = "./modules/cloudfront"
  
  project_name                   = "my-project"
  environment                    = "prod"
  s3_bucket_regional_domain_name = module.frontend.bucket_regional_domain_name
  s3_bucket_id                   = module.frontend.bucket_name
}
```

## Development

### Prerequisites

- Python 3.10+ (recommend using `uv`)
- Rust 1.75+ (install via `rustup`)
- CMake 3.20+ (for C++ builds)

### Setup

```bash
# Python
uv sync --dev

# Rust
cargo build

# Run all tests
uv run pytest
cargo test
```

## License

MIT
