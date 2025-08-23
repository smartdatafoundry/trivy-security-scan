# Trivy Security Scan Action

A reusable GitHub Action for comprehensive Docker image security scanning using Trivy. This action performs vulnerability scanning, generates detailed reports, uploads artifacts, and posts results as PR comments.

## Features

- **Comprehensive Scanning**: Scans for vulnerabilities using Trivy with configurable severity levels
- **Multiple Report Formats**: Generates both table and JSON format reports for different use cases
- **Artifact Upload**: Automatically uploads scan results as workflow artifacts with configurable retention
- **PR Comments**: Posts concise security scan results as comments on pull requests
- **Flexible Configuration**: Highly configurable with sensible defaults
- **Non-blocking**: Doesn't fail workflows by default, allowing builds to continue while flagging security issues

## Usage

### Basic Usage

```yaml
- name: Security Scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: 'ghcr.io/${{ github.repository }}:latest'
```

### Advanced Usage

```yaml
- name: Security Scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: 'ghcr.io/${{ github.repository }}:latest'
    registry: 'ghcr.io'
    severity: 'CRITICAL,HIGH'
    detailed-severity: 'CRITICAL,HIGH,MEDIUM,LOW'
    ignore-unfixed: 'true'
    exit-code: '0'
    artifact-name: 'security-scan-results'
    artifact-retention-days: '30'
    github-token: ${{ secrets.GITHUB_TOKEN }}
    post-pr-comment: 'true'
```

### Using in External Repositories

To use this action in other repositories, reference it using the GitHub repository format:

```yaml
- name: Security Scan
  uses: smartdatafoundry/trivy-security-scan/.github/actions/trivy-security-scan@v1.0.0
  with:
    image-ref: 'ghcr.io/${{ github.repository }}:latest'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `image-ref` | Docker image reference to scan (e.g., ghcr.io/user/repo:tag) | ✅ | - |
| `registry` | Container registry URL | ❌ | `ghcr.io` |
| `severity` | Comma-separated list of severities to scan for | ❌ | `CRITICAL,HIGH` |
| `detailed-severity` | Comma-separated list of severities for detailed JSON report | ❌ | `CRITICAL,HIGH,MEDIUM,LOW` |
| `ignore-unfixed` | Ignore vulnerabilities with no available fix | ❌ | `true` |
| `exit-code` | Exit code when vulnerabilities are found | ❌ | `0` |
| `artifact-name` | Name for the artifact containing scan results | ❌ | `trivy-scan-results` |
| `artifact-retention-days` | Number of days to retain the scan results artifact | ❌ | `30` |
| `github-token` | GitHub token for commenting on PRs | ❌ | `${{ github.token }}` |
| `post-pr-comment` | Whether to post scan results as PR comment | ❌ | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `scan-status` | Status of the security scan (`success` or `vulnerabilities_found`) |
| `vulnerability-count` | Total number of vulnerabilities found |
| `artifact-id` | ID of the uploaded artifact containing scan results |

## Example Workflows

### Complete Docker Build and Scan Workflow

```yaml
name: Docker Build and Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io

jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      pull-requests: write
      
    steps:
    - uses: actions/checkout@v4
    
    - name: Build Docker image
      run: |
        docker build . -t ${{ env.REGISTRY }}/${{ github.repository }}:latest
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Security Scan
      id: security-scan
      uses: ./.github/actions/trivy-security-scan
      with:
        image-ref: ${{ env.REGISTRY }}/${{ github.repository }}:latest
        github-token: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Handle scan results
      run: |
        echo "Scan status: ${{ steps.security-scan.outputs.scan-status }}"
        echo "Vulnerabilities found: ${{ steps.security-scan.outputs.vulnerability-count }}"
        
        if [ "${{ steps.security-scan.outputs.scan-status }}" = "vulnerabilities_found" ]; then
          echo "⚠️ Security vulnerabilities detected!"
          echo "Check the artifacts for detailed reports."
        fi
    
    - name: Push Docker image
      if: github.ref == 'refs/heads/main'
      run: |
        docker push ${{ env.REGISTRY }}/${{ github.repository }}:latest
```

### Conditional Scanning

```yaml
- name: Security Scan
  if: github.event_name == 'pull_request' || github.ref == 'refs/heads/main'
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: ${{ env.REGISTRY }}/${{ github.repository }}:latest
    severity: 'CRITICAL,HIGH'
```

### Fail on Vulnerabilities

```yaml
- name: Security Scan (Strict)
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: ${{ env.REGISTRY }}/${{ github.repository }}:latest
    exit-code: '1'  # Fail if vulnerabilities are found

- name: Check scan results
  run: |
    if [ "${{ steps.security-scan.outputs.scan-status }}" = "vulnerabilities_found" ]; then
      echo "❌ Build failed due to security vulnerabilities!"
      exit 1
    fi
```

## Generated Reports

The action generates several types of reports:

### 1. Table Format Report (`trivy-scan-table.txt`)
Human-readable table format showing vulnerabilities with basic information.

### 2. Detailed JSON Report (`trivy-scan-detailed.json`)
Machine-readable JSON format with comprehensive vulnerability details.

### 3. Summary Report (`scan-summary.md`)
Markdown-formatted summary with image metadata and scan results.

### 4. PR Comment (`pr-comment.md`)
Concise markdown report optimized for GitHub PR comments, including:
- Scan status and vulnerability count
- Top 10 critical/high severity vulnerabilities
- Scan metadata and links to detailed reports

## Artifact Contents

The uploaded artifact contains:
- `trivy-scan-table.txt` - Table format scan results
- `trivy-scan-detailed.json` - Detailed JSON scan results
- `scan-summary.md` - Human-readable summary report
- `pr-comment.md` - PR comment content

## Permissions Required

When using this action, ensure your workflow has the following permissions:

```yaml
permissions:
  contents: read          # For checking out code
  packages: read          # For pulling Docker images (if from same registry)
  pull-requests: write    # For commenting on PRs
  actions: write          # For uploading artifacts
```

## Troubleshooting

### Action Not Found
When referencing the action from external repositories, ensure:
- The repository is public, or you have appropriate access permissions
- The reference format is correct: `owner/repo/.github/actions/action-name@ref`
- The branch or tag reference exists

### Docker Image Not Found
Ensure the image reference is correct and accessible:
- The image exists in the specified registry
- Your workflow has appropriate permissions to access the image
- For private registries, ensure you're logged in before scanning

### PR Comments Not Posted
Check that your workflow has the `pull-requests: write` permission and that the `github-token` input has the necessary permissions.

## Contributing

This action is part of the Smart Data Foundry infrastructure toolkit. To contribute improvements or report issues, please use the main repository's issue tracker.

## License

This action is provided under the same license as the parent repository.
