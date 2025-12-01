# Trivy Security Scanner Action

A GitHub Action that wraps [Trivy](https://github.com/aquasecurity/trivy) security scanner to generate SARIF and SBOM reports for your repositories.

## Features

- 🔍 Run Trivy security scans on filesystems, container images, repositories, and configuration files
- 📊 Generate SARIF reports for GitHub Code Scanning integration
- 📦 Generate SBOM (Software Bill of Materials) reports in multiple formats
- ⚙️ Highly configurable with sensible defaults
- 🚀 Easy to integrate into existing workflows

## Usage

### Basic Usage

```yaml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy Scanner
        uses: your-username/trivy-scanner-action@v1

      - name: Upload SARIF to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-results.sarif
```

### Scan Container Image

```yaml
- name: Run Trivy Scanner on Docker Image
  uses: your-username/trivy-scanner-action@v1
  with:
    scan-type: 'image'
    scan-target: 'nginx:latest'
    severity: 'HIGH,CRITICAL'
```

### Custom Output Paths

```yaml
- name: Run Trivy Scanner
  uses: your-username/trivy-scanner-action@v1
  with:
    output-sarif: 'reports/security-scan.sarif'
    output-sbom: 'reports/sbom.json'
    sbom-format: 'spdx-json'
```

### Scan Specific Directory

```yaml
- name: Run Trivy Scanner on Source Code
  uses: your-username/trivy-scanner-action@v1
  with:
    scan-type: 'fs'
    scan-target: './src'
    skip-dirs: 'test,node_modules'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `scan-type` | Type of scan to perform (`fs`, `image`, `repo`, `config`) | No | `fs` |
| `scan-target` | Target to scan (path, image name, etc.) | No | `.` |
| `severity` | Severities to include (comma-separated) | No | `UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL` |
| `format` | Output format for SARIF report | No | `sarif` |
| `output-sarif` | Path to save SARIF report | No | `trivy-results.sarif` |
| `output-sbom` | Path to save SBOM report | No | `trivy-sbom.json` |
| `sbom-format` | SBOM format (`cyclonedx`, `spdx`, `spdx-json`) | No | `cyclonedx` |
| `trivy-version` | Trivy version to use | No | `latest` |
| `skip-dirs` | Comma-separated list of directories to skip | No | `""` |
| `timeout` | Scan timeout duration | No | `5m` |

## Outputs

| Output | Description |
|--------|-------------|
| `sarif-report` | Path to the generated SARIF report |
| `sbom-report` | Path to the generated SBOM report |

## Advanced Examples

### Full Security Workflow with Artifact Upload

```yaml
name: Complete Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  security-scan:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy Scanner
        id: trivy
        uses: your-username/trivy-scanner-action@v1
        with:
          severity: 'MEDIUM,HIGH,CRITICAL'
          output-sarif: 'trivy-results.sarif'
          output-sbom: 'sbom.cyclonedx.json'
          sbom-format: 'cyclonedx'
          timeout: '10m'

      - name: Upload SARIF to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: ${{ steps.trivy.outputs.sarif-report }}

      - name: Upload Reports as Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: security-reports
          path: |
            ${{ steps.trivy.outputs.sarif-report }}
            ${{ steps.trivy.outputs.sbom-report }}
```

### Multi-Target Scanning

```yaml
jobs:
  scan-multiple-targets:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        target:
          - { type: 'fs', path: '.', name: 'filesystem' }
          - { type: 'config', path: '.', name: 'config' }
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy Scanner - ${{ matrix.target.name }}
        uses: your-username/trivy-scanner-action@v1
        with:
          scan-type: ${{ matrix.target.type }}
          scan-target: ${{ matrix.target.path }}
          output-sarif: trivy-${{ matrix.target.name }}.sarif
          output-sbom: sbom-${{ matrix.target.name }}.json
```

## Permissions Required

For uploading SARIF results to GitHub Security, your workflow needs:

```yaml
permissions:
  security-events: write
  contents: read
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

If you encounter any issues or have questions, please file an issue on the GitHub repository.
