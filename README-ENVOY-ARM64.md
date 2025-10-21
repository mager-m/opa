# OPA-Envoy ARM64 Build

This repository contains infrastructure to build OPA with the Envoy External Authorization plugin for ARM64 architecture.

## Overview

The [OPA-Envoy plugin](https://github.com/open-policy-agent/opa-envoy-plugin) extends OPA with a gRPC server that implements the Envoy External Authorization API. This allows you to use OPA to enforce fine-grained, context-aware access control policies with Envoy without modifying your microservices.

## Files

- **Dockerfile.envoy**: Multi-stage Dockerfile that builds the OPA-Envoy plugin for ARM64
- **.github/workflows/build-push-envoy-arm64.yaml**: GitHub Actions workflow that builds and pushes ARM64 images to GitHub Container Registry

## Docker Image

The workflow automatically builds and pushes Docker images to GitHub Container Registry with the following tags:

- `ghcr.io/<org>/opa-envoy:arm64` - Latest ARM64 build
- `ghcr.io/<org>/opa-envoy:latest-arm64` - Latest from main branch
- `ghcr.io/<org>/opa-envoy:<branch-name>` - Branch-specific builds
- `ghcr.io/<org>/opa-envoy:<version>` - Tagged releases

## Usage

### Using the Pre-built Image

```yaml
# In your Kubernetes deployment or docker-compose.yaml
apiVersion: v1
kind: Pod
metadata:
  name: opa-envoy
spec:
  containers:
  - name: opa
    image: ghcr.io/<org>/opa-envoy:arm64
    args:
    - "run"
    - "--server"
    - "--addr=localhost:8181"
    - "--set=plugins.envoy_ext_authz_grpc.addr=:9191"
    - "--set=plugins.envoy_ext_authz_grpc.path=envoy/authz/allow"
```

### Building Locally

To build the image locally:

```bash
# Build for ARM64
docker buildx build --platform linux/arm64 -f Dockerfile.envoy -t opa-envoy:arm64 .

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -f Dockerfile.envoy -t opa-envoy:latest .
```

## GitHub Actions Workflow

The workflow is triggered on:
- Push to `main` branch
- Push to branches matching `copilot/**`
- Tagged releases (`v*`)
- Pull requests to `main`
- Manual workflow dispatch

### Secrets Required

The workflow uses GitHub's built-in `GITHUB_TOKEN` for authentication to GitHub Container Registry. No additional secrets are required.

### Workflow Features

- Uses Docker BuildKit for efficient caching
- Builds specifically for ARM64 platform
- Pushes to GitHub Container Registry (GHCR)
- Generates multiple tags for easy reference
- Skips push on pull requests (build-only for validation)

## Configuration

### Envoy External Authorization Plugin

The plugin configuration can be set via command-line arguments or a configuration file:

```yaml
# config.yaml
plugins:
  envoy_ext_authz_grpc:
    addr: ":9191"
    path: "envoy/authz/allow"
    dry-run: false
    enable-reflection: false
```

Run with: `opa run --server --config-file=config.yaml`

### Available Plugin Options

- `addr`: gRPC server listening address (default: `:9191`)
- `path`: Policy decision path (default: `envoy/authz/allow`)
- `dry-run`: Test mode that always allows requests (default: `false`)
- `enable-reflection`: Enable gRPC server reflection (default: `false`)

## References

- [OPA-Envoy Plugin Repository](https://github.com/open-policy-agent/opa-envoy-plugin)
- [OPA Envoy Documentation](https://www.openpolicyagent.org/docs/latest/envoy-introduction/)
- [Envoy External Authorization](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/security/ext_authz_filter)

## License

This project follows the same license as the Open Policy Agent project (Apache 2.0).
