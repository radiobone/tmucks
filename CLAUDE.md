# CLAUDE.md - AI Assistant Guide for tmucks

**Last Updated:** 2025-11-25
**Project:** tmucks - tmux configuration manager
**Language:** Rust
**Version:** 0.1.0

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Codebase Structure](#codebase-structure)
3. [Architecture and Design Patterns](#architecture-and-design-patterns)
4. [Development Workflow](#development-workflow)
5. [Code Conventions](#code-conventions)
6. [Common Tasks](#common-tasks)
7. [Testing Strategy](#testing-strategy)
8. [File Reference](#file-reference)
9. [Key Dependencies](#key-dependencies)
10. [Working with This Codebase](#working-with-this-codebase)

---

## Project Overview

### What is tmucks?

tmucks is a command-line terminal user interface (TUI) application that manages multiple tmux configurations. It allows users to:
- Save multiple tmux configurations
- Apply saved configurations to active tmux sessions
- Update existing configurations
- Delete configurations
- View all saved configurations

### Dual-Mode Operation

The application operates in two modes:

1. **CLI Mode:** Execute single commands and exit (e.g., `tmucks list`, `tmucks apply dev`)
2. **TUI Mode:** Interactive event-driven interface launched when no command is provided (`tmucks`)

### Key Features

- Configuration storage in `~/.config/tmucks/`
- Automatic config file extension normalization (`.conf`)
- Live tmux session reloading after applying configs
- Interactive TUI with vim-style keybindings
- Auto-dismissing status messages (5 seconds)
- Update confirmation dialog to prevent accidental overwrites

---

## Codebase Structure

```
/home/user/tmucks/
├── src/
│   ├── main.rs           (58 lines)   - Entry point and CLI command routing
│   ├── cli.rs            (31 lines)   - CLI argument definitions
│   ├── config.rs         (120 lines)  - Configuration file management
│   ├── app.rs            (161 lines)  - Application state management
│   └── tui.rs            (576 lines)  - Terminal UI rendering and events
├── Cargo.toml                         - Project manifest
├── Cargo.lock                         - Dependency lock
├── README.md                          - User-facing documentation
├── CRUSH.md                           - Development commands and style guide
├── CLAUDE.md                          - This file (AI assistant guide)
├── LICENSE                            - Apache 2.0
└── .gitignore                         - Git ignore rules
```

**Total Lines of Code:** 946 lines across 5 modules

---

## Architecture and Design Patterns

### Module Dependency Graph

```
main.rs (CLI Router)
  ↓
  ├→ cli.rs (Command Definitions)
  ├→ config.rs (File Operations) ←─────┐
  └→ tui.rs (UI Rendering) ←─ app.rs (State Management)
```

### Key Design Patterns

#### 1. Command Pattern (main.rs)
```rust
match cli.command {
    Some(Commands::List) => { /* handler */ }
    Some(Commands::Apply { name }) => { /* handler */ }
    // ... other commands
    None => tui::run()? // Launch TUI
}
```

#### 2. State Machine Pattern (app.rs)
```rust
pub enum InputMode {
    Normal,          // Navigation and shortcuts
    Saving,          // Text input for new config names
    UpdateConfirm,   // Confirmation dialog
}
```

Different input modes enable context-specific key handling without complex branching.

#### 3. Auto-Dismiss Status Messages
```rust
pub status_message_time: Option<Instant>

pub fn update_status_message(&mut self) {
    if let Some(message_time) = self.status_message_time {
        if message_time.elapsed() >= Duration::from_secs(5) {
            self.status_message = self.default_status_message.clone();
            self.status_message_time = None;
        }
    }
}
```

Non-blocking UI feedback that resets after 5 seconds.

#### 4. Event-Driven UI Updates (tui.rs)
```rust
event::poll(Duration::from_millis(100))?
```

Allows UI updates (status message timer) even without user input.

#### 5. Extension Normalization (cli.rs)
```rust
pub fn ensure_conf_extension(name: String) -> String {
    if name.ends_with(".conf") {
        name
    } else {
        format!("{}.conf", name)
    }
}
```

Users can specify `tmucks apply dev` or `tmucks apply dev.conf` - both work.

#### 6. Consistent Error Handling
- All fallible functions return `Result<(), Box<dyn std::error::Error>>`
- Use `?` operator for error propagation
- Descriptive error messages with context (e.g., "Config 'dev' does not exist. Use 'save' command...")

---

## Development Workflow

### Git Branch Strategy

- **Main Branch:** `main` (or master)
- **Feature Branches:** PR-based workflow
- **Current Branch:** `claude/claude-md-miemjesjswyrdwcv-01U4PK4rPbNhhMBb2BCyA9Yn`

### Build Commands

```bash
# Build the project
cargo build

# Run the application
cargo run

# Run with CLI args
cargo run -- --help
cargo run -- list
cargo run -- apply dev

# Run tests
cargo test

# Lint with Clippy
cargo clippy

# Format code
cargo fmt
```

### Commit Message Style

Based on recent commits:
- Lowercase imperative mood (e.g., "add more comprehensive report messages")
- PR merges reference issue numbers (e.g., "Merge pull request #4 from...")
- Feature commits use `feat:` prefix occasionally (e.g., "feat: enhance tmucks...")

### Recent Development Activity

Recent features added:
- Notification bar auto-dismiss after 5 seconds (PR #4)
- Event polling for updates (PR #3)
- Vim keybindings (PR #2)
- Comprehensive improvements (d732374)

---

## Code Conventions

### Naming Conventions

- **Variables/Functions:** `snake_case`
- **Types/Structs:** `PascalCase`
- **Constants:** `SCREAMING_SNAKE_CASE`
- **Modules:** `snake_case`

### Import Organization

```rust
// 1. Standard library imports
use std::{fs, path::PathBuf};

// 2. External crate imports
use clap::Parser;
use ratatui::widgets::ListState;

// 3. Local module imports
use crate::config::ConfigManager;
```

### Error Handling Rules

1. **Always use `Result<(), Box<dyn std::error::Error>>`** for fallible functions
2. **Use `?` operator** for error propagation
3. **Return descriptive errors** with context:
   ```rust
   return Err(format!("Config file not found: {}", config_name).into());
   ```
4. **Check existence before operations:**
   ```rust
   if !source_path.exists() {
       return Err(format!("Config file not found: {}", config_name).into());
   }
   ```

### Struct Implementation Pattern

```rust
pub struct ConfigManager {
    pub configs: Vec<String>,
    config_dir: PathBuf,           // private
    tmux_config_path: PathBuf,     // private
}

impl ConfigManager {
    pub fn new() -> Result<Self, Box<dyn std::error::Error>> {
        // Initialization logic
    }

    pub fn public_method(&self) -> Result<(), Box<dyn std::error::Error>> {
        // Public interface
    }

    fn private_helper(param: &Type) -> Result<Type, Box<dyn std::error::Error>> {
        // Private helper
    }
}
```

### TUI-Specific Conventions

1. **Terminal Setup:** Use raw mode, enable mouse capture, enter alternate screen
2. **Event Handling:** Match on `KeyCode` with mode-specific logic
3. **Rendering:** Separate functions for each UI section (header, content, footer, popups)
4. **Cleanup:** Always restore terminal state on exit (even after errors)

---

## Common Tasks

### Adding a New CLI Command

1. **Define command in `cli.rs`:**
   ```rust
   pub enum Commands {
       // Existing commands...
       NewCommand { arg: String },
   }
   ```

2. **Add handler in `main.rs`:**
   ```rust
   Some(Commands::NewCommand { arg }) => {
       let config_manager = ConfigManager::new()?;
       // Handle command logic
       println!("✓ Success message");
   }
   ```

3. **Add config operation in `config.rs` if needed:**
   ```rust
   pub fn new_operation(&self, param: &str) -> Result<(), Box<dyn std::error::Error>> {
       // Implementation
   }
   ```

### Adding a New TUI Keybinding

1. **Update `tui.rs` event handler:**
   ```rust
   if let Event::Key(key) = event::read()? {
       match app.input_mode {
           InputMode::Normal => match key.code {
               // Existing keys...
               KeyCode::Char('x') => {
                   app.new_action()?;
               }
               _ => {}
           }
           // Other modes...
       }
   }
   ```

2. **Add method to `app.rs`:**
   ```rust
   pub fn new_action(&mut self) -> Result<(), Box<dyn std::error::Error>> {
       // Implementation
       self.set_status_message(format!("Action completed"));
       Ok(())
   }
   ```

3. **Update help text in `tui.rs`:**
   ```rust
   let help_text = match app.input_mode {
       InputMode::Normal => "...existing help..., x to new action",
       // Other modes...
   };
   ```

### Adding a New Input Mode

1. **Define mode in `app.rs`:**
   ```rust
   pub enum InputMode {
       // Existing modes...
       NewMode,
   }
   ```

2. **Add event handling in `tui.rs`:**
   ```rust
   InputMode::NewMode => match key.code {
       KeyCode::Enter => {
           // Handle confirmation
           app.input_mode = InputMode::Normal;
       }
       KeyCode::Esc => {
           // Handle cancellation
           app.input_mode = InputMode::Normal;
       }
       _ => {}
   }
   ```

3. **Add rendering logic in `tui.rs`:**
   ```rust
   if app.input_mode == InputMode::NewMode {
       // Render popup or special UI
   }
   ```

### Modifying File Operations

All file operations are centralized in `config.rs`. When modifying:

1. **Always check file existence first**
2. **Use descriptive error messages**
3. **Handle both success and error cases**
4. **Update ConfigManager state if list changes**

Example pattern:
```rust
pub fn operation(&self, name: &str) -> Result<(), Box<dyn std::error::Error>> {
    let path = self.config_dir.join(name);

    // Validation
    if !path.exists() {
        return Err(format!("File not found: {}", name).into());
    }

    // Operation
    fs::some_operation(&path)?;

    // Reload tmux if needed
    if let Some(path_str) = self.tmux_config_path.to_str() {
        let _ = std::process::Command::new("tmux")
            .args(["source-file", path_str])
            .output();
    }

    Ok(())
}
```

---

## Testing Strategy

### Current State

**No tests currently exist** in the codebase.

### Recommended Testing Approach

#### Unit Tests

Add to each module with `#[cfg(test)]`:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_ensure_conf_extension() {
        assert_eq!(ensure_conf_extension("dev".to_string()), "dev.conf");
        assert_eq!(ensure_conf_extension("dev.conf".to_string()), "dev.conf");
    }
}
```

#### Integration Tests

Create `tests/integration_test.rs`:

```rust
use std::fs;
use std::path::PathBuf;
use tempfile::TempDir;

#[test]
fn test_save_and_apply_config() {
    let temp_dir = TempDir::new().unwrap();
    // Test full workflow
}
```

#### Testable Areas

1. **cli.rs:** `ensure_conf_extension()` - pure function
2. **config.rs:** All file operations - use temp directories
3. **app.rs:** State management - no file I/O required
4. **TUI:** Integration tests for event handling (harder, lower priority)

#### Dependencies for Testing

Add to `Cargo.toml`:
```toml
[dev-dependencies]
tempfile = "3.0"
```

---

## File Reference

### main.rs (58 lines)

**Purpose:** Entry point and CLI command routing

**Key Responsibilities:**
- Parse CLI arguments with `Cli::parse()`
- Route commands to appropriate handlers
- Initialize ConfigManager
- Launch TUI when no command provided

**When to Modify:**
- Adding new CLI commands
- Changing command routing logic
- Modifying user-facing messages

**Important Functions:**
- `main()` - Entry point with match statement for command routing

---

### cli.rs (31 lines)

**Purpose:** CLI argument definitions using clap

**Key Responsibilities:**
- Define `Cli` struct with `#[derive(Parser)]`
- Define `Commands` enum with subcommands
- Provide `ensure_conf_extension()` utility

**When to Modify:**
- Adding new CLI commands
- Changing command arguments
- Adding flags or options

**Important Types:**
- `Cli` - Top-level CLI struct
- `Commands` - Enum of all subcommands (List, Apply, Save, Update, Delete)

**Important Functions:**
- `ensure_conf_extension()` - Normalizes config names to have `.conf` extension

---

### config.rs (120 lines)

**Purpose:** Configuration file management

**Key Responsibilities:**
- Manage `~/.config/tmucks/` directory
- CRUD operations on config files
- Interact with tmux for reloading
- Validate file operations

**When to Modify:**
- Adding new file operations
- Changing config directory location
- Modifying tmux interaction
- Adding validation logic

**Important Types:**
- `ConfigManager` - Main struct with public `configs: Vec<String>` and private paths

**Important Functions:**
- `new()` - Initialize manager, create directory, read configs
- `read_configs()` - Private helper to scan directory
- `apply_config()` - Copy config to `~/.tmux.conf` and reload tmux
- `save_current_config()` - Save current config (error if exists)
- `update_config()` - Overwrite existing config (error if doesn't exist)
- `delete_config()` - Remove config file

**Key Paths:**
- Config directory: `~/.config/tmucks/`
- Tmux config: `~/.tmux.conf`

---

### app.rs (161 lines)

**Purpose:** Application state management for TUI

**Key Responsibilities:**
- Maintain app state (selected config, input buffer, mode)
- Manage three input modes (Normal, Saving, UpdateConfirm)
- Handle status messages with auto-dismiss
- Coordinate between UI events and config operations

**When to Modify:**
- Adding new input modes
- Changing navigation behavior
- Adding new TUI actions
- Modifying status message behavior

**Important Types:**
- `InputMode` - Enum with Normal, Saving, UpdateConfirm
- `App` - Main state struct

**Important Fields:**
- `config_manager: ConfigManager` - File operations
- `list_state: ListState` - ratatui selection state
- `status_message: String` - Current status text
- `input_mode: InputMode` - Current mode
- `input_buffer: String` - Text input accumulator
- `pending_update_config: Option<String>` - Config to update after confirmation
- `status_message_time: Option<Instant>` - For auto-dismiss
- `default_status_message: String` - Help text

**Important Functions:**
- `new()` - Initialize app with default state
- `next()` / `previous()` - Navigate list (wraps around)
- `apply_config()` - Apply selected config
- `delete_config()` - Delete selected config and refresh list
- `save_current_config()` - Save with given name and refresh list
- `start_update_mode()` - Enter UpdateConfirm mode
- `confirm_update()` / `cancel_update()` - Handle update confirmation
- `set_status_message()` - Set message with timestamp
- `update_status_message()` - Check for 5-second expiry

---

### tui.rs (576 lines - largest module)

**Purpose:** Terminal UI rendering and event handling

**Key Responsibilities:**
- Terminal initialization/cleanup
- Event loop with 100ms polling
- Keyboard event handling for all input modes
- UI rendering (header, content, footer, popups)
- Layout calculations
- Styling and colors

**When to Modify:**
- Adding new keybindings
- Changing UI layout
- Adding new UI elements
- Modifying colors/styles
- Changing event polling interval

**Important Functions:**
- `run()` - Main entry point, sets up terminal and event loop
- `restore_terminal()` - Cleanup function (always called)
- `run_app()` - Event loop with polling
- `ui()` - Main rendering function
- `render_header()` - Title and statistics
- `render_main_content()` - Config list or empty state
- `render_footer()` - Status bar and help text
- `render_update_confirmation()` - Popup dialog

**Event Handling:**
- `event::poll(Duration::from_millis(100))` - Non-blocking poll
- Match on `InputMode` then `KeyCode`
- Calls `app.update_status_message()` on each iteration (for auto-dismiss)

**Key Keybindings:**
- **Normal Mode:** j/k (navigate), Enter (apply), s (save), u (update), d (delete), q (quit)
- **Saving Mode:** Type to input, Enter (confirm), Esc (cancel)
- **UpdateConfirm Mode:** y (confirm), n (cancel)

---

## Key Dependencies

### ratatui (0.26)

**Purpose:** Terminal UI framework

**Usage:**
- `Frame` - Rendering context
- `Layout` - Constraint-based layouts
- `Paragraph`, `List`, `Block` - UI widgets
- `Style`, `Color`, `Modifier` - Styling

**Documentation:** https://ratatui.rs/

---

### crossterm (0.27)

**Purpose:** Cross-platform terminal manipulation

**Usage:**
- `terminal::enable_raw_mode()` - Capture all input
- `terminal::EnterAlternateScreen` - Use alternate buffer
- `event::poll()` - Non-blocking event polling
- `event::read()` - Read keyboard/mouse events
- `event::KeyCode` - Key code enum
- `execute!()` - Execute terminal commands

**Documentation:** https://docs.rs/crossterm/

---

### dirs (5.0)

**Purpose:** Cross-platform directory paths

**Usage:**
- `dirs::home_dir()` - Get home directory path

**Why:** Avoids hard-coding `~` or using shell expansion

---

### clap (4.5 with derive feature)

**Purpose:** Command-line argument parsing

**Usage:**
- `#[derive(Parser)]` - Auto-generate parser
- `#[derive(Subcommand)]` - Define subcommands
- `Cli::parse()` - Parse arguments

**Documentation:** https://docs.rs/clap/

---

## Working with This Codebase

### For AI Assistants

#### Before Making Changes

1. **Read relevant files first** - Always use Read tool before modifying
2. **Understand the module** - Check imports and struct definitions
3. **Check error handling** - Ensure consistency with existing patterns
4. **Review recent commits** - Understand recent changes and style

#### When Adding Features

1. **Start with tests** (even though none exist) - Consider testability
2. **Follow existing patterns** - State machine, error handling, file operations
3. **Update help text** - If adding keybindings or commands
4. **Maintain separation** - Config operations in config.rs, state in app.rs, UI in tui.rs
5. **Use descriptive errors** - Include context in error messages
6. **Auto-dismiss status** - Use `set_status_message()` for temporary feedback

#### When Fixing Bugs

1. **Identify the layer** - Is it UI (tui.rs), state (app.rs), or file ops (config.rs)?
2. **Check error paths** - Are errors properly propagated?
3. **Test edge cases** - Empty config list, missing files, invalid input
4. **Preserve UX** - Maintain auto-dismiss, confirmation dialogs, help text

#### Code Quality Checklist

- [ ] All functions return `Result<(), Box<dyn std::error::Error>>`
- [ ] Errors include descriptive messages with context
- [ ] Imports are organized (std, external, local)
- [ ] Naming follows conventions (snake_case, PascalCase)
- [ ] File existence checked before operations
- [ ] Status messages set for user feedback
- [ ] Help text updated if UI changed
- [ ] Terminal cleanup in error paths
- [ ] No unwrap() or panic!() in production code
- [ ] Code formatted with `cargo fmt`
- [ ] No clippy warnings

#### Common Pitfalls to Avoid

1. **Don't unwrap()** - Use `?` operator or proper error handling
2. **Don't hard-code paths** - Use `dirs::home_dir()` and `PathBuf`
3. **Don't forget terminal cleanup** - Always restore terminal state
4. **Don't break separation of concerns** - Keep UI logic in tui.rs, state in app.rs
5. **Don't skip status messages** - Users need feedback for actions
6. **Don't ignore input modes** - Different keys do different things in different modes
7. **Don't assume tmux is running** - Use `let _ = ...output()` for tmux commands

#### When Stuck

1. **Check CRUSH.md** - Development commands and style guide
2. **Check recent commits** - See how similar features were implemented
3. **Check dependency docs** - ratatui.rs, docs.rs/crossterm, docs.rs/clap
4. **Test in TUI** - Run `cargo run` to see actual behavior
5. **Use cargo clippy** - Catch common mistakes

---

## Quick Reference

### File System Layout

```
~/.config/tmucks/       - Stored configs (dev.conf, prod.conf, etc.)
~/.tmux.conf            - Active tmux configuration
```

### Build Artifacts

```
/target/                - Compiled binaries (gitignored)
/target/debug/tmucks    - Development build
/target/release/tmucks  - Release build (use `cargo build --release`)
```

### Module Import Paths

```rust
use crate::app::App;
use crate::cli::{Cli, Commands};
use crate::config::ConfigManager;
// tui is only called, not imported in other modules
```

### Common Type Signatures

```rust
fn main() -> Result<(), Box<dyn std::error::Error>>
pub fn new() -> Result<Self, Box<dyn std::error::Error>>
pub fn operation(&self) -> Result<(), Box<dyn std::error::Error>>
fn helper() -> Result<Type, Box<dyn std::error::Error>>
```

---

## Changelog

### 2025-11-25

- Initial CLAUDE.md creation
- Documented current state of codebase (946 lines, 5 modules)
- Cataloged architecture patterns and conventions
- Added testing strategy recommendations
- Documented all modules and their responsibilities

---

## Notes for Future Development

### Potential Enhancements

1. **Testing Infrastructure** - Add unit and integration tests
2. **Config Validation** - Validate tmux config syntax before saving
3. **Config Diff** - Show differences between saved configs
4. **Config Search** - Filter configs by name in TUI
5. **Backup on Update** - Create backup before overwriting configs
6. **Config Templates** - Provide starter templates for common setups
7. **Import/Export** - Share configs across machines
8. **Config Metadata** - Store description, tags, creation date

### Known Limitations

1. No tests exist yet
2. No config validation (saves any file as .conf)
3. No backup system (destructive updates/deletes)
4. No search/filter in TUI
5. No config preview before applying
6. No multi-select operations
7. Update requires confirmation but delete doesn't

### Compatibility Notes

- **Platform:** Cross-platform (Linux, macOS, Windows with WSL)
- **Rust Edition:** 2021
- **MSRV:** Not specified (likely 1.70+)
- **Tmux:** Any version (uses basic `source-file` command)

---

**End of CLAUDE.md**
