# Trivy Security Scan Action - Usage Guide

This guide provides comprehensive examples of how to use the Trivy Security Scan custom action in different scenarios.

## Table of Contents

- [Quick Start](#quick-start)
- [Basic Usage](#basic-usage)
- [Advanced Configurations](#advanced-configurations)
- [Integration Patterns](#integration-patterns)
- [Security Gates](#security-gates)
- [Multi-Repository Usage](#multi-repository-usage)
- [Troubleshooting](#troubleshooting)

## Quick Start

Add this step to your workflow after building your Docker image:

```yaml
- name: Security Scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: 'ghcr.io/${{ github.repository }}:latest'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Basic Usage

### Minimal Configuration

```yaml
steps:
  # ... build your Docker image ...
  
  - name: Security Scan
    uses: ./.github/actions/trivy-security-scan
    with:
      image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
```

### With Custom Settings

```yaml
- name: Security Scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
    severity: 'CRITICAL,HIGH,MEDIUM'
    ignore-unfixed: 'false'
    artifact-retention-days: '60'
    post-pr-comment: 'true'
```

## Advanced Configurations

### Scan Multiple Images

```yaml
strategy:
  matrix:
    image: ['app', 'worker', 'api']
    
steps:
  - name: Security Scan - ${{ matrix.image }}
    uses: ./.github/actions/trivy-security-scan
    with:
      image-ref: '${{ env.REGISTRY }}/${{ github.repository }}-${{ matrix.image }}:latest'
      artifact-name: 'security-scan-${{ matrix.image }}'
```

### Conditional Scanning

```yaml
- name: Security Scan
  if: github.event_name == 'pull_request' || github.ref == 'refs/heads/main'
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
    severity: ${{ github.ref == 'refs/heads/main' && 'CRITICAL,HIGH,MEDIUM' || 'CRITICAL,HIGH' }}
```

### Custom Artifact Names

```yaml
- name: Security Scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
    artifact-name: 'security-report-${{ github.event_name }}-${{ github.run_number }}'
```

## Integration Patterns

### Complete CI/CD Pipeline

```yaml
name: Complete CI/CD with Security

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  security-pipeline:
    runs-on: ubuntu-latest
    outputs:
      scan-status: ${{ steps.security-scan.outputs.scan-status }}
      vulnerability-count: ${{ steps.security-scan.outputs.vulnerability-count }}
      
    steps:
    - uses: actions/checkout@v4
    
    - name: Build Image
      run: docker build -t myapp:latest .
    
    - name: Security Scan
      id: security-scan
      uses: ./.github/actions/trivy-security-scan
      with:
        image-ref: 'myapp:latest'
        
  deploy:
    needs: security-pipeline
    if: needs.security-pipeline.outputs.scan-status == 'success'
    runs-on: ubuntu-latest
    steps:
    - name: Deploy
      run: echo "Deploying secure image..."
```

### With Slack Notifications

```yaml
- name: Security Scan
  id: scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'

- name: Notify Slack on Vulnerabilities
  if: steps.scan.outputs.scan-status == 'vulnerabilities_found'
  uses: 8398a7/action-slack@v3
  with:
    status: custom
    custom_payload: |
      {
        text: "🚨 Security vulnerabilities found in ${{ github.repository }}",
        attachments: [{
          color: 'warning',
          fields: [{
            title: 'Vulnerabilities Found',
            value: '${{ steps.scan.outputs.vulnerability-count }}',
            short: true
          }, {
            title: 'Repository',
            value: '${{ github.repository }}',
            short: true
          }]
        }]
      }
```

## Security Gates

### Fail Build on Critical Vulnerabilities

```yaml
- name: Security Scan
  id: scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
    exit-code: '1'  # This will make Trivy exit with code 1 if vulnerabilities are found

- name: Security Gate
  if: steps.scan.outputs.scan-status == 'vulnerabilities_found'
  run: |
    echo "❌ Security gate failed - vulnerabilities detected!"
    echo "Found ${{ steps.scan.outputs.vulnerability-count }} vulnerabilities"
    exit 1
```

### Vulnerability Threshold

```yaml
- name: Security Scan
  id: scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'

- name: Enforce Security Threshold
  run: |
    VULN_COUNT=${{ steps.scan.outputs.vulnerability-count }}
    MAX_ALLOWED=5
    
    if [ "$VULN_COUNT" -gt "$MAX_ALLOWED" ]; then
      echo "❌ Too many vulnerabilities: $VULN_COUNT (max allowed: $MAX_ALLOWED)"
      exit 1
    else
      echo "✅ Vulnerability count within acceptable threshold: $VULN_COUNT"
    fi
```

### Environment-Specific Gates

```yaml
- name: Security Scan
  id: scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'

- name: Production Security Gate
  if: github.ref == 'refs/heads/main' && steps.scan.outputs.scan-status == 'vulnerabilities_found'
  run: |
    echo "❌ Cannot deploy to production with security vulnerabilities!"
    echo "Please fix the following issues before deploying:"
    echo "- Vulnerabilities found: ${{ steps.scan.outputs.vulnerability-count }}"
    exit 1

- name: Staging Deployment (Allow with Warnings)
  if: github.ref == 'refs/heads/develop'
  run: |
    if [ "${{ steps.scan.outputs.scan-status }}" = "vulnerabilities_found" ]; then
      echo "⚠️ Deploying to staging with ${{ steps.scan.outputs.vulnerability-count }} vulnerabilities"
      echo "Please address these issues before promoting to production"
    fi
    echo "Deploying to staging..."
```

## Multi-Repository Usage

### Using from External Repository

```yaml
- name: Security Scan
  uses: smartdatafoundry/trivy-security-scan/.github/actions/trivy-security-scan@v1.0.0
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Pinning to Specific Version

```yaml
- name: Security Scan
  uses: smartdatafoundry/income_volatility_pipeline/.github/actions/trivy-security-scan@v1.0.0
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
```

### With Custom Registry

```yaml
- name: Security Scan
  uses: smartdatafoundry/trivy-security-scan/.github/actions/trivy-security-scan@v1.0.0
  with:
    image-ref: 'my-registry.com/${{ github.repository }}:latest'
    registry: 'my-registry.com'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Troubleshooting

### Common Issues and Solutions

#### Issue: Action Not Found
```yaml
# ❌ Wrong format
uses: smartdatafoundry/trivy-security-scan/trivy-security-scan@main

# ✅ Correct format
uses: smartdatafoundry/trivy-security-scan/.github/actions/trivy-security-scan@v1.0.0
```

#### Issue: Permissions Error
```yaml
# Add these permissions to your job
permissions:
  contents: read
  packages: read
  pull-requests: write
  actions: write
```

#### Issue: Image Not Found
```yaml
# Ensure you're logged in before scanning
- name: Login to Registry
  uses: docker/login-action@v3
  with:
    registry: ${{ env.REGISTRY }}
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

# Then run the scan
- name: Security Scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
```

#### Issue: PR Comments Not Working
```yaml
# Ensure you have the right permissions and token
- name: Security Scan
  uses: ./.github/actions/trivy-security-scan
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
    github-token: ${{ secrets.GITHUB_TOKEN }}  # Make sure this is set
    post-pr-comment: 'true'
```

### Debug Mode

```yaml
- name: Security Scan (Debug)
  uses: ./.github/actions/trivy-security-scan
  env:
    ACTIONS_STEP_DEBUG: true
  with:
    image-ref: '${{ env.REGISTRY }}/${{ github.repository }}:latest'
```

### Custom Trivy Version

If you need a specific version of Trivy, you can fork the action and modify the `aquasecurity/trivy-action@0.32.0` reference to your desired version.

## Best Practices

1. **Always scan on PRs**: Include security scanning in your PR workflow to catch issues early
2. **Use appropriate severity levels**: Adjust severity based on your environment (stricter for production)
3. **Set up security gates**: Prevent deployment of vulnerable images to production
4. **Monitor scan results**: Use the outputs to integrate with monitoring and alerting systems
5. **Keep artifacts**: Store scan results for compliance and historical analysis
6. **Regular updates**: Keep the action updated to get the latest security database

## Examples Repository

For more examples and templates, check the `examples/` directory in this repository.
