# Changelog

All notable changes to the Trivy Security Scan action will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-08-22

### Added
- Initial release of the Trivy Security Scan custom GitHub Action
- Comprehensive Docker image vulnerability scanning using Trivy
- Multiple output formats (table and JSON)
- Automatic artifact upload with configurable retention
- PR comment integration with vulnerability summaries
- Flexible configuration options for severity levels and scanning parameters
- Non-blocking scan execution (configurable exit codes)
- Detailed documentation and usage examples

### Features
- **Security Scanning**: Complete vulnerability scanning for Docker images
- **Report Generation**: Table format for humans, JSON for automation
- **PR Integration**: Automatic commenting on pull requests with scan results
- **Artifact Management**: Automatic upload of scan results with configurable retention
- **Flexible Configuration**: Customizable severity levels, output formats, and behaviors
- **Multi-format Outputs**: Status, vulnerability count, and artifact ID outputs for workflow integration

### Inputs
- `image-ref`: Docker image reference to scan (required)
- `registry`: Container registry URL (default: ghcr.io)
- `severity`: Severity levels for scanning (default: CRITICAL,HIGH)
- `detailed-severity`: Severity levels for detailed reports (default: CRITICAL,HIGH,MEDIUM,LOW)
- `ignore-unfixed`: Ignore vulnerabilities without fixes (default: true)
- `exit-code`: Exit code when vulnerabilities found (default: 0)
- `artifact-name`: Artifact name for scan results (default: trivy-scan-results)
- `artifact-retention-days`: Artifact retention period (default: 30)
- `github-token`: GitHub token for PR comments (default: github.token)
- `post-pr-comment`: Enable/disable PR commenting (default: true)

### Outputs
- `scan-status`: Scan result status (success/vulnerabilities_found)
- `vulnerability-count`: Total vulnerabilities found
- `artifact-id`: Uploaded artifact ID

### Documentation
- Comprehensive README with usage examples
- Detailed usage guide with advanced scenarios
- Example workflows for different use cases
- Test workflow for action validation
