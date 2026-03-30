 

### Core Concept
Registries store and distribute container images. Docker Hub is public default, but production workloads need private registries (ECR, GAR, ACR, Harbor, Nexus). Registries implement OCI distribution specification for content-addressable storage, manifest management, and blob upload/download APIs.

### Key Terminologies
- **Registry**: Server implementing OCI distribution API
- **Repository**: Collection of related images (e.g., `library/nginx`)
- **Tag**: Human-readable reference to image (e.g., `nginx:1.25`)
- **Digest**: Content-addressable SHA256 hash (e.g., `@sha256:abc123...`)
- **Manifest**: JSON describing image layers and configuration
- **Blob**: Binary large object (compressed layer tarball)
- **Registry Mirror**: Pull-through cache for upstream registries

### How it Works
Push flow:
1. Client computes digest for each layer
2. HEAD request checks if blob exists in registry
3. POST initiates blob upload (chunked or monolithic)
4. PUT completes upload with digest verification
5. Final PUT uploads manifest referencing blobs
6. Registry updates tag to point to new manifest

Pull flow:
1. Client requests manifest by tag/digest
2. Registry returns manifest with layer digests
3. Client checks local storage for existing layers
4. Downloads missing layers in parallel
5. Unpacks layers to storage driver (overlay2)

Authentication: Basic, Token (Bearer), OAuth2. Registry supports delegated auth via external service.

### CLI Commands/Syntax
```bash
# Login and authentication
docker login ghcr.io -u USERNAME --password-stdin
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account>.dkr.ecr.us-east-1.amazonaws.com

# Push/pull with digests
docker tag myapp:latest myregistry.com/myapp:v1.0
docker push myregistry.com/myapp:v1.0
docker pull myregistry.com/myapp@sha256:abc123...  # Immutable pull

# Registry operations with skopeo
skopeo copy docker-daemon:myapp:latest docker://myregistry.com/myapp:latest
skopeo inspect --creds=user:pass docker://myregistry.com/myapp:v1.0

# Run private registry
docker run -d -p 5000:5000 --name registry -v /data/registry:/var/lib/registry registry:2

# Mirror configuration
# /etc/docker/daemon.json
{
  "registry-mirrors": ["https://mirror.gcr.io", "https://registry-mirror.company.com"]
}
```

### Best Practices
- **Always pull by digest** in production: `image: myapp@sha256:...` (immutable, prevents tag overwrites)
- **Implement vulnerability scanning** in registry (ECR scanning, Trivy, Clair)
- **Set retention policies** to delete old tags/images: `docker image prune -a --filter "until=720h"`
- **Use registry authentication** never pull from public without credentials
- **Configure garbage collection** for private registries: `docker exec registry bin/registry garbage-collect /etc/docker/registry/config.yml`
- **Enable registry caching** with `--cache-blobs` to reduce upstream bandwidth
- **Implement image signing** with cosign: `cosign sign --key cosign.key myregistry.com/myapp:latest`

---
