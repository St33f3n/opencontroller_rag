# Rust Binary Crate Documentation Prompt

## Core Philosophy
Generate **English-only** documentation for Rust binary crates with appropriate depth levels - enabling 6+ month project re-entry with full context understanding.

**⚠️ CRITICAL: ALL documentation MUST be written in English, regardless of input language.**

## Documentation Depth Levels

### Level 1: Architecture & Key Components (Full "Why" Documentation)
- Crate-level documentation
- Module-level documentation  
- Key business logic functions
- Complex algorithms
- External integration points

### Level 2: Standard Functions (Moderate Documentation)
- Regular business functions
- Data processing functions
- Most async functions

### Level 3: Utility & Helper Functions (Minimal Documentation)
- Simple transformations
- Getters/setters
- Basic validation functions
- Internal helpers

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

### Function-Level Documentation (by Level)

#### Level 1: Key Functions (Full Documentation)
```rust
/// Processes incoming user commands and routes them to appropriate handlers.
///
/// This is the main command dispatcher that validates, normalizes, and routes
/// all user input. It sits between the CLI parser and business logic handlers.
///
/// ## Design Rationale
/// Uses async dispatch pattern to handle I/O-heavy operations without blocking.
/// Command validation happens early to fail fast on invalid inputs.
///
/// ## Error Handling
/// Commands that fail validation are logged but don't crash the application.
/// Network-related commands use retry logic for resilience.
///
/// # Errors
/// 
/// Returns [`CommandError`] when:
/// - Invalid command format - logs and continues processing
/// - Network timeout - retries up to 3 times before failing
/// - Permission denied - stops processing and requires user intervention
///
async fn dispatch_user_command(cmd: Command) -> Result<CommandResult, CommandError> {
    // implementation
}
```

#### Level 2: Standard Functions (Moderate Documentation)
```rust
/// Validates user input according to application rules.
///
/// Checks format, length limits, and required fields. Used by all input
/// processing functions before data reaches business logic.
///
/// # Errors
/// 
/// Returns [`ValidationError`] for invalid input format or missing required fields.
///
fn validate_user_input(input: &UserInput) -> Result<(), ValidationError> {
    // implementation
}
```

#### Level 3: Utility Functions (Minimal Documentation)
```rust
/// Converts string to lowercase and trims whitespace.
fn normalize_string(input: &str) -> String {
    // implementation
}

/// Returns true if the configuration is in debug mode.
fn is_debug_mode() -> bool {
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

❌ **Don't over-document simple utilities**:
```rust
/// Converts the input string to uppercase and removes all whitespace characters
/// because we need standardized formatting for comparison operations throughout
/// the application and this ensures consistent data processing across modules...
fn clean_string(input: &str) -> String  // TOO MUCH for a simple utility
```

❌ **Don't write in German** (even if input is German):
```rust
/// Verarbeitet Benutzereingaben und validiert sie  // WRONG - must be English
fn process_input() // CORRECT: document in English
```

❌ **Don't just repeat the type signature**:
```rust
/// Takes a string and returns a Result<String, Error>
fn process_string(input: String) -> Result<String, Error>
```

✅ **Do match documentation level to function importance**:
```rust
// Level 3: Simple and sufficient for utility
/// Converts text to title case.
fn to_title_case(text: &str) -> String

// Level 1: Detailed for key business logic
/// Processes payment transactions through the configured payment gateway.
///
/// This function handles the complete payment lifecycle including validation,
/// gateway communication, and result persistence. It's designed to be resilient
/// against network failures and provides detailed error context for debugging.
async fn process_payment(payment: PaymentRequest) -> Result<PaymentResult, PaymentError>
```

## Documentation Level Decision Guide

**Use Level 1 (Full) when:**
- Function is called from main business logic
- Complex error handling or async behavior
- Integration with external systems
- Performance-critical operations
- Would take >30 minutes to understand without docs

**Use Level 2 (Moderate) when:**
- Standard business functions
- Clear purpose but some complexity
- Error handling that affects user experience
- Called by multiple other functions

**Use Level 3 (Minimal) when:**
- Simple transformations or utilities
- Self-explanatory from name and signature
- Internal helper functions
- Getters, setters, simple validators

## Final Notes

**Language Requirement**: ALL documentation must be written in English, regardless of the language of input or comments in the code.

**Documentation Depth**: Match the documentation depth to function importance. Not every utility function needs a design rationale - save the "why" explanations for functions that actually need them.

**Target Audience**: You're writing for **future you** who forgot the context. For Level 1 functions, explain the reasoning. For Level 3 functions, just make it clear what they do.
