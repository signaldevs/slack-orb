# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a CircleCI Orb that provides Slack notification commands. It's a light wrapper around the official `circleci/slack` orb to abstract common SignalDevs configuration patterns. The orb is published to the CircleCI Orb Registry at `signaldevs/slack`.

## Development Commands

### Linting and Validation
```bash
# The orb-tools/lint job in CircleCI handles YAML linting automatically
# No local linting commands configured
```

### Testing Workflow
The repository uses CircleCI's orb-tools pipeline for development:
1. Push changes to a feature branch
2. CircleCI runs the `test-pack` workflow (lint + pack)
3. Manual approval required to publish dev version
4. Dev version published as `dev:${CIRCLE_SHA1:0:7}`
5. Integration tests run against the dev version
6. On merge to `develop` with semver tag, production version publishes

## Architecture

### Orb Structure
CircleCI orbs are packaged YAML configurations. This orb follows the standard structure:

- `src/@orb.yml` - Main orb metadata, description, and imports the base `circleci/slack@6.1.2` orb along with `circleci/curl@2.1.0` and `circleci/jq@3.0.0` for tool installation
- `src/commands/*.yml` - Reusable command definitions
- `src/examples/*.yml` - Usage examples displayed in the orb registry

### Commands

**notify_failure** (`src/commands/notify_failure.yml`)
- Executes on job failure (`when: on_fail`)
- Installs curl and jq via circleci orbs
- Fetches workflow name via CircleCI API using `CIRCLE_API_TOKEN`
- Downloads custom template from `https://cdn.signalapis.com/slack-templates/failed-build-template.json`
- Sends two notifications with retry logic (2-3 retries):
  1. Standard failure notification to `SLACK_DEFAULT_CHANNEL`
  2. Critical branch failures (main/master/autorelease/develop) to hardcoded channel `C05US7T31QR` (alerts-dev-ops-production)

**notify_release** (`src/commands/notify_release.yml`)
- Executes on job success for release notifications
- Installs curl via circleci orb
- Extracts release version from branch name format: `release/X.Y.Z`
- Downloads template from `https://cdn.signalapis.com/slack-templates/success-new-release-template.json`
- Posts to `SLACK_DEFAULT_CHANNEL` with retry logic (1 retry)

### Required Environment Variables

All commands require these environment variables (typically set via a `slack` context):
- `CIRCLE_API_TOKEN` - CircleCI personal API token for workflow metadata
- `SLACK_ACCESS_TOKEN` - Slack OAuth token from bot setup
- `SLACK_DEFAULT_CHANNEL` - Slack channel ID (not name) for notifications

### Publishing Process

Pull requests to `develop` must include a semver tag in the title:
- `[semver:major]` - 1.0.0 increment
- `[semver:minor]` - x.1.0 increment
- `[semver:patch]` - x.x.1 increment
- `[semver:skip]` - No release

The semver tag must be preserved in the squash-merge commit message. On merge to `develop`, after manual approval, the orb automatically publishes to the CircleCI Orb Registry.

### Pipeline Flow

1. **test-pack workflow** (runs on every commit):
   - `orb-tools/lint` - Validate YAML syntax
   - `orb-tools/pack` - Package orb source
   - Manual approval gate (`hold-for-dev-publish`)
   - `orb-tools/publish-dev` - Publish dev version (requires `orb_publishing` context)
   - `orb-tools/trigger-integration-tests-workflow` - Trigger integration tests

2. **integration-test_deploy workflow** (triggered by test-pack):
   - `integration-test-1` - Run integration tests (currently empty)
   - `orb-tools/dev-promote-prod-from-commit-subject` - Promote to production if semver tag present

### Template System

Commands dynamically load Slack message templates from `cdn.signalapis.com`. Templates are Block Kit JSON that can reference environment variables:
- `CIRCLE_WORKFLOW_NAME` - Fetched via API
- `RELEASE_VERSION` - Parsed from branch name (release command only)
- Standard CircleCI variables (`CIRCLE_BUILD_URL`, `CIRCLE_PROJECT_REPONAME`, etc.)

### Version 6.1.2 Upgrade Changes

This orb uses CircleCI Slack Orb v6.1.2, which introduced several enhancements:

**New Parameters Implemented:**
- `retries` - Number of times to retry failed Slack API calls
  - notify_failure: 2-3 retries (critical operational alerts)
  - notify_release: 1 retry (success notifications, lower urgency)
- `retry_delay` - Seconds between retry attempts (30-60s)
- `unfurl_links` - Always enabled for message context visibility
- `unfurl_media` - Enabled for releases (artifacts), disabled for failures (reduce clutter)

**Deprecated Commands Removed:**
- notify_e2e_failure: Command removed as it is no longer needed

**Tool Installation:**
- Commands now use circleci/curl and circleci/jq orbs to ensure curl and jq are installed
- notify_failure: Installs both curl and jq (needed for API calls and JSON parsing)
- notify_release: Installs curl (needed for template download)

**Parameters NOT Currently Used (Future Considerations):**
- `thread_id` - Could enable threaded conversations
- `scheduled_offset_seconds` - Not applicable to on-fail/on-pass triggers
- `windows_execution` - Not relevant to CircleCI-based workflow

### Context Usage

The repository expects a CircleCI context named `orb_publishing` containing credentials for publishing orbs. Consumer projects should create a `slack` context with the required Slack/CircleCI tokens.
