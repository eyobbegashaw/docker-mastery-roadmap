 

### Core Concept
Image scanning identifies vulnerabilities in OS packages, language dependencies, and application binaries. Scanners compare image contents against vulnerability databases (NVD, Alpine SecDB, GitHub Advisory). Critical for compliance (PCI DSS, HIPAA) and supply chain security. Modern scanning includes secrets detection, malware scanning, and policy enforcement.

### Key Terminologies
- **CVE**: Common Vulnerabilities and Exposures (standardized identifier)
- **CVSS Score**: Severity rating (0-10) for vulnerabilities
- **Fix Version**: Package version patching the vulnerability
- **SBOM**: Software Bill of Materials (CycloneDX, SPDX formats)
- **Base Image**: Parent image (vulnerabilities inherit to child)
- **Policy as Code**: Automated compliance rules (Rego, OPA)

### How it Works
Scanning process:
1. Extract all installed packages (dpkg, rpm, apk, pip, npm, gem)
2. Query vulnerability database for each package + version
3. Match CVEs to packages
4. Calculate fix availability and severity
5. Generate report (JSON, SARIF, HTML)

Database sources:
- **OS**: Alpine SecDB, Debian Security Tracker, Ubuntu CVE Tracker
- **Language**: NVD, GitHub Security Advisory, Sonatype OSS Index
- **Proprietary**: Snyk, Docker Scout, Trivy DB (aggregated)

### CLI Commands/Syntax
```bash
# Trivy (most comprehensive OSS scanner)
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity CRITICAL,HIGH --exit-code 1 myapp:latest
trivy image --format sarif --output report.sarif myapp:latest
trivy image --ignore-unfixed --vuln-type os,library myapp:latest

# Docker Scout (native Docker scanning)
docker scout quickview myapp:latest
docker scout cves --format sarif myapp:latest
docker scout policy --output policy.json myapp:latest

# Grype (from Anchore)
grype myapp:latest --fail-on high --output json
grype myapp:latest --only-fixed --add-cpes-if-none

# Clair (API-based scanning)
clair-scanner --clair=http://clair:6060 --ip=localhost myapp:latest

# Generate SBOM
docker sbom myapp:latest --format cyclonedx-json
syft myapp:latest -o spdx-json > sbom.spdx.json
```

### Best Practices
- **Scan in CI/CD pipeline** with fail on CRITICAL+HIGH vulnerabilities
- **Scan base images before building** to catch issues early
- **Implement image signing** (cosign) to verify provenance
- **Use vulnerability allowlists** for false positives/exploits
- **Rebase images regularly** (weekly) for security patches
- **Scan for secrets** with `trufflehog` or `gitleaks` in CI
- **Monitor runtime vulnerabilities** with admission controllers (Kyverno, OPA)
- **Use minimal base images** (distroless, alpine) to reduce attack surface
- **Track base image updates** with Dependabot or Renovate

---