# Puppet Module Workflows

Automated validation and release workflows for Puppet modules using semantic-release and Conventional Commits.

## Overview

Two reusable workflows for Puppet module CI/CD:

1. **`validate.yml`** - Comprehensive validation (linting, syntax, tests)
2. **`release.yml`** - Automated semantic versioning and releases

## Workflow Strategy

```
develop branch:
  ├─ Pull Request → validate runs
  └─ Push → validate runs

main/master branch (merge from develop):
  ├─ validate runs (must pass)
  └─ release runs (if validate passes)
      ├─ Analyzes Conventional Commits
      ├─ Determines version (major.minor.patch)
      ├─ Updates metadata.json
      ├─ Generates/updates CHANGELOG.md
      ├─ Creates GitHub Release
      └─ Commits changes back to main
```

## Features

**Validation includes:**
- YAML Lint
- JSON Lint (metadata.json validation)
- Markdown Lint
- Puppet syntax check (`puppet parser validate`)
- puppet-lint (style and best practices)
- PDK validation (optional, recommended)
- **Conventional Commits format check** (required for semantic-release)

**Release automation:**
- Semantic versioning based on Conventional Commits
- Automatic version bump in `metadata.json`
- CHANGELOG.md generation and updates
- GitHub Release creation with release notes
- Git tagging

## Prerequisites

**Your Puppet module repository must:**

1. Use Conventional Commits format for all commits
2. Have a `metadata.json` file (standard Puppet module metadata)
3. Have a `.releaserc.yml` semantic-release configuration
4. Use `develop` and `main` (or `master`) branches

## Quick Start

### 1. Copy example files to your Puppet module repository

```bash
# Copy CI/CD workflow
cp examples/puppet-module/ci.yml YOUR_MODULE/.github/workflows/ci.yml

# Copy semantic-release config
cp examples/puppet-module/.releaserc.yml YOUR_MODULE/.releaserc.yml
```

### 2. Update the workflow file

Edit `.github/workflows/ci.yml` in your module repository:

```yaml
jobs:
  validate:
    uses: YOUR_ORG/github-workflows/.github/workflows/puppet-module/validate.yml@main
    # ...

  release:
    uses: YOUR_ORG/github-workflows/.github/workflows/puppet-module/release.yml@main
    # ...
```

Replace `YOUR_ORG/github-workflows` with the path to this repository.

### 3. Ensure Conventional Commits

All commits must follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Examples:**
```
feat(init): add initial module structure
fix(config): correct parameter validation
docs(readme): update installation instructions
refactor(manifests): simplify class structure
feat(api)!: change default behavior (BREAKING CHANGE)
```

**Valid types:**
- `feat`: New feature (triggers minor version bump)
- `fix`: Bug fix (triggers patch version bump)
- `docs`: Documentation only
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `build`: Build system changes
- `ci`: CI/CD changes
- `chore`: Other changes (dependencies, etc.)
- `revert`: Revert previous commit

**Breaking changes** (triggers major version bump):
- Add `!` after type: `feat!: breaking change`
- Or include `BREAKING CHANGE:` in commit body

## Workflow Inputs

### validate.yml

```yaml
inputs:
  puppet-version:
    description: 'Puppet version to use for validation'
    default: '7'
  ruby-version:
    description: 'Ruby version to use'
    default: '3.1'
  pdk-version:
    description: 'PDK version to use'
    default: 'latest'
  skip-pdk:
    description: 'Skip PDK validation (use individual tools instead)'
    default: false
  working-directory:
    description: 'Working directory for the module'
    default: '.'
```

### release.yml

```yaml
inputs:
  working-directory:
    description: 'Working directory for the module'
    default: '.'
  node-version:
    description: 'Node.js version for semantic-release'
    default: '20'
  dry-run:
    description: 'Run semantic-release in dry-run mode'
    default: false
  puppet-version:
    description: 'Puppet version to use for validation'
    default: '7'
  ruby-version:
    description: 'Ruby version to use'
    default: '3.1'

secrets:
  github-token:
    description: 'GitHub token for creating releases'
    required: true
```

## Semantic Release Configuration

The `.releaserc.yml` configures semantic-release behavior. See `examples/puppet-module/.releaserc.yml` for the recommended configuration.

**Plugins used:**
- `@semantic-release/commit-analyzer` - Analyzes commits
- `@semantic-release/release-notes-generator` - Generates release notes
- `@semantic-release/changelog` - Updates CHANGELOG.md
- `@google/semantic-release-replace-plugin` - Updates metadata.json version
- `@semantic-release/git` - Commits changes back to repository
- `@semantic-release/github` - Creates GitHub Releases

## Release Rules

| Commit Type | Version Bump | Example |
|-------------|--------------|---------|
| `feat:` | minor (0.1.0 → 0.2.0) | New feature |
| `fix:` | patch (0.1.0 → 0.1.1) | Bug fix |
| `perf:` | patch | Performance improvement |
| `revert:` | patch | Revert previous change |
| BREAKING CHANGE | major (0.1.0 → 1.0.0) | Breaking change |
| `docs:`, `style:`, `test:`, `ci:`, `chore:` | none | No release |

## Example Usage

**Minimal configuration:**

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]

jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/puppet-module/validate.yml@main

  release:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: yourorg/github-workflows/.github/workflows/puppet-module/release.yml@main
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

**With custom options:**

```yaml
jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/puppet-module/validate.yml@main
    with:
      puppet-version: "8"
      ruby-version: "3.2"
      skip-pdk: false

  release:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: yourorg/github-workflows/.github/workflows/puppet-module/release.yml@main
    with:
      puppet-version: "8"
      ruby-version: "3.2"
      node-version: "20"
      dry-run: false  # Set to true to test without actual releases
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Testing Releases

Use dry-run mode to test semantic-release without creating actual releases:

```yaml
release:
  uses: yourorg/github-workflows/.github/workflows/puppet-module/release.yml@main
  with:
    dry-run: true
  secrets:
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Troubleshooting

**"No semantic-release config found"**
- Ensure `.releaserc.yml` exists in your repository root
- Copy from `examples/puppet-module/.releaserc.yml`

**"Invalid commit message" error**
- All commits must follow Conventional Commits format
- Check your commit history: `git log --oneline`
- Rewrite commits if needed (before pushing): `git rebase -i`

**No release created**
- Check if commits warrant a release (feat/fix/BREAKING CHANGE)
- Commits like `docs:`, `chore:`, `ci:` don't trigger releases
- Check semantic-release output in GitHub Actions logs

**Version not updated in metadata.json**
- Verify `.releaserc.yml` includes `@google/semantic-release-replace-plugin`
- Check the regex pattern matches your metadata.json format
- Look for semantic-release errors in Actions logs

## Development Workflow

Recommended workflow for Puppet module development:

1. **Create feature branch from develop**
   ```bash
   git checkout develop
   git checkout -b feat/my-new-feature
   ```

2. **Make changes with Conventional Commits**
   ```bash
   git commit -m "feat(init): add new parameter for X"
   git commit -m "test(init): add tests for new parameter"
   git commit -m "docs(readme): document new parameter"
   ```

3. **Push and create PR to develop**
   ```bash
   git push origin feat/my-new-feature
   # Create PR to develop branch
   # Validation workflow runs automatically
   ```

4. **Merge to develop**
   - Validation runs on develop branch
   - Continue development/testing

5. **Merge develop to main**
   ```bash
   git checkout main
   git merge develop
   git push origin main
   # Validation runs, then release runs automatically
   # semantic-release creates new version, updates files, creates GitHub Release
   ```
