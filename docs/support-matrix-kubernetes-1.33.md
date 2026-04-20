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
# Kubernetes 1.33 support matrix summary

## Support verification from primary sources

- Kubernetes **1.33** is currently active, and the latest patch release is **1.33.10**.
- NVIDIA **GPU Operator** support docs list Kubernetes **1.29–1.33** as supported on validated platforms.
- NVIDIA **AI Enterprise 7.0** support matrix shows **Upstream Kubernetes 1.29–1.33** with **GPU Operator**, **Network Operator**, and **DPU Operator (DPF)** supported in the listed platform combinations.
- NVIDIA **AI Enterprise 7.1** lists **GPU Operator 25.3.2**, **Network Operator 25.7.0**, and **DPU Operator (DPF) 25.7.0** as supported infrastructure software.
- KubeVirt release notes state **KubeVirt v1.6** is built for **Kubernetes v1.33** and supports the previous two versions as well.

## Caution

This verifies **1.33 compatibility in principle** and for NVIDIA-supported release lines, but this package still uses specific chart/version pins and non-NVIDIA components too. Those should stay aligned with the exact platform combination you plan to deploy.
