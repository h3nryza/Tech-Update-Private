# Rust — CLAUDE.md Template

> Copy this entire file into `.claude/CLAUDE.md` at your project root.
> Replace all `<!-- CUSTOMIZE -->` sections with your project-specific values.

---

```markdown
# CLAUDE.md

<!-- CUSTOMIZE: Replace with your project name and description -->
## Project: My Tool
A Rust CLI application / library / web service.

---

## Build & Run Commands

```bash
# Build
cargo build                       # Debug build
cargo build --release             # Release build (optimized)
cargo build --all-targets         # Build everything including tests and benches

# Run
cargo run                         # Run default binary
cargo run -- --help               # Run with arguments
cargo run --bin my-tool           # Run specific binary in workspace
cargo run --example demo          # Run an example

# Test
cargo test                        # Run all tests
cargo test -- --nocapture         # Show println! output
cargo test test_parse             # Run tests matching "test_parse"
cargo test -p my-crate            # Test specific crate in workspace
cargo test --lib                  # Only unit tests
cargo test --doc                  # Only doc tests
cargo test --test integration     # Only integration tests

# Lint & Format
cargo clippy -- -D warnings       # Clippy with warnings as errors
cargo clippy --fix                # Auto-fix clippy suggestions
cargo fmt                         # Format all code
cargo fmt -- --check              # Check formatting without changing files

# Audit & Check
cargo check                       # Type check (faster than build)
cargo audit                       # Check for known vulnerabilities
cargo deny check                  # Check dependency licenses and advisories
cargo outdated                    # Show outdated dependencies

# Documentation
cargo doc --open                  # Build and open docs
cargo doc --no-deps               # Build docs without dependencies

# Benchmarks
cargo bench                       # Run benchmarks (criterion)
cargo bench -- "parse"            # Run benchmarks matching "parse"
```

---

## Code Conventions

### Rust Style
- Follow the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- **No `unwrap()` in library code** — use proper error handling with `Result` and `?`
- `unwrap()` is acceptable in tests and examples only
- Use `#[must_use]` on functions that return values that should not be ignored
- Prefer iterators and combinators over manual loops
- Use `derive` macros generously: `Debug`, `Clone`, `PartialEq`, `Eq`, `Hash`

### Error Handling
```rust
// Define domain errors with thiserror
use thiserror::Error;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("user not found: {0}")]
    NotFound(String),

    #[error("invalid input: {0}")]
    Validation(String),

    #[error("database error")]
    Database(#[from] sqlx::Error),

    #[error("internal error")]
    Internal(#[from] anyhow::Error),
}

// Use Result type alias for ergonomics
pub type Result<T> = std::result::Result<T, AppError>;
```

### Naming
- Crates/modules: `snake_case`
- Types/traits: `PascalCase`
- Functions/methods: `snake_case`
- Constants/statics: `UPPER_SNAKE_CASE`
- Lifetimes: short, descriptive (`'a`, `'conn`, `'src`)
- Type parameters: single uppercase (`T`, `E`) or short descriptive (`Item`, `Key`)
- Builder methods: `with_*` pattern
- Fallible constructors: `new` returns `Self`, `try_new` returns `Result<Self>`
- Conversion: `from_*`, `to_*`, `into_*`, `as_*` following std conventions

```rust
// GOOD: Idiomatic builder pattern
pub struct ServerConfig {
    host: String,
    port: u16,
    max_connections: usize,
}

impl ServerConfig {
    pub fn new(host: impl Into<String>, port: u16) -> Self {
        Self {
            host: host.into(),
            port,
            max_connections: 100,
        }
    }

    #[must_use]
    pub fn with_max_connections(mut self, max: usize) -> Self {
        self.max_connections = max;
        self
    }
}
```

### Module Organization
```rust
// lib.rs — public API surface
pub mod config;
pub mod error;
pub mod models;
pub mod handlers;

// Re-export important types at the crate root
pub use config::Config;
pub use error::{AppError, Result};
```

---

## Project Structure

<!-- CUSTOMIZE: Choose the layout that matches your project type -->

### Binary Project
```
src/
  main.rs                      # Entry point, CLI parsing, setup
  lib.rs                       # Library root (enables unit testing without binary)
  config.rs                    # Configuration loading
  error.rs                     # Error types
  cli.rs                       # CLI argument definitions (clap)
  commands/
    mod.rs
    run.rs                     # Subcommand implementations
    init.rs
  core/
    mod.rs
    parser.rs                  # Core logic
    transformer.rs
tests/
  integration_test.rs          # Integration tests
  fixtures/                    # Test data files
benches/
  benchmark.rs                 # Criterion benchmarks
examples/
  basic.rs                     # Usage examples
Cargo.toml
```

