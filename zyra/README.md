# ZyraOS Capability Overlay

ZyraOS is a project overlay on top of NVIDIA OpenShell. It keeps the upstream
runtime, policy engine, sandboxing, providers, GPU support, OCSF logging, and
OpenTelemetry intact while adding a ZyraOS-specific operator surface.

## Goals

- Preserve easy synchronization with `NVIDIA/OpenShell`.
- Make a high-capability local agent workspace reproducible.
- Keep capability escalation explicit, observable, and policy-controlled.
- Support macOS Apple Silicon, Linux, Docker/Podman, Kubernetes, and GPU-ready
  gateways through capabilities already exposed by OpenShell.
- Keep credentials outside the sandbox by using OpenShell providers.

## Components

| Path | Purpose |
|---|---|
| `zyra/policy.dev.yaml` | Enforced development policy for package, source, NVIDIA, and model-provider endpoints |
| `zyra/bin/zyra-doctor` | Read-only environment and gateway health report |
| `zyra/bin/zyra-launch` | Reproducible sandbox launcher |
| `zyra/capabilities.yaml` | Machine-readable ZyraOS capability manifest |

## Quick start

```bash
bash ./zyra/bin/zyra-doctor
bash ./zyra/bin/zyra-launch
```

The launcher uses OpenShell's normal sandbox and policy machinery. It does not
disable policy enforcement. The default upload target is `/sandbox`, which is
the canonical writable workdir for MicroVM sandboxes.

### Optional environment variables

```bash
export ZYRA_SANDBOX=zyra-dev
export ZYRA_IMAGE=ubuntu:24.04
export ZYRA_CPU=4
export ZYRA_MEMORY=8Gi
export ZYRA_GPU=1
export ZYRA_PROVIDERS=github,openai
export ZYRA_UPLOAD=.
export ZYRA_WORKDIR=/sandbox
./zyra/bin/zyra-launch
```

`ZYRA_GPU` is optional and only works when the selected OpenShell gateway and
compute driver support GPU allocation.

`ZYRA_PROVIDERS` is a comma-separated list of provider instance names already
configured in OpenShell. Credentials stay behind the OpenShell provider boundary.

To execute a command instead of opening the retained shell:

```bash
bash ./zyra/bin/zyra-launch -- python3 --version
```

## Capability model

ZyraOS composes existing OpenShell primitives rather than bypassing them:

1. Sandbox isolation — filesystem, process, and network policy.
2. Provider boundary — credentials are resolved at approved endpoints.
3. Policy verification — policy changes remain reviewable through OpenShell.
4. Observability — OCSF events, logs, and OpenTelemetry remain available.
5. Multi-agent orchestration — build on OpenShell examples and SDKs while
   keeping each worker in a bounded sandbox.
6. GPU / inference — opt into supported GPU resources and provider-backed
   inference without making GPU access mandatory.
7. Human authorization — capability growth is additive and explicit; the
   operator remains the authority for policy and credential changes.

## Extending the policy

Do not broaden network policy with a single unrestricted wildcard. Add a named
endpoint and, where supported, use request-level inspection with
`enforcement: enforce`.

For GitHub, the development policy allows read access broadly but scopes
mutation to the `sonoxo/ZyraOS` repository.

## Upstream sync

Keep ZyraOS custom work inside the `zyra/` namespace whenever possible. This
minimizes merge conflicts when pulling updates from `NVIDIA/OpenShell`.
