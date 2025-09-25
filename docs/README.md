# 📚 .github Repository – Documentation  

> **Repository name:** `.github`  
> **Description:** *None*  

This repository is intended to hold shared GitHub configuration for an organization or a set of projects. It typically contains:

- Issue and pull‑request templates  
- Community health files (CODE_OF_CONDUCT, CONTRIBUTING, SECURITY, etc.)  
- GitHub Actions workflows (CI/CD, linting, release automation, etc.)  
- Funding and sponsorship files  
- Other repository‑wide settings (e.g., `dependabot.yml`, `renovate.json`)

Below you’ll find a complete guide on how to **install**, **use**, **extend**, and **example** configurations for this repository.

---

## Table of Contents
1. [Installation](#installation)  
2. [Usage](#usage)  
   - 2.1 [Adding the repository to a project](#adding-the-repository-to-a-project)  
   - 2.2 [Enabling GitHub Actions workflows](#enabling-github-actions-workflows)  
   - 2.3 [Customising templates & community files](#customising-templates--community-files)  
3. [API Documentation](#api-documentation)  
   - 3.1 [Workflow Dispatch API](#workflow-dispatch-api)  
   - 3.2 [Repository‑wide Settings API](#repository-wide-settings-api)  
4. [Examples](#examples)  
   - 4.1 [Typical CI workflow](#typical-ci-workflow)  
   - 4.2 [Issue template for bug reports](#issue-template-for-bug-reports)  
   - 4.3 [Dependabot configuration](#dependabot-configuration)  
5. [Contributing & License](#contributing--license)  

---

## Installation

> The `.github` repository does **not** contain compiled code, so “installation” means **making its contents available to other repositories**.

| Method | When to use | Steps |
|--------|-------------|-------|
| **Git Submodule** | You want a single source of truth for all shared files across many repos. | 1. In the target repo: <br>`git submodule add https://github.com/your-org/.github .github` <br>2. Commit the submodule reference. <br>3. Run `git submodule update --init --recursive` after cloning. |
| **GitHub Repository‑Level Inheritance** (recommended) | You want GitHub to automatically apply the files without extra git plumbing. | 1. Create a **public** or **private** repository named `.github` in the same organization/user. <br>2. Add the desired files (templates, workflows, etc.). <br>3. GitHub will automatically apply them to **all** repositories in that org/user that **do not** have a conflicting file. |
| **Copy‑Paste / Sync Script** | You need a one‑off copy or want to keep a local fork. | 1. Clone the repo: `git clone https://github.com/your-org/.github.git` <br>2. Copy the needed files into your repo (e.g., `cp -r .github/.github/* .`). <br>3. Add them to version control. |
| **GitHub CLI (`gh`)** | You prefer a CLI‑only workflow. | ```bash<br># Install the repo as a template (if it’s marked as a template)<br>gh repo create my-repo --template your-org/.github<br>``` |

### Prerequisites
- A GitHub account with **write** permission on the target repository (or admin rights on the organization for inheritance).
- Git ≥ 2.13 (for submodule support) or the **GitHub CLI** (`gh`) if you prefer that route.
- (Optional) **Node.js** / **Python** / other runtimes if the workflows you import require them.

---

## Usage

### Adding the repository to a project

#### 1️⃣ Inheritance (no extra steps)

If you have placed the `.github` repo at the organization level, GitHub will automatically:

- Show the issue and PR templates when contributors open a new issue/PR.
- Run any workflow files located under `.github/workflows/`.
- Apply community health files (CODE_OF_CONDUCT, CONTRIBUTING, etc.) to the repository’s UI.

> **Tip:** You can override any inherited file by adding a file with the same path in the target repo. The local file wins.

#### 2️⃣ Submodule approach

```bash
# From the root of your target repo
git submodule add https://github.com/your-org/.github .github
git commit -m "Add shared .github configuration as submodule"
git push
```

After cloning the target repo elsewhere:

```bash
git clone https://github.com/your-org/target-repo.git
cd target-repo
git submodule update --init --recursive   # pulls the .github files
```

> **Important:** GitHub Actions will still look for workflow files under `.github/workflows/` **relative to the repository root**, so you must either:
- Keep the submodule at the repository root (`.github/`), **or**
- Use a symlink (`ln -s .github/workflows .github/workflows`) (works on Linux/macOS runners).

### Enabling GitHub Actions workflows

All workflow files placed under `.github/workflows/` are automatically discovered by GitHub Actions. No extra configuration is required, but you may want to:

- **Set repository secrets** (e.g., `GH_TOKEN`, `PYPI_PASSWORD`) via the repository Settings → Secrets → Actions.
- **Adjust branch protection rules** to require status checks from the imported workflows.
- **Enable/disable specific workflows** by adding a `if:` condition in the workflow YAML or by renaming the file with a `.disabled.yml` suffix.

### Customising templates & community files

| File | Purpose | How to customise |
|------|---------|------------------|
| `ISSUE_TEMPLATE/*.md` | Pre‑filled issue forms | Edit the markdown to add fields, checklists, or default labels. |
| `PULL_REQUEST_TEMPLATE.md` | PR description scaffold | Add sections like “Motivation”, “Testing”, “Screenshots”. |
| `CODE_OF_CONDUCT.md` | Community conduct guidelines | Replace the default text with your organization’s policy. |
| `CONTRIBUTING.md` | How to contribute | Include steps for local setup, testing, and PR guidelines. |
| `SECURITY.md` | Vulnerability reporting process | Provide a contact email or a link to a bug‑bounty program. |
| `FUNDING.yml` | Sponsorship options | List GitHub Sponsors, OpenCollective, Patreon, etc. |

> **Best practice:** Keep the files **generic** enough to be useful for many projects, but add placeholders (e.g., `{{PROJECT_NAME}}`) that can be replaced via a simple script or CI step if you need project‑specific values.

---

## API Documentation

While the `.github` repo itself does not expose a public API, the **GitHub REST & GraphQL APIs** can be used to programmatically manage the resources it provides (workflows, secrets, templates, etc.). Below are the most common endpoints you’ll interact with.

### 1️⃣ Workflow Dispatch API

Trigger a workflow manually (useful for scheduled releases, nightly builds, etc.).

**Endpoint**  
`POST /repos/{owner}/{repo}/actions/workflows/{workflow_id}/dispatches`

**Parameters**

| Name | Type | Description |
|------|------|-------------|
| `owner` | string | Repository owner (org or user). |
| `repo` | string | Repository name. |
| `workflow_id` | string or integer | The workflow file name (`ci.yml`) or its numeric ID. |
| `ref` | string | The git reference (branch, tag, or SHA) to run the workflow on. |
| `inputs` | object (optional) | Map of input names to values (must match `inputs:` defined in the workflow). |

**Example (cURL)**

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token $GH_TOKEN" \
  https://api.github.com/repos/your-org/your-repo/actions/workflows/ci.yml/dispatches \
  -d '{"ref":"main","inputs":{"environment":"staging"}}'
```

### 2️⃣ Repository‑wide Settings API

Manipulate community health files, secrets, and other repo‑level settings.

| Feature | Endpoint | Method | Description |
|---------|----------|--------|-------------|
| **Create/Update a secret** | `/repos/{owner}/{repo}/actions/secrets/{secret_name}` | `PUT` | Store encrypted secrets for workflows. |
| **List workflow runs** | `/repos/{owner}/{repo}/actions/runs` | `GET` | Retrieve recent workflow executions. |
| **Get a file from the `.github` repo** | `/repos/{owner}/.github/contents/{path}` | `GET` | Fetch a template, workflow, or config file. |
| **Update repository topics** | `/repos/{owner}/{repo}` | `PATCH` |