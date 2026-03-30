 

### Core Concept
CI/CD pipelines automate building, testing, and deploying containerized applications. Docker integration requires caching strategies, build optimization, image tagging, and security scanning. Modern approaches use BuildKit, Kaniko (rootless), or buildah for secure builds in CI environments.

### Key Terminologies
- **Docker-in-Docker (DinD)**: Running Docker daemon inside CI container (privileged)
- **Docker-out-of-Docker (DooD)**: Mounting host Docker socket (faster, less isolated)
- **Kaniko**: Running Docker builds without privileged access (in Kubernetes)
- **Build Cache**: Layer caching across CI runs (registry, GitHub Actions cache)
- **Image Promotion**: Tagging images for dev → staging → prod
- **SBOM**: Software Bill of Materials for supply chain security

### How it Works
CI/CD flow:
1. Checkout code
2. Set up Docker Buildx for multi-arch builds
3. Cache image layers (pull previous version, mount cache)
4. Build with `--cache-from` using registry or local cache
5. Run tests (unit, integration, security scan)
6. Push to registry with short SHA tag + `latest`
7. Deploy (update Kubernetes manifests, restart services)

Caching strategies:
- **Registry cache**: `--cache-from type=registry,ref=myapp:buildcache`
- **GitHub Actions cache**: `type=gha,mode=max`
- **Local directory**: `--cache-from type=local,src=/tmp/cache`

### CLI Commands/Syntax
```yaml
# GitHub Actions workflow
name: CI/CD Pipeline
on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        with:
          driver: docker-container
      
      - name: Cache Docker layers
        uses: actions/cache@v3
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-
      
      - name: Login to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}
            ghcr.io/${{ github.repository }}:${{ github.ref_name }}
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache-new,mode=max
          target: production
          build-args: |
            BUILDKIT_INLINE_CACHE=1
      
      - name: Trivy vulnerability scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/${{ github.repository }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp \
            myapp=ghcr.io/${{ github.repository }}:${{ github.sha }}
          kubectl rollout status deployment/myapp

# GitLab CI
build:
  stage: build
  variables:
    DOCKER_DRIVER: overlay2
    DOCKER_TLS_CERTDIR: ""
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --cache-from $CI_REGISTRY_IMAGE:latest -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

### Best Practices
- **Use Kaniko or Buildah** instead of DinD in Kubernetes CI (more secure)
- **Implement multi-stage cache**: registry for layer sharing, local for speed
- **Tag images with commit SHA** for traceability, `latest` only for main branch
- **Scan images for vulnerabilities** before pushing (Trivy, Grype, Snyk)
- **Use image promotion** (promote SHA tags, don't rebuild for environments)
- **Sign images** with cosign in CI pipeline for supply chain security
- **Set resource limits** for CI runners to prevent noisy neighbor issues
- **Implement atomic deployments** with rollback capability (Argo Rollouts, Flagger)

---
