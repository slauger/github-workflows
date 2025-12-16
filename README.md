# Reusable GitHub Workflows

This repository contains reusable GitHub Actions workflows for Puppet infrastructure projects.

## Available Workflows

### Puppet Module

Automated validation and release workflows for Puppet modules with semantic versioning.

- **Workflows:** `validate.yml`, `release.yml`
- **Features:** YAML/JSON/Puppet lint, syntax checks, semantic-release, CHANGELOG generation
- **[Documentation](.github/workflows/puppet-module/README.md)**
- **[Examples](examples/puppet-module/)**

### Hieradata

Minimal YAML validation for Puppet Hieradata repositories.

- **Workflow:** `validate.yml`
- **Features:** yamllint with Hiera-friendly defaults
- **[Documentation](.github/workflows/hieradata/README.md)**
- **[Examples](examples/hieradata/)**

### Control Repository

Validation for Puppet Control Repositories (R10K/Code Manager).

- **Workflow:** `validate.yml`
- **Features:** Puppetfile syntax, hiera.yaml validation, site modules validation
- **[Documentation](.github/workflows/control-repo/README.md)**
- **[Examples](examples/control-repo/)**

## Quick Start

1. Choose the workflow type for your repository
2. Copy the example CI workflow from `examples/` to your repository
3. Update the workflow reference to point to this repository
4. Follow the specific documentation for your workflow type

## Repository Structure

```
.
├── .github/workflows/
│   ├── puppet-module-validate.yml       # Module validation
│   ├── puppet-module-release.yml        # Semantic release
│   ├── hieradata-validate.yml           # YAML validation
│   ├── control-repo-validate.yml        # Control-repo validation
│   ├── puppet-module/
│   │   └── README.md                    # Documentation
│   ├── hieradata/
│   │   └── README.md                    # Documentation
│   └── control-repo/
│       └── README.md                    # Documentation
│
├── examples/
│   ├── puppet-module/
│   │   ├── ci.yml                       # Example workflow
│   │   └── .releaserc.yml               # Semantic-release config
│   ├── hieradata/
│   │   ├── ci.yml                       # Example workflow
│   │   └── .yamllint                    # yamllint config
│   └── control-repo/
│       └── ci.yml                       # Example workflow
│
└── README.md                            # This file
```

## Usage Example

```yaml
# In your repository: .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  validate:
    uses: YOUR_ORG/github-workflows/.github/workflows/WORKFLOW_NAME.yml@main
```

Replace:
- `YOUR_ORG/github-workflows` with your repository path (e.g., `slauger/github-workflows`)
- `WORKFLOW_NAME` with one of:
  - `puppet-module-validate`, `puppet-module-release`
  - `hieradata-validate`
  - `control-repo-validate`

## Contributing

Contributions welcome! Please ensure:
- Workflows are tested
- Documentation is updated
- Examples are provided

## License

MIT
