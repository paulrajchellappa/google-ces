# GitHub Actions Workflow Documentation

## Table of Contents

- [Overview](#overview)
- [Core Concepts](#core-concepts)
- [Workflow File Structure](#workflow-file-structure)
- [Triggers (on)](#triggers-on)
- [Jobs](#jobs)
- [Steps](#steps)
- [Actions](#actions)
- [Environment Variables & Secrets](#environment-variables--secrets)
- [Expressions & Contexts](#expressions--contexts)
- [Artifacts & Caching](#artifacts--caching)
- [Common Workflow Patterns](#common-workflow-patterns)
- [Best Practices](#best-practices)

---

## Overview

GitHub Actions is a CI/CD platform built into GitHub that allows you to automate build, test, and deployment pipelines. Workflows are defined in YAML files stored in the `.github/workflows/` directory of your repository.

**Key benefits:**
- Native GitHub integration (PRs, issues, releases)
- Thousands of community-built actions on the Marketplace
- Matrix builds for testing across multiple environments
- Free tier for public repositories

---

## Core Concepts

| Concept | Description |
|--------|-------------|
| **Workflow** | An automated process defined in a YAML file |
| **Event** | A trigger that starts a workflow (push, PR, schedule, etc.) |
| **Job** | A set of steps that run on the same runner |
| **Step** | An individual task within a job (run a command or use an action) |
| **Action** | A reusable unit of code (from Marketplace or custom) |
| **Runner** | The server that executes your jobs (GitHub-hosted or self-hosted) |

---

## Workflow File Structure

```yaml
name: CI Pipeline                  # Display name in GitHub UI

on:                                # Trigger(s)
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:                               # Workflow-level environment variables
  NODE_VERSION: '20'

jobs:
  build:                           # Job ID
    name: Build & Test             # Display name
    runs-on: ubuntu-latest         # Runner type

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

---

## Triggers (`on`)

### Push & Pull Request

```yaml
on:
  push:
    branches: [main, develop]
    tags: ['v*.*.*']
    paths:
      - 'src/**'
      - '!docs/**'         # Exclude docs changes

  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]
```

### Schedule (Cron)

```yaml
on:
  schedule:
    - cron: '0 8 * * 1-5'   # 8am UTC, Mon–Fri
```

### Manual Trigger

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]
      dry_run:
        description: 'Run without deploying'
        type: boolean
        default: false
```

### Other Common Triggers

```yaml
on:
  release:
    types: [published]

  workflow_call:              # Called by another workflow
    inputs:
      version:
        type: string
        required: true

  repository_dispatch:        # Triggered via API
    types: [deploy-event]
```

---

## Jobs

### Runner Types

```yaml
jobs:
  linux-job:
    runs-on: ubuntu-latest      # ubuntu-22.04, ubuntu-20.04

  windows-job:
    runs-on: windows-latest     # windows-2022, windows-2019

  macos-job:
    runs-on: macos-latest       # macos-14, macos-13, macos-12

  self-hosted-job:
    runs-on: [self-hosted, linux, x64]
```

### Job Dependencies

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    needs: build                # Runs after build succeeds
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."

  deploy:
    needs: [build, test]        # Runs after both succeed
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

### Matrix Builds

```yaml
jobs:
  test:
    strategy:
      fail-fast: false          # Don't cancel other jobs on failure
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [18, 20, 22]
        exclude:
          - os: windows-latest
            node: 18

    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

### Conditional Jobs

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production"
```

---

## Steps

### Run Shell Commands

```yaml
steps:
  - name: Single-line command
    run: echo "Hello World"

  - name: Multi-line commands
    run: |
      echo "Step 1"
      echo "Step 2"
      npm run build

  - name: PowerShell (Windows)
    shell: pwsh
    run: Write-Output "Hello from PowerShell"

  - name: Python script
    shell: python
    run: |
      import os
      print(os.environ.get('GITHUB_SHA'))
```

### Conditional Steps

```yaml
steps:
  - name: Only on main
    if: github.ref == 'refs/heads/main'
    run: echo "This is main"

  - name: Only on failure
    if: failure()
    run: echo "Something went wrong"

  - name: Always run (cleanup)
    if: always()
    run: echo "Cleanup"
```

### Step Outputs

```yaml
steps:
  - name: Generate version
    id: version
    run: echo "tag=v1.0.0" >> $GITHUB_OUTPUT

  - name: Use version
    run: echo "Deploying ${{ steps.version.outputs.tag }}"
```

---

## Actions

### Using Marketplace Actions

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4
    with:
      fetch-depth: 0            # Full history

  - name: Setup Node.js
    uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'

  - name: Upload artifact
    uses: actions/upload-artifact@v4
    with:
      name: build-output
      path: dist/
      retention-days: 7
```

### Commonly Used Actions

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Check out repository code |
| `actions/setup-node@v4` | Set up Node.js environment |
| `actions/setup-python@v5` | Set up Python environment |
| `actions/setup-java@v4` | Set up Java environment |
| `actions/upload-artifact@v4` | Upload build artifacts |
| `actions/download-artifact@v4` | Download artifacts between jobs |
| `actions/cache@v4` | Cache dependencies |
| `docker/build-push-action@v5` | Build and push Docker images |

### Custom Composite Action

Create `.github/actions/my-action/action.yml`:

```yaml
name: 'My Custom Action'
description: 'Does something useful'

inputs:
  token:
    description: 'GitHub token'
    required: true

outputs:
  result:
    description: 'The result'
    value: ${{ steps.compute.outputs.result }}

runs:
  using: 'composite'
  steps:
    - id: compute
      shell: bash
      run: echo "result=done" >> $GITHUB_OUTPUT
```

Use it in your workflow:

```yaml
steps:
  - uses: ./.github/actions/my-action
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Environment Variables & Secrets

### Setting Variables

```yaml
env:
  GLOBAL_VAR: 'available everywhere'

jobs:
  build:
    env:
      JOB_VAR: 'available in all steps of this job'
    steps:
      - name: Step with own var
        env:
          STEP_VAR: 'available only here'
        run: echo "$GLOBAL_VAR $JOB_VAR $STEP_VAR"
```

### Using Secrets

```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    run: ./deploy.sh
```

> **Note:** Secrets are masked in logs. Never print them directly with `echo`.

### Default Environment Variables

| Variable | Description |
|----------|-------------|
| `GITHUB_SHA` | Commit SHA that triggered the workflow |
| `GITHUB_REF` | Branch or tag ref (e.g., `refs/heads/main`) |
| `GITHUB_REPOSITORY` | Owner and repo name (e.g., `org/repo`) |
| `GITHUB_ACTOR` | Username that triggered the workflow |
| `GITHUB_WORKSPACE` | Path to checked-out repository |
| `GITHUB_RUN_ID` | Unique ID for this workflow run |
| `GITHUB_EVENT_NAME` | Name of the triggering event |

### Writing to GITHUB_ENV

```yaml
steps:
  - name: Set dynamic variable
    run: echo "BUILD_DATE=$(date +'%Y-%m-%d')" >> $GITHUB_ENV

  - name: Use it
    run: echo "Built on $BUILD_DATE"
```

---

## Expressions & Contexts

### Syntax

```yaml
${{ <expression> }}
```

### Useful Contexts

```yaml
# github context
${{ github.sha }}
${{ github.ref_name }}
${{ github.event.pull_request.number }}
${{ github.actor }}

# runner context
${{ runner.os }}
${{ runner.temp }}

# secrets context
${{ secrets.MY_SECRET }}

# inputs context (workflow_dispatch)
${{ inputs.environment }}
```

### Built-in Functions

```yaml
# String functions
${{ contains(github.ref, 'main') }}
${{ startsWith(github.ref, 'refs/tags/') }}
${{ format('Hello {0}!', github.actor) }}

# Status functions (in if conditions)
${{ success() }}
${{ failure() }}
${{ cancelled() }}
${{ always() }}
```

---

## Artifacts & Caching

### Uploading & Downloading Artifacts

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist-files
          path: dist/
      - run: ./deploy.sh
```

### Caching Dependencies

```yaml
steps:
  - uses: actions/cache@v4
    with:
      path: ~/.npm
      key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-npm-

  - run: npm ci
```

---

## Common Workflow Patterns

### CI Pipeline (Node.js)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

### Deploy on Tag

```yaml
name: Release

on:
  push:
    tags: ['v*.*.*']

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: dist/**
          generate_release_notes: true
```

### Docker Build & Push

```yaml
name: Docker

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

### Scheduled Cleanup

```yaml
name: Cleanup

on:
  schedule:
    - cron: '0 0 * * 0'   # Every Sunday at midnight

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Delete old branches
        run: ./scripts/cleanup-branches.sh
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Best Practices

**Security**
- Always pin actions to a specific SHA or major version tag (`@v4`, not `@latest`)
- Never hardcode secrets in workflow files — always use `${{ secrets.NAME }}`
- Use `permissions` to restrict the `GITHUB_TOKEN` to minimum required access
- Avoid printing secrets, even indirectly

**Performance**
- Use `actions/cache` for dependencies to speed up runs
- Use `fail-fast: false` on matrix builds when you need all results
- Split long workflows into reusable workflow files with `workflow_call`
- Use `paths` filters to skip workflows on unrelated changes

**Reliability**
- Set `timeout-minutes` on jobs to prevent runaway builds
- Use `continue-on-error: true` only when a step's failure is acceptable
- Always test workflow changes on a feature branch before merging to main

**Permissions**

```yaml
permissions:
  contents: read          # Default; add write only when needed
  pull-requests: write    # For PR comments
  packages: write         # For pushing to GitHub Container Registry
  id-token: write         # For OIDC authentication
```

---

*Last updated: April 2026 — GitHub Actions documentation: [docs.github.com/actions](https://docs.github.com/actions)*
