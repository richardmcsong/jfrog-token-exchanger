# Claude Code Guidelines

This file contains important notes and guidelines for Claude Code when working with this repository.

## Capabilities

### Workflow File Editing
Claude Code **CAN** edit files in the `.github/workflows` directory. The GitHub App has been configured with permissions to modify workflow files. This allows Claude to:
- Update workflow configurations
- Modify CI/CD pipelines
- Adjust GitHub Actions settings

Previously, workflow editing was restricted, but this limitation has been removed through the updated GitHub App installation.

### Comment Editing Limitations
When editing previous review comments via the GitHub API:
- Claude may lack permissions to edit comments created under different authentication contexts
- Comments may be locked or frozen by repository settings
- GitHub API rate limits may prevent bulk comment edits
- If comment editing fails, Claude should gracefully skip those comments and continue with the review
- This is expected behavior and not a critical failure

## Repository Context

This repository contains JFrog token exchange functionality and related tooling.
