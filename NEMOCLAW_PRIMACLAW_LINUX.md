# NemoClaw + Primaclaw on Linux

## Overview

This document describes the setup and known limitations of running NemoClaw (NVIDIA OpenShell) with Primaclaw (local llama.cpp inference) on Linux.

## Setup Components

### Primaclaw (e727-local-ai)
- Location: `/home/m/e727-local-ai`
- Model: `qwen2.5-coder-3b-instruct-q6_k.gguf`
- Port: 8082
- Direct URL: `http://localhost:8082/v1/models`

### NemoClaw (OpenShell)
- Location: `/home/m/NemoClaw`
- Gateway: Running on port 8085
- Sandbox: `test2`
- Provider: `local-server` (configured for primaclaw-bridge)

### Bridge Container
A Docker container running prima.cpp's llama-server on the openshell network:
- Container name: `primaclaw-bridge`
- Network: `openshell-cluster-nemoclaw`
- Accessible from k8s pods at: `http://primaclaw-bridge:8080`

## Known Limitation: Sandbox DNS on Linux

### The Problem

On Linux, the OpenShell sandbox's k8s DNS (CoreDNS) is non-functional. When connecting to a sandbox:

```bash
openshell sandbox connect test2
```

Inside the sandbox, any DNS lookup fails:
```
python3 -c "import urllib.request; urllib.request.urlopen('http://example.com')"
# Result: [Errno -3] Temporary failure in name resolution
```

This includes:
- External domains (google.com, github.com)
- Kubernetes services (inference.local, openshell.default)
- Docker host aliases (host.docker.internal)

### Why This Happens

The sandbox runs inside k3s-in-Docker. The sandbox pod's DNS resolver points to k8s CoreDNS (10.42.0.3), but CoreDNS inside the container cannot reach external DNS servers due to container network isolation.

### Root Cause

On Windows with Docker Desktop, `host.docker.internal` works reliably because Docker Desktop handles networking differently (WSL2).

On Linux with native Docker/k3s, the network isolation prevents k8s pods from:
1. Reaching the host Docker bridge (172.17.0.0/16)
2. Resolving external domains through the proxy
3. Using CoreDNS to forward queries upstream

### Verified Working

- Primaclaw bridge container can reach primaclaw: `curl http://primaclaw-bridge:8080/v1/models` ✅
- Proxy pod can reach primaclaw: `kubectl exec primaclaw-proxy -- curl http://primaclaw-bridge:8080/v1/models` ✅
- Sandbox cannot reach anything due to DNS failure ❌

## Workarounds

### 1. Use Primaclaw Directly (Recommended)

Primaclaw works independently of NemoClaw:

```bash
cd ~/e727-local-ai
source .venv/bin/activate
python -m src.worker
```

Access at `http://localhost:8082/v1/models`

### 2. Use Open-WebUI

Open-WebUI container is running and can connect to Primaclaw:

```bash
# Access Open-WebUI at http://localhost:8080
# Configure to use http://localhost:8082 as backend
```

### 3. Accept Cloud Inference

NemoClaw is designed for cloud inference (NVIDIA NIM, Anthropic, OpenAI). For local inference, use Primaclaw directly.

## Files

- Primaclaw config: `/home/m/e727-local-ai/src/config.py`
- Worker script: `/home/m/e727-local-ai/src/worker.py`
- NemoClaw policy: `/home/m/local-policy.yaml`
- Bridge startup: `/home/m/start-primaclaw-bridge.sh`
- Bridge Dockerfile: `/home/m/primaclaw-container/Dockerfile`

## Related Issues

- NemoClaw fork: https://github.com/bopalvelut-prog/NemoClaw
- e727-local-ai: https://github.com/bopalvelut-prog/e727-local-ai
- OpenShell: https://github.com/NVIDIA/OpenShell
