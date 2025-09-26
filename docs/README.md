# 📁 .github Repository  
*No description provided.*

> This repository contains shared GitHub configuration files (workflows, issue & pull‑request templates, funding files, etc.) that can be reused across multiple projects. The documentation below explains how to **install**, **use**, and **extend** the assets in this repo, along with an overview of the available **API** (inputs/outputs of the bundled GitHub Actions) and concrete **examples**.

---

## Table of Contents
1. [Installation](#installation)  
2. [Usage](#usage)  
   - 2.1 [Workflows](#workflows)  
   - 2.2 [Issue & PR Templates](#issue--pr-templates)  
   - 2.3 [Funding & Community Files](#funding--community-files)  
3. [API Documentation](#api-documentation)  
   - 3.1 [Custom Actions](#custom-actions)  
   - 3.2 [Reusable Workflow Inputs/Outputs](#reusable-workflow-inputsoutputs)  
4. [Examples](#examples)  
   - 4.1 [CI/CD Pipeline](#ci-cd-pipeline)  
   - 4.2 [Automatic Release Draft](#automatic-release-draft)  
   - 4.3 [Issue Triage Bot](#issue-triage-bot)  
5. [Contributing](#contributing)  
6. [License](#license)  

---

## Installation

> The `.github` repository is **not** a library you `npm install` or `pip install`. Instead, you **import** its contents into another repository.

### 1️⃣ Add as a Git Submodule (recommended)

```bash
# From the root of the target repository
git submodule add https://github.com/<ORG_OR_USER>/.github .github
git submodule update --init --recursive
```

> **Tip:** Keep the submodule up‑to‑date with `git submodule update --remote`.

### 2️⃣ Use the `actions/checkout` reusable workflow (GitHub‑only)

If you only need the workflows, you can reference them directly without a submodule:

```yaml
# .github/workflows/ci.yml in your target repo
name: CI
on: [push, pull_request]

jobs:
  build:
    uses: <ORG_OR_USER>/.github/.github/workflows/ci.yml@main
    with:
      node-version: '20'
```

### 3️⃣ Copy‑Paste (quick‑start)

```bash
# Copy the whole folder (or selected files) into your repo
cp -R .github/* /path/to/your/repo/.github/
```

> **Caution:** Manual copies won’t receive automatic updates. Use submodules or reusable workflows for long‑term maintenance.

---

## Usage

### Workflows

All reusable workflows live under `.github/workflows/`. They are written as **reusable** workflows (i.e., `uses:` syntax) and can be called from any repository.

| Workflow | Purpose | Typical `with` inputs |
|----------|---------|-----------------------|
| `ci.yml` | Run tests, lint, and build | `node-version`, `python-version` |
| `release.yml` | Draft a new GitHub Release | `tag`, `release-notes` |
| `codeql-analysis.yml` | Security scanning with CodeQL | `language`, `upload-sarif` |
| `docker-publish.yml` | Build & push Docker images | `image-name`, `registry`, `tags` |

#### Example: Calling a reusable workflow

```yaml
# .github/workflows/ci-caller.yml (in your repo)
name: CI (caller)

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: <ORG_OR_USER>/.github/.github/workflows/ci.yml@main
    with:
      node-version: '20'
      python-version: '3.11'
```

### Issue & PR Templates

- **Issue templates** live in `.github/ISSUE_TEMPLATE/`.  
- **Pull‑request template** lives in `.github/PULL_REQUEST_TEMPLATE.md`.

To enable them, simply keep the files in the same location in your repository. GitHub will automatically surface them when users create new issues or PRs.

#### Customizing a template

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug report
description: Report a bug in the project
title: "[BUG] <short description>"
labels: ["bug"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        Thanks for taking the time to report a bug! Please fill out the sections below.
  - type: textarea
    id: steps
    attributes:
      label: Steps to reproduce
      description: Provide a clear set of steps.
      placeholder: |
        1. ...
        2. ...
```

### Funding & Community Files

- `FUNDING.yml` – defines how contributors can sponsor the project.  
- `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md` – standard community guidelines.

Just keep these files at the repository root (or inside `.github/`) and GitHub will render them automatically.

---

## API Documentation

> The term **API** here refers to the inputs/outputs of the **custom GitHub Actions** and **reusable workflows** shipped with this repo.

### Custom Actions

| Action | Path | Description | Inputs | Outputs |
|--------|------|-------------|--------|---------|
| `setup-node` | `.github/actions/setup-node/` | Installs a specific Node.js version and caches `node_modules`. | `node-version` (string, required) | `cache-hit` (boolean) |
| `docker-login` | `.github/actions/docker-login/` | Logs into Docker Hub / GitHub Container Registry. | `registry` (string, default: `ghcr.io`), `username`, `password` | — |
| `release-notes` | `.github/actions/release-notes/` | Generates release notes from merged PR titles. | `from-tag`, `to-tag` | `notes` (string) |

#### Example: Using `setup-node`

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v4

  - name: Set up Node.js
    uses: ./.github/actions/setup-node
    with:
      node-version: '20'
```

### Reusable Workflow Inputs / Outputs

Reusable workflows expose **inputs** (via `with:`) and **outputs** (via `outputs:`). Below is a generic pattern used across the repo.

```yaml
# .github/workflows/ci.yml (excerpt)
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      python-version:
        required: false
        type: string
    outputs:
      test-status:
        description: "Result of the test job (success/failure)"
        value: ${{ jobs.test.result }}
```

When you call the workflow:

```yaml
jobs:
  ci:
    uses: <ORG_OR_USER>/.github/.github/workflows/ci.yml@main
    with:
      node-version: '20'
    outputs:
      test-status: ${{ steps.ci.outputs.test-status }}
```

---

## Examples

### 1️⃣ CI/CD Pipeline (Node + Python)

```yaml
# .github/workflows/ci.yml (in your repo)
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: <ORG_OR_USER>/.github/.github/workflows/ci.yml@main
    with:
      node-version: '20'
      python-version: '3.11'
```

*What it does:*  
- Installs Node 20 and Python 3.11.  
- Caches dependencies.  
- Runs `npm test`, `pytest`, and linting.  
- Uploads test coverage as an artifact.

---

### 2️⃣ Automatic Release Draft

```yaml
# .github/workflows/release-draft.yml
name: Draft Release

on:
  push:
    tags:
      - 'v*.*.*'   # e.g., v1.2.3

jobs:
  draft:
    uses: <ORG_OR_USER>/.github/.github/workflows/release.yml@main
    with:
      tag: ${{ github.ref_name }}
      release-notes: ${{ steps.notes.outputs.notes }}
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

*Features:*  
- Generates release notes from merged PR titles.  
- Creates a **draft** release (editable before publishing).  
