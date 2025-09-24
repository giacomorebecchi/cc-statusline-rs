# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust-based statusline generator for Claude Code that creates rich terminal status displays with git information, model context usage, session details, and PR status. The tool processes JSON input from stdin and outputs a formatted statusline with ANSI color codes.

## Key Commands

### Build and Run

```bash
cargo build --release
cargo run
```

### Testing with sample data

```bash
cargo run < test.json
```

**Important**: Never generate test JSON files. Always look for existing ones in `~/.claude` directory for testing.

### Development

```bash
cargo check    # Quick syntax/type checking
cargo clippy   # Linting
cargo fmt      # Code formatting
```

## Architecture

### Core Components

**src/main.rs**: Entry point that handles command-line arguments (`--short`, `--skip-pr-status`) and calls the main statusline function.

**statusline() function**: The main orchestrator that:

1. Parses JSON input containing workspace, model, and session information
2. Determines display strategy based on directory type (non-git, git repo, worktree)
3. Assembles final output string with proper color coding and formatting

### Key Features

**Smart Path Display**: Shows abbreviated paths in `--short` mode, hiding standard project locations (`~/Projects/{repo_name}`) but always showing non-standard paths.

**Git Integration**:

- Detects git repositories and worktrees
- Shows branch names with special `↟` indicator for worktrees
- Displays git status with file change counts (+, ~, -, ?) and line deltas (Δ)

**Context Management**:

- Parses transcript files to calculate context usage percentage
- Color-codes context percentage (red ≥90%, orange ≥70%, yellow ≥50%, gray <50%)
- Handles both string and numeric timestamp formats

**Caching System**:

- PR URLs cached for 60 seconds in `.git/statusbar/pr-{branch}`
- PR status (CI checks) cached for 30 seconds in `.git/statusbar/pr-status-{branch}`
- Session summaries cached in `.git/statusbar/session-{id}-summary`

**Session Analysis**:

- Extracts session duration from transcript timestamps
- Generates AI-powered summaries of user's first substantial message
- Displays session ID and duration information

**PR Status Display**:

- Shows GitHub PR URLs and CI check status using `gh` CLI
- Groups checks by status (fail ✗, pending ○, pass ✓) with counts and names
- Handles missing `gh` CLI gracefully

### Display Format

The output follows this order: `path [branch+status] • context%+model • summary • PR+status • session_id • duration`

Example: `~/project [main +2 ~1] • 45% Opus • fix login bug • https://github.com/... ✓3 • abc123 • 15m`

### Dependencies

- **serde_json**: JSON parsing for input data and transcript analysis
- **chrono**: Timestamp parsing and duration calculations
- **External tools**: Requires `git` and optionally `gh` CLI for full functionality

### Input Format

Expects JSON on stdin with fields:

- `workspace.current_dir`: Working directory path
- `model.display_name`: AI model name for display
- `transcript_path`: Path to conversation transcript file
- `session_id`: Unique session identifier

The `test.json` file shows the expected input structure for development and testing.
