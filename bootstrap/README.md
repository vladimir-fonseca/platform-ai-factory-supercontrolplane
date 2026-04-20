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
# Bootstrap

Use one of these bootstrap entrypoints:

- `bootstrap/dev/kustomization.yaml`
- `bootstrap/prod/kustomization.yaml`

Each bundle creates the package-specific Argo CD project first and then the matching site-specific root Application.
