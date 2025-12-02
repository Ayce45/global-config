# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a VSCode extension called "Global Config" that copies shared configuration files (tasks.json, settings.json, launch.json, etc.) from a global location into workspace `.vscode` folders. It's designed to solve the problem of having to manually copy config files into hundreds of workspace folders.

**Main entry point**: [extension.js](extension.js)

## Key Architecture

The extension follows a simple architecture:

1. **Single Command**: The extension registers one command (`global-config.copy`) that performs all operations
2. **File Operations**: Supports three modes for transferring files:
   - Copy (default)
   - Symlink (soft links via `global-config.links` setting)
   - Hard link (via `global-config.hardLinks` setting)
3. **Glob Pattern Matching**: Uses `micromatch` library to match files in settings (`links`, `hardLinks`, `destinations`)
4. **Variable Substitution**: Supports `${workspaceFolder}` and environment variables (e.g., `${HOME}`) in destination paths using regex replacement

## Core Logic Flow

The `copyConfig()` function in [extension.js](extension.js):

1. Reads the source folder from settings (`global-config.folder`, defaults to `~/.vscode/`)
2. Checks if source folder has subfolders - if yes, prompts user to select one
3. For each workspace folder, calls `copyToWorkspace()`:
   - Ensures `.vscode` directory exists
   - Reads all files from source
   - For each file:
     - Determines destination (default `.vscode` or alternative from `global-config.destinations`)
     - Applies variable substitution to destination path
     - Checks if file already exists (skips if it does)
     - Creates hard link, symlink, or copy based on settings
4. Logs all operations to "Global Config" output channel

## Configuration Settings

All settings are prefixed with `global-config.`:

- `folder`: Source folder for config files
- `links`: Array of files/globs to symlink
- `hardLinks`: Array of files/globs to hard link
- `destinations`: Object mapping files/globs to alternative destination paths

## Important Implementation Details

- **Windows Symlinks**: Require Developer Mode to work correctly
- **Multi-root Workspace Support**: Extension processes each workspace folder independently
- **Glob Pattern Support**: `findMatch()` function in [extension.js:50-63](extension.js#L50-L63) uses `micromatch.isMatch()` to match files against glob patterns in settings
- **No Overwriting**: Existing files are never replaced (checked at [extension.js:97-100](extension.js#L97-L100))
- **Activation**: Extension activates on any event (`*` in `activationEvents`)

## Development Commands

```bash
# Install dependencies
npm install

# Package extension (requires vsce)
vsce package
```

Note: This project has no test suite, linting, or build scripts defined in package.json.

## Dependencies

- `fs-extra`: Enhanced file system operations (copying, linking)
- `micromatch`: Glob pattern matching for file selection
