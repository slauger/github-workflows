# Control Repository Workflows

Validation workflow for Puppet Control Repositories (R10K/Code Manager).

## Overview

Single reusable workflow for Control Repository validation:

**`validate.yml`** - Validates Puppetfile, Hiera config, and site modules

## Features

**Validation includes:**
- Control repository structure check
- Puppetfile syntax check (Ruby syntax)
- hiera.yaml validation (YAML lint)
- Puppet syntax check for site modules
- puppet-lint for site modules (style and best practices)

## Quick Start

### 1. Copy example workflow to your control repository

```bash
# Copy CI workflow
cp examples/control-repo/ci.yml YOUR_CONTROL_REPO/.github/workflows/ci.yml
```

### 2. Update the workflow file

Edit `.github/workflows/ci.yml` in your control repository:

```yaml
jobs:
  validate:
    uses: YOUR_ORG/github-workflows/.github/workflows/control-repo/validate.yml@main
```

Replace `YOUR_ORG/github-workflows` with the path to this repository.

### 3. Adjust paths if needed

If your control repository uses non-standard paths, configure them:

```yaml
jobs:
  validate:
    uses: YOUR_ORG/github-workflows/.github/workflows/control-repo/validate.yml@main
    with:
      site-modules-path: "site"        # Path to site modules
      puppetfile-path: "Puppetfile"    # Path to Puppetfile
      hiera-config-path: "hiera.yaml"  # Path to hiera.yaml
```

## Workflow Inputs

```yaml
inputs:
  working-directory:
    description: 'Working directory for the control repository'
    default: '.'
  puppet-version:
    description: 'Puppet version to use for validation'
    default: '7'
  ruby-version:
    description: 'Ruby version to use'
    default: '3.1'
  site-modules-path:
    description: 'Path to site modules directory'
    default: 'site'
  puppetfile-path:
    description: 'Path to Puppetfile'
    default: 'Puppetfile'
  hiera-config-path:
    description: 'Path to hiera.yaml'
    default: 'hiera.yaml'
  yamllint-config:
    description: 'Path to yamllint config file'
    default: ''  # Auto-detects .yamllint
  strict-puppet-lint:
    description: 'Fail on puppet-lint warnings'
    default: false
```

## Example Usage

**Minimal configuration:**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [production, development, main]
  pull_request:
    branches: [production, development, main]

jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/control-repo/validate.yml@main
```

**With custom options:**

```yaml
jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/control-repo/validate.yml@main
    with:
      puppet-version: "8"
      ruby-version: "3.2"
      site-modules-path: "site-modules"
      strict-puppet-lint: true  # Fail on lint warnings
```

**Custom control-repo structure:**

```yaml
jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/control-repo/validate.yml@main
    with:
      site-modules-path: "dist/site"
      puppetfile-path: "Puppetfile"
      hiera-config-path: "config/hiera.yaml"
      yamllint-config: ".yamllint"
```

## Control Repository Structure

Expected structure (standard R10K layout):

```
control-repo/
├── Puppetfile              # R10K module dependencies
├── hiera.yaml              # Hiera configuration
├── environment.conf        # Environment configuration (optional)
├── site/                   # Site-specific modules
│   ├── profile/           # Profile modules
│   │   └── manifests/
│   └── role/              # Role modules
│       └── manifests/
└── data/                   # Hiera data (optional, can be in hieradata repo)
    └── common.yaml
```

## Validation Details

**Puppetfile Syntax Check:**
- Validates Ruby syntax of Puppetfile
- Does NOT resolve or fetch modules (works on public runners)
- Checks for syntax errors, missing quotes, etc.

**hiera.yaml Validation:**
- YAML syntax check using yamllint
- Uses custom `.yamllint` config if present
- Falls back to sensible defaults

**Site Modules Validation:**
- Puppet syntax check for all `.pp` files
- puppet-lint for style and best practices
- Optional strict mode (fail on warnings)

## Troubleshooting

**"Puppetfile syntax check failed"**
- Check for Ruby syntax errors in Puppetfile
- Common issues: missing commas, unmatched quotes, missing `mod` keyword
- Test locally: `ruby -c Puppetfile`

**"Puppet syntax error"**
- Check site module manifests for syntax errors
- Common issues: missing commas, unmatched quotes, typos
- Test locally: `puppet parser validate site/profile/manifests/myprofile.pp`

**"puppet-lint warnings"**
- Fix style issues reported by puppet-lint
- Many can be auto-fixed: `puppet-lint --fix site/`
- Or set `strict-puppet-lint: false` to allow warnings

**"hiera.yaml validation failed"**
- Check YAML syntax and indentation
- Ensure proper spacing (spaces, not tabs)
- Test locally: `yamllint hiera.yaml`

## Development Workflow

Typical workflow for control repository development:

1. **Create feature branch**
   ```bash
   git checkout -b feature/add-profile
   ```

2. **Make changes**
   ```bash
   # Add new profile module
   mkdir -p site/profile/manifests
   vim site/profile/manifests/myapp.pp

   # Update Puppetfile if needed
   vim Puppetfile
   ```

3. **Test locally (optional)**
   ```bash
   # Check Puppetfile syntax
   ruby -c Puppetfile

   # Check Puppet syntax
   puppet parser validate site/profile/manifests/myapp.pp

   # Run puppet-lint
   puppet-lint site/
   ```

4. **Push and create PR**
   ```bash
   git add .
   git commit -m "feat(profile): add myapp profile"
   git push origin feature/add-profile
   # Create PR - validation runs automatically
   ```

5. **Merge to production/main**
   - Validation runs on merge
   - R10K/Code Manager deploys changes (external to this workflow)

## Best Practices

**Branch Strategy:**
- `production` - production environment (maps to Puppet production env)
- `development` - development/testing environment
- `feature/*` - feature branches (create dynamic environments if needed)

**Site Modules:**
- Keep role/profile pattern in `site/` directory
- Use roles for node classification
- Use profiles for reusable components
- Avoid business logic in roles

**Puppetfile:**
- Pin module versions (don't use `:latest`)
- Use git refs or tags for internal modules
- Comment module purposes
- Group related modules

**Hiera:**
- Consider separating hieradata into dedicated repository
- Use hierarchy that matches your infrastructure
- Keep sensitive data in separate backend (eyaml, Vault, etc.)
