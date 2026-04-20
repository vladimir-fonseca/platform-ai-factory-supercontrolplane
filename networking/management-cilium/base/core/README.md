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
# management-cilium/base/core

This layer currently installs management-cluster Cilium entirely through Helm values.

Keep authored non-Helm resources here if you later add BGP, CiliumEnvoyConfig,
CIDR policy objects, or explicit network bootstrap manifests.