### Library Crate
```
src/
  lib.rs                       # Public API, module declarations, doc comments
  error.rs
  types.rs                     # Public types
  parser/
    mod.rs                     # Parser module
    lexer.rs
    ast.rs
  internal/                    # Implementation details (pub(crate))
    mod.rs
    helpers.rs
tests/
  integration_test.rs
benches/
  benchmark.rs
Cargo.toml
```

### Workspace (Multiple Crates)
```
Cargo.toml                     # [workspace] definition
crates/
  my-core/                     # Core library
    src/lib.rs
    Cargo.toml
  my-cli/                      # CLI binary
    src/main.rs
    Cargo.toml
  my-server/                   # Web server
    src/main.rs
    Cargo.toml
```

---

## Testing

### Unit Tests (In-Module)
```rust
// src/parser.rs
pub fn parse_config(input: &str) -> Result<Config> {
    // ...
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn parse_valid_config() {
        let input = r#"
            host = "localhost"
            port = 8080
        "#;
        let config = parse_config(input).unwrap();
        assert_eq!(config.host, "localhost");
        assert_eq!(config.port, 8080);
    }

    #[test]
    fn parse_empty_input_returns_error() {
        let result = parse_config("");
        assert!(result.is_err());
        assert!(result.unwrap_err().to_string().contains("empty input"));
    }

    #[test]
    fn parse_config_with_all_fields() {
        // Test with every field populated
        // ...
    }
}
```

### Integration Tests
```rust
// tests/integration_test.rs
use my_crate::Config;

#[test]
fn full_pipeline_produces_expected_output() {
    let input = std::fs::read_to_string("tests/fixtures/sample.txt").unwrap();
    let result = my_crate::process(&input).unwrap();
    assert_eq!(result.items.len(), 3);
}
```

### Property-Based Testing (proptest)
```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn parse_roundtrip(s in "[a-zA-Z0-9_]{1,64}") {
        let parsed = parse_identifier(&s).unwrap();
        assert_eq!(parsed.to_string(), s);
    }

    #[test]
    fn port_always_valid(port in 1u16..=65535) {
        let config = Config::new("localhost", port);
        assert!(config.is_valid());
    }
}
```

### Benchmarks (Criterion)
```rust
// benches/benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn bench_parse(c: &mut Criterion) {
    let input = include_str!("../tests/fixtures/large_input.txt");
    c.bench_function("parse_large_input", |b| {
        b.iter(|| my_crate::parse(black_box(input)))
    });
}

criterion_group!(benches, bench_parse);
criterion_main!(benches);
```

### Test Requirements
- All public functions must have at least one test
- Error paths must be tested (not just happy paths)
- Doc examples must compile and pass (`cargo test --doc`)
- No `#[ignore]` tests without a comment explaining why

---

## Cargo.toml Conventions

```toml
[package]
name = "my-tool"
version = "0.1.0"
edition = "2021"
rust-version = "1.75"        # Minimum supported Rust version
description = "A brief description"
license = "MIT OR Apache-2.0"

# Optimize dependencies in dev builds for faster iteration
[profile.dev.package."*"]
opt-level = 2

[profile.release]
lto = true                   # Link-time optimization
strip = true                 # Strip debug symbols
codegen-units = 1            # Single codegen unit for max optimization

[dependencies]
# Pin major versions, allow patch updates
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
clap = { version = "4", features = ["derive"] }
thiserror = "2"
anyhow = "1"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }

[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }
proptest = "1"
tempfile = "3"
assert_cmd = "2"             # For CLI integration tests
predicates = "3"             # For assertion matchers
```

---

## Clippy Configuration

```toml
# In Cargo.toml or .clippy.toml
# Cargo.toml approach:
[lints.clippy]
# Deny dangerous patterns
unwrap_used = "deny"          # In lib code; allow in tests via #[cfg(test)]
expect_used = "warn"
panic = "deny"
# Encourage good practices
missing_docs_in_private_items = "warn"
needless_pass_by_value = "warn"
redundant_closure_for_method_calls = "warn"
```

---

## Common Tasks

### Add a new subcommand (clap)
1. Add variant to the `Commands` enum in `cli.rs`
2. Create handler in `commands/your_command.rs`
3. Wire up in `main.rs` match
4. Add integration test in `tests/`
5. Add example in `examples/`

### Add a new dependency
1. `cargo add <crate>` (or `cargo add --dev <crate>` for dev deps)
2. Run `cargo deny check` to verify license compatibility
3. Run `cargo audit` to check for vulnerabilities
4. Run `cargo test` to verify nothing breaks

### Add async support (tokio)
1. Add `tokio = { version = "1", features = ["full"] }` to dependencies
2. Change `fn main()` to `#[tokio::main] async fn main()`
3. Use `tokio::spawn` for concurrent tasks
4. Use `tokio::select!` for racing futures

### Publish a crate
1. `cargo publish --dry-run` to verify
2. Ensure all doc tests pass
3. Update `CHANGELOG.md`
4. Tag the release: `git tag v0.1.0`
5. `cargo publish`
```
