# Tinfoil Function Configuration

This repository contains the configuration for your Tinfoil Function deployment.

## Quick Start

1. Go to [dash.tinfoil.sh](https://dash.tinfoil.sh)
2. Click **Create Function**
3. Select **Simple Deploy**
4. Provide:
   - This repository URL
   - Your container image with SHA hash (e.g., `vllm/vllm-openai:v0.14.1@sha256:abc123...`)
   - Environment variables and secrets
   - Resource requirements (CPUs, memory, GPUs)
5. Click **Deploy**

The `tinfoil-config.yml` file will be automatically updated with your settings and an initial deployment will be triggered.

## Updating Your Function

After the initial deployment, update your function by editing the config directly and pushing a new tag:

```bash
# 1. Edit tinfoil-config.yml (update image version, env vars, etc.)

# 2. Commit your changes
git add tinfoil-config.yml
git commit -m "Update to vllm v0.14.1"

# 3. Push a new tag to trigger deployment
git tag v2
git push origin main --tags
```

Your new deployment will be built from the tagged commit. Each tag creates an auditable record in the transparency log.

### Common Updates

**Update container image:**
```yaml
containers:
  - name: "app"
    image: "vllm/vllm-openai:v0.14.1@sha256:6fc52be4609fc19b09c163be2556976447cc844b8d0d817f19bc9e1f44b48d5a"
```
To get the SHA hash for an image: `docker pull <image> && docker inspect --format='{{index .RepoDigests 0}}' <image>`

**Add environment variable:**
```yaml
containers:
  - name: "app"
    env:
      - LOG_LEVEL: "info"      # Hardcoded value
      - MAX_WORKERS: "4"
    secrets:
      - API_KEY               # Looked up from external-config
```

**Expose new path:**
```yaml
shim:
  paths:
    - /v1/chat/completions
    - /v1/embeddings    # Add new endpoint
    - /health
```

## Manual Configuration

If you prefer to configure manually, edit `tinfoil-config.yml` directly. See the [configuration reference](https://docs.tinfoil.sh/functions/config) for all available options.

### Example: vLLM Inference Server

```yaml
shim-version: v0.3.12@sha256:b81f2295ae6750d61e94f810ce24077360001b6ec795d13643f3170df29e304d
cvm-version: 0.6.6
cpus: 16
memory: 65536

containers:
  - name: "inference"
    image: "vllm/vllm-openai:v0.14.1@sha256:6fc52be4609fc19b09c163be2556976447cc844b8d0d817f19bc9e1f44b48d5a"
    runtime: nvidia
    gpus: all
    ipc: host
    command: [
      "--model", "/models/my-model",
      "--port", "8001"
    ]

shim:
  listen-port: 443
  upstream-port: 8001
  paths:
    - /v1/chat/completions
    - /v1/completions
    - /health
    - /metrics
```

### Configuration Reference

#### Top-level Fields

| Field | Description | Valid Values |
|-------|-------------|--------------|
| `shim-version` | Version of the Tinfoil shim | Pinned version with SHA |
| `cvm-version` | Confidential VM version | Version string (e.g., `0.6.6`) |
| `cpus` | Number of vCPUs | 2, 4, 8, 16, 32 |
| `memory` | Memory in MB | 8192, 16384, 32768, 65536, 131072 |

#### Container Configuration

Each container in the `containers` array supports:

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Container identifier | `"inference"` |
| `image` | Container image with SHA256 digest | `"image:tag@sha256:..."` |
| `command` | Command arguments | `["--port", "8001"]` |
| `entrypoint` | Override entrypoint | `["python"]` |
| `env` | Environment variables | See below |
| `secrets` | Secret keys from external-config | `["API_KEY"]` |
| `runtime` | Container runtime | `"nvidia"` |
| `gpus` | GPU allocation | `"all"`, `"0,1"`, or count |
| `ipc` | IPC mode (for multi-GPU) | `"host"` |
| `volumes` | Bind mounts | `["/mnt/ramdisk/data:/data"]` |

**Environment variables** support two formats:
```yaml
env:
  - LOG_LEVEL: "info"    # Hardcoded value
  - CONFIG_PATH          # Looked up from external-config env section

secrets:
  - API_KEY              # Looked up from external-config secrets section
```

#### Shim Configuration

| Field | Description | Example |
|-------|-------------|---------|
| `shim.listen-port` | External port (usually 443) | `443` |
| `shim.upstream-port` | Port your container listens on | `8000`, `8001` |
| `shim.paths` | URL paths to expose | `["/health", "/v1/*"]` |

## Security

Your deployment runs in a Confidential Virtual Machine (CVM) with:
- Hardware-level memory encryption
- Attestation via Sigstore
- No access to your data by Tinfoil or cloud providers

**Container images must include a SHA256 hash** (e.g., `image:tag@sha256:...`).

The configuration in this repository is signed and included in the transparency log for auditability.

## Support

- [Documentation](https://docs.tinfoil.sh)
- [Email Support](mailto:contact@tinfoil.sh)
