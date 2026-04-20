<!--
-----------------------------------------------------------------------------
Copyright (c) 2026 Vladimir Fonseca

This file is part of the platform-ai-factory-supercontrolplane project.

Defines platform architecture, integration patterns, and deployment models.

Third-party components referenced here remain the property of their
respective owners.

Licensed under the Apache License, Version 2.0.
-----------------------------------------------------------------------------

-->
# MIG posture by hardware family

## Intent

This note records the hardware-family-oriented MIG posture used by the package.

The package keeps the shared GPU Operator ClusterPolicy on the safe universal default of full-GPU presentation, and then routes clusters into family-specific overlays that make the MIG posture explicit.

## Base posture

The shared base keeps:

- `mig.strategy: none`
- `migManager.enabled: false`

That avoids imposing a single MIG choice on every cluster.

## Family-specific overlays

```text
Overlay     Target families   MIG posture
----------  ----------------  -----------------------------------
prod-h100   h100, h200        single / all-1g.10gb
prod-b200   b200, b300        none, migManager enabled
prod-gb200  gb200, gb300      none, migManager disabled
```

## Rationale

### Hopper / H100

H100-class clusters are the most natural place to expose smaller MIG slices for shared development and inference use.

The current overlay uses:

- `mig.strategy: single`
- `migManager.config.default: all-1g.10gb`

That is a practical starting point for shared H100 environments.

### Blackwell / B200

For the B200 production posture, the package stays on full-GPU presentation for now.

That is the safer default until the workload split is qualified. The overlay still enables `migManager` so a validated geometry can be introduced later without changing the control path.

### Blackwell-Ultra / GB200

For GB200 rack-scale systems, the package keeps full GPUs and disables the MIG manager.

That reflects the intended rack-scale NVLink fabric posture, where partitioning individual GPUs is usually less aligned with the overall system design.

## ApplicationSet routing

The ApplicationSet now selects the addon overlay by the `gpu-family.nvidia.com/name` label carried on the Argo CD cluster secret.

If the label is absent, the package falls back to the neutral production overlay.

That means the package can express a different MIG posture per hardware family without forking the shared base ClusterPolicy.

## Environment note

These are package-authored example postures, not final production qualification.

Before deployment, validate the chosen geometry against:

- the actual workload split
- the GPU Operator release
- the CUDA / driver stack
- and the scheduler posture used in the target environment



## MIG specifications by hardware family

| Feature | NVIDIA GB300 NVL72 (Blackwell Ultra) | NVIDIA HGX B300 (Blackwell Ultra) | NVIDIA GB200 NVL72 | NVIDIA HGX B200 |
|--------|--------------------------------------|-----------------------------------|--------------------|-----------------|
| AI Security | Yes | Yes | Yes | Yes |
| Instance Types | 7× 34 GB<br>4× 69 GB<br>2× 139 GB<br>1× 279 GB | 7× 32 GB<br>4× 67 GB<br>2× 135 GB<br>1× 270 GB | 7× 23 GB<br>4× 46 GB<br>2× 93 GB<br>1× 186 GB | 7× 21 GB<br>4× 45 GB<br>2× 90 GB<br>1× 180 GB |
| GPU Profiling and Monitoring | Concurrently on all instances | Concurrently on all instances | Concurrently on all instances | Concurrently on all instances |
| Secure Tenants | 7× | 7× | 7× | 7× |
| Media Decoders | Dedicated NVJPEG and NVDEC per instance | Dedicated NVJPEG and NVDEC per instance | Dedicated NVJPEG and NVDEC per instance | Dedicated NVJPEG and NVDEC per instance |
