# Rust Binary Crate Documentation Prompt

## Core Philosophy
Generate comprehensive documentation for Rust binary crates that prioritizes **"Why"** over **"What"** - enabling 6+ month project re-entry with full context understanding.

## Documentation Structure

### Crate-Level Documentation (`main.rs` or `lib.rs` top)
```rust
//! # [Application Name]
//! 
//! [One-line description of what this application does and why it exists]
//!
//! ## Architecture Overview
//! 
//! [High-level explanation of the overall design approach]
//! 
//! ## Key Design Decisions
//! 
//! - **[Decision 1]**: [Why this approach was chosen over alternatives]
//! - **[Decision 2]**: [Rationale and trade-offs considered]
//! 
//! ## Dependencies & Rationale
//! 
//! - **tokio**: [Why async runtime was needed, why tokio over alternatives]
//! - **anyhow**: [Error handling strategy explanation]
//! - **[other key crates]**: [Justification for inclusion]
//!
//! ## Running & Development
//! 
//! [Essential information for getting back into development]
```

### Module-Level Documentation
```rust
//! # [Module Purpose]
//!
//! ## Why This Module Exists
//! [Explanation of its role in the larger system architecture]
//!
//! ## Key Abstractions
//! [What problems these abstractions solve]
//!
//! ## Error Handling Strategy
//! [How errors are handled in this module and why]
//!
//! ## Async Patterns Used
//! [If applicable: async patterns and their reasoning]
```

### Function-Level Documentation

#### For Async Functions
```rust
/// [Brief description of what this function accomplishes]
///
/// ## Design Rationale
/// [Why this function exists and why it's implemented this way]
///
/// ## Async Behavior
/// [What async operations are performed and why they need to be async]
/// [Cancellation behavior, if relevant]
///
/// ## Error Handling
/// [What errors can occur and the recovery strategy]
///
/// # Errors
/// 
/// Returns [`ErrorType`] when:
/// - [Specific condition] - [Why this is an error and what to do]
/// - [Another condition] - [Recovery guidance]
///
/// # Examples
/// 
/// ```rust,no_run
/// // Only include if genuinely helpful for understanding
/// // Focus on the "why" rather than mechanical usage
/// ```
async fn example_function() -> Result<(), anyhow::Error> {
    // implementation
}
```

#### For Regular Functions
```rust
/// [Brief description focusing on purpose, not mechanics]
///
/// ## Why This Function Exists
/// [Its role in the larger flow/algorithm]
///
/// ## Implementation Choice
/// [Why this particular approach was chosen]
///
/// # Errors
/// [If fallible - what can go wrong and why]
///
/// # Panics
/// [If function can panic - when and why]
fn example_function() {
    // implementation
}
```

### Struct/Enum Documentation
```rust
/// [What this type represents in the domain]
///
/// ## Design Rationale
/// [Why this structure was chosen over alternatives]
/// [What invariants it maintains]
///
/// ## Usage Context
/// [Where and how this type fits into the application flow]
#[derive(Debug)]
struct ExampleStruct {
    /// [Why this field exists and what constraints it has]
    field: String,
}
```

## Special Sections for Binary Crates

### Configuration & Environment
```rust
/// ## Configuration
/// [How this component is configured and why those options exist]
///
/// ## Environment Dependencies
/// [What external dependencies this code assumes and why]
```

### Performance Considerations
```rust
/// ## Performance Notes
/// [Why certain performance trade-offs were made]
/// [What the bottlenecks are and why they're acceptable]
```

### Future Maintenance
```rust
/// ## Known Limitations
/// [What doesn't work perfectly and why it's OK for now]
///
/// ## Extension Points
/// [Where future features would naturally fit]
```

## Error Handling Documentation Patterns

### For Result-Returning Functions
```rust
/// # Errors
/// 
/// This function can fail in the following scenarios:
/// 
/// - **Network connectivity**: Returns [`NetworkError`] when unable to reach the service
///   - *Recovery*: Retry with exponential backoff
/// - **Authentication**: Returns [`AuthError`] when credentials are invalid
///   - *Recovery*: Prompt user for re-authentication
/// - **Data corruption**: Returns [`DataError`] when response format is unexpected
///   - *Recovery*: Log error and use cached data if available
```

### For Complex Error Chains
```rust
/// ## Error Propagation Strategy
/// 
/// This module uses [`anyhow`] for error handling because:
/// - [Reason 1: e.g., "Need rich context without defining custom error types"]
/// - [Reason 2: e.g., "Errors are primarily for logging, not programmatic handling"]
/// 
/// Errors bubble up through: [Module A] → [Module B] → [Main error handler]
```

## Async Documentation Patterns

### For Async Functions
```rust
/// ## Async Behavior
/// 
/// This function is async because it:
/// - [Specific reason: e.g., "Waits for network I/O"]
/// - [Performance reason: e.g., "Allows other tasks to run during file operations"]
/// 
/// **Cancellation**: [What happens if this future is dropped/cancelled]
/// **Concurrency**: [Whether this function is safe to call concurrently]
```

### For Async Contexts
```rust
/// ## Runtime Requirements
/// 
/// Requires tokio runtime because:
/// - [Specific feature needed: e.g., "async filesystem operations"]
/// - [Why tokio over alternatives: e.g., "Better ecosystem compatibility"]
```

## Cargo.toml Documentation
```toml
[package]
name = "your-app"
version = "0.1.0"
edition = "2021"
description = "Brief description for cargo doc"
authors = ["Your Name <your.email@example.com>"]

# Documentation generation
[package.metadata.docs.rs]
# Include docs for private items in binary crates
rustdoc-args = ["--document-private-items"]
```

## Usage Commands

### Generate Documentation
```bash
# Standard documentation generation
cargo doc --document-private-items

# Open in browser
cargo doc --document-private-items --open

# Include source code links
cargo doc --document-private-items --open --no-deps
```

### Documentation Linting
```rust
// Add to main.rs or lib.rs for enforcing documentation
#![warn(missing_docs)]
#![warn(rustdoc::missing_doc_code_examples)]
```

## Quality Checklist

For each documented item, ensure:

- [ ] **Why it exists** is explained (not just what it does)
- [ ] **Design rationale** is documented for non-obvious choices
- [ ] **Error conditions** are explained with recovery guidance
- [ ] **Async behavior** is justified and described
- [ ] **Dependencies** are explained (why this crate/approach)
- [ ] **Future maintainer** would understand the context
- [ ] **Configuration/environment** requirements are documented

## Anti-Patterns to Avoid

❌ **Don't document obvious things**:
```rust
/// Returns true if the value is true
fn is_true(value: bool) -> bool { value }
```

❌ **Don't just repeat the type signature**:
```rust
/// Takes a string and returns a Result<String, Error>
fn process_string(input: String) -> Result<String, Error>
```

✅ **Do explain the "why"**:
```rust
/// Normalizes user input by trimming whitespace and converting to lowercase.
/// 
/// This standardization is necessary because user inputs come from multiple
/// sources (CLI args, config files, environment variables) and need consistent
/// formatting for comparison operations.
fn normalize_user_input(input: String) -> Result<String, ValidationError>
```

## Final Note

Remember: You're writing for **future you** who forgot the context. Explain the reasoning, not just the mechanics.
