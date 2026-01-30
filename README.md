# Tinfoil Function Configuration

This repository contains the configuration for your Tinfoil Function deployment.

## Quick Start

1. Go to [dash.tinfoil.sh](https://dash.tinfoil.sh)
2. Click **Create Function**
3. Select **Simple Deploy**
4. Provide:
   - This repository URL
   - Your container image with SHA hash (e.g., `vllm/vllm-openai:v0.13.0@sha256:abc123...`)
   - Container arguments
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
git commit -m "Update to vllm v0.14.0"

# 3. Push a new tag to trigger deployment
git tag v2
git push origin main --tags
```

Your new deployment will be built from the tagged commit. Each tag creates an auditable record in the transparency log.

### Common Updates

**Update container image:**
```yaml
args:
  - "vllm/vllm-openai:v0.14.0@sha256:d623253f2ba246378421c9642e20885e65257f38418ff26d48c81aea1702521b"
```
To get the SHA hash for an image: `docker pull <image> && docker inspect --format='{{index .RepoDigests 0}}' <image>`

**Add environment variable:**
```yaml
args:
  - "-e"
  - "NEW_VAR=value"
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
shim-version: v0.3.5@sha256:f44c65a1f7db476be20c3431bb6facea8c2966606a07fbb5724a619f82bfa558
cvm-version: 0.5.16
cpus: 16
memory: 65536
gpus: full

containers:
  - name: "inference"
    image: ""
    args:
      - "--runtime"
      - "nvidia"
      - "--gpus"
      - "all"
      - "vllm/vllm-openai:v0.13.0@sha256:d623253f2ba246378421c9642e20885e65257f38418ff26d48c81aea1702521b"
      - "--model"
      - "/models/my-model"
      - "--port"
      - "8001"

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
| `cvm-version` | Confidential VM version | Version string (e.g., `0.5.16`) |
| `cpus` | Number of vCPUs | 2, 4, 8, 16, 32 |
| `memory` | Memory in MB | 8192, 16384, 32768, 65536, 131072 |
| `gpus` | GPU allocation | `""` (none), `"full"` |

#### Container Configuration

Each container in the `containers` array supports:

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Container identifier | `"inference"` |
| `image` | Leave empty (image specified in args) | `""` |
| `args` | Docker run arguments as array | See below |

Common `args` patterns:

```yaml
args:
  # Runtime options
  - "--runtime"
  - "nvidia"
  - "--gpus"
  - "all"
  - "--ipc"
  - "host"

  # Environment variables
  - "-e"
  - "LOG_LEVEL=info"
  - "-e"
  - "MAX_WORKERS=4"

  # Secrets (referenced from Tinfoil secrets manager)
  - "-e"
  - "API_KEY=${SECRET_API_KEY}"
  - "-e"
  - "DATABASE_URL=${SECRET_DB_URL}"

  # Image and command (SHA hash required)
  - "vllm/vllm-openai:v0.13.0@sha256:d623253f2ba246378421c9642e20885e65257f38418ff26d48c81aea1702521b"
  - "--model"
  - "/models/llama"
  - "--port"
  - "8001"
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
