# Hieradata Workflows

Minimal YAML validation workflow for Puppet Hieradata repositories.

## Overview

Single reusable workflow for Hieradata validation:

**`validate.yml`** - YAML syntax validation using yamllint

## Features

**Validation includes:**
- YAML syntax check using yamllint
- Configurable rules via `.yamllint` config file
- Hiera-friendly default configuration
- Strict mode option (fail on warnings)
- Custom file pattern support

## Quick Start

### 1. Copy example files to your hieradata repository

```bash
# Copy CI workflow
cp examples/hieradata/ci.yml YOUR_HIERADATA_REPO/.github/workflows/ci.yml

# Copy yamllint config (optional, but recommended)
cp examples/hieradata/.yamllint YOUR_HIERADATA_REPO/.yamllint
```

### 2. Update the workflow file

Edit `.github/workflows/ci.yml` in your hieradata repository:

```yaml
jobs:
  validate:
    uses: YOUR_ORG/github-workflows/.github/workflows/hieradata/validate.yml@main
```

Replace `YOUR_ORG/github-workflows` with the path to this repository.

### 3. Customize yamllint rules (optional)

Create or edit `.yamllint` in your repository root to customize validation rules. See `examples/hieradata/.yamllint` for a Hiera-friendly configuration.

## Workflow Inputs

```yaml
inputs:
  working-directory:
    description: 'Working directory for the hieradata repository'
    default: '.'
  yamllint-config:
    description: 'Path to yamllint config file (relative to working-directory)'
    default: ''  # Auto-detects .yamllint or .yamllint.yml
  strict:
    description: 'Fail on warnings (strict mode)'
    default: true
  file-pattern:
    description: 'File pattern to validate (e.g., "**/*.yaml" or "data/")'
    default: '.'  # Validates all YAML files
```

## Example Usage

**Minimal configuration:**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [develop, main, master]
  pull_request:
    branches: [develop, main, master]

jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/hieradata/validate.yml@main
```

**With custom options:**

```yaml
jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/hieradata/validate.yml@main
    with:
      # Only validate files in the data/ directory
      file-pattern: 'data/'
      # Use custom yamllint config
      yamllint-config: '.yamllint'
      # Allow warnings without failing
      strict: false
```

**Validate specific file patterns:**

```yaml
jobs:
  validate:
    uses: yourorg/github-workflows/.github/workflows/hieradata/validate.yml@main
    with:
      # Only validate YAML files matching pattern
      file-pattern: '**/*.yaml'
```

## yamllint Configuration

The workflow automatically detects `.yamllint` or `.yamllint.yml` in your repository. If not found, it uses Hiera-friendly defaults.

**Default configuration highlights:**
- Line length: 120 characters
- Allows `yes/no/true/false` (common in Puppet)
- Document start (`---`) not required
- Flexible comment indentation
- 2-space indentation (Puppet standard)

See `examples/hieradata/.yamllint` for the complete example configuration.

## Troubleshooting

**"YAML validation failed"**
- Check the error output for specific syntax issues
- Common issues: indentation, missing spaces after colons, tabs instead of spaces
- Test locally: `yamllint data/` or `yamllint .`

**Custom rules not applied**
- Ensure `.yamllint` or `.yamllint.yml` exists in repository root
- Or specify path with `yamllint-config` input
- Check yamllint config syntax: `yamllint -d .yamllint --print-config`

**Too strict validation**
- Set `strict: false` to allow warnings
- Or customize `.yamllint` rules to be less strict
- Disable specific rules: `rule-name: disable`

## Development Workflow

Simple workflow for hieradata repositories:

1. **Create feature branch**
   ```bash
   git checkout -b update/common-data
   ```

2. **Make changes to YAML files**
   ```bash
   # Edit your hieradata files
   vim data/common.yaml
   ```

3. **Test locally (optional)**
   ```bash
   yamllint data/
   ```

4. **Push and create PR**
   ```bash
   git add data/common.yaml
   git commit -m "Update common data"
   git push origin update/common-data
   # Create PR - validation runs automatically
   ```

5. **Merge to main**
   - Validation runs on merge
   - Deploy hieradata (manual or automated)
