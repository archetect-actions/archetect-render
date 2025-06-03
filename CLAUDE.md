# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Action called "Archetect Render Action" that renders Archetypes using the Archetect tool. It's a composite action that downloads the Archetect binary, processes JSON answers, and executes the render command.

## Architecture

The action is implemented as a composite action in `action.yml` with three main steps:
1. **Install Archetect**: Downloads the Archetect binary from GitHub releases to the runner's temp directory
2. **Write Answers**: Processes JSON input answers and writes them to a temporary file
3. **Render Archetype**: Executes the archetect render command with provided parameters

## Key Components

- `action.yml`: The main action definition with inputs, steps, and composite shell commands
- `.github/workflows/release.yml`: Release workflow that uses p6m-actions/repository-release for version management

## Inputs

- `version`: Archetect version to use (default: v2.0.0)
- `source`: Source directory or Git repo containing an Archetype (required)
- `destination`: Output destination (default: current directory)
- `answers`: JSON object with key/value answers
- `args`: Additional command-line arguments for archetect

## Release Process

Releases are managed through a manual workflow dispatch with two options:
- `minor_release`: Creates a minor version release
- `major_branch`: Creates a major branch release

The release workflow uses `p6m-actions/repository-release@v1` for automated versioning and tagging.