# NemoClaw-at-Home (CPU/Intel Fork)

This is a modified fork of NVIDIA NemoClaw designed to run on systems without NVIDIA GPUs (e.g., Intel Integrated Graphics, CPU-only).

## Workarounds Applied
1. **Bypassed NVIDIA GPU Requirement:** `nim.js` now detects Intel and generic GPUs to allow the onboarding wizard to proceed.
2. **First-Class Local Inference:** Removed `EXPERIMENTAL` flags for Ollama and vLLM. The wizard will now prioritize your local Ollama/vLLM instances.
3. **Optional OpenShell Gateway:** If the NVIDIA-specific `openshell` gateway fails to start (common on Windows/WSL without custom kernels), the stack will continue in "lite mode" using standard containers.
4. **Ollama Integration:** Added a native `ollama` profile to the NemoClaw blueprint.

## How to Run
1. Ensure **Ollama** is running on your machine:
   ```bash
   ollama serve
   ```
2. Navigate to this folder:
   ```bash
   cd C:\Users\uusis\nemoclaw-at-home
   ```
3. Run the onboarding wizard:
   ```bash
   node bin/nemoclaw.js onboard
   ```

## Collaboration (Autoresearch Style)
This fork is designed to work alongside the `autoresearch-at-home` swarm. Once your sandbox is running, you can point your `coordinator.py` to the sandbox's API endpoint to share results with the community.

---
*Based on NVIDIA NemoClaw (GTC 2026).*
