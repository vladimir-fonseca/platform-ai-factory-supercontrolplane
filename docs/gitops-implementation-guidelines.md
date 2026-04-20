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
# GitOps implementation guidelines

## Intent

This document captures the final implementation guidelines for this package.

The goal is to keep the repository readable as an architecture implementation, not as generated output, release notes, or a sequence of historical fixes.

## 1. General design rules

The repository is organized around a clear control-plane model.

- The management plane owns provisioning, registration, policy, and orchestration.
- The child-cluster plane owns the attached NVIDIA platform stack.
- Each layer should express one responsibility clearly.
- Overlays should express deployment-specific decisions without polluting the shared base.

The package should remain declarative, reviewable, layered, predictable, and neutral in tone.

## 2. Naming guidelines

Names should be explicit, stable, and architecture-oriented.

### 2.1 Repository and package naming

Use the canonical package name consistently:

`platform-ai-factory-supercontrolplane`

Do not reintroduce earlier names such as:

`platform-ai-factory-supercontrolplane-kyverno-fullstack`

Any repo URL, bootstrap manifest, `Application`, `ApplicationSet`, or document reference should use the current canonical name only.

Canonical repo URL:

`https://git.example.com/platform-ai-factory-supercontrolplane.git`

### 2.2 Directory naming

Directory names should describe responsibility, not implementation history.

Examples of the intended pattern:

- `cluster-api`
- `capm3`
- `inventory`
- `catalog`
- `onboarding`
- `networking/management-cilium`
- `managed-addons/nvidia-ai-stack`
- `secret-management`

Avoid names that imply temporary work, generated output, or migration history.

### 2.3 Overlay naming

Overlay names should communicate deployment intent clearly.

Approved model:

- `dev`
- `prod`
- `prod-vast`
- `prod-weka`
- `prod-h100`
- `prod-b200`
- `prod-gb200`

Use overlay names to express environment, backend, and hardware family.

### 2.4 Resource naming

Kubernetes object names should be stable, lowercase, explicit, and aligned to layer purpose.

Examples:

- `supercontrol-root`
- `management-cilium`
- `cluster-api`
- `capm3`
- `inventory`
- `catalog`
- `kyverno`
- `services`
- `onboarding`
- `nvidia-ai-stack`

Avoid shorthand, cute names, or names that only make sense to the author.

## 3. Module design pattern

The package follows a repeatable module pattern.

Each major module should be structured so that the shared base is readable and neutral, while overlays carry deployment-specific decisions.

Example pattern:

`<module>/`
- `base/`
  - `core/`
  - `values/`
  - `kustomization.yaml`
- `overlays/`
  - `dev/`
  - `prod/`
  - `prod-vast/`
  - `prod-weka/`

### 3.1 base/core

`base/core` owns authored Kubernetes objects that define the structural content of the module.

Examples:

- namespaces
- `Application` objects
- singleton custom resources
- DPF objects
- service templates
- inventory objects
- policy manifests

### 3.2 base/values

`base/values` owns neutral Helm values and shared defaults.

These files should keep the base readable and renderable without embedding deployment-qualified decisions into the shared layer.

Examples:

- neutral resource requests
- neutral chart options
- common defaults
- values that are truly shared across environments

### 3.3 base/kustomization.yaml

`base/kustomization.yaml` composes the module.

It should:

- include `core/`
- reference `values/` where Helm is used
- define the shared structural rendering model
- stay neutral in tone and intent

### 3.4 overlays

`overlays/` owns environment-specific, backend-specific, and family-specific decisions.

That includes:

- validated image pins
- overlay-owned values files
- overlay-owned patch files
- family-specific profile choices
- backend-specific changes
- DPF flavor and sizing
- release-qualified operator values

### 3.5 Overlay entrypoint rule

An overlay `kustomization.yaml` should clearly show what it inherits and what it overrides.

Typical examples:

- `dev` starts from `../../base`
- `prod` starts from `../../base`
- `prod-vast` starts from `../vast`
- `prod-weka` starts from `../weka`
- `prod-h100` starts from `../prod` and adds a family patch

### 3.6 Pattern goals

This pattern exists for four reasons:

1. to keep the base structurally clear
2. to keep deployment-specific decisions out of the shared layer
3. to make review easier
4. to prevent environment drift from being hidden inside the base

In practice, the rule is simple:

- `core` defines structure
- `values` define neutral shared inputs
- `overlays` define qualified runtime decisions

## 4. Commenting guidelines

Comments should explain intent, not narrate history.

A good comment answers one of these questions:

- What responsibility does this layer own?
- Why is this object here?
- Why does this value belong in base instead of an overlay?
- What operational boundary is being enforced?
- What must be replaced before deployment?

### 4.1 Preferred comment style

Use concise architect-style comments.

Examples of the intended tone:

- Keep the shared base neutral. Environment overlays own validated runtime pins.
- This layer defines the registration boundary between a newly available child cluster and the GitOps control plane.
- Family-specific overlays replace the base full-GPU posture where MIG partitioning is intended.

Avoid change-log language such as:

- fixed
- corrected
- restored
- now uses
- this version adds
- compared to the previous version

### 4.2 Scope of comments

Comments should exist where they improve readability, especially in:

- top-level `kustomization.yaml`
- Argo CD `Application` manifests
- `ApplicationSet`
- overlay entrypoints
- DPF resources
- bootstrap templates
- major policy resources

Do not over-comment obvious YAML syntax.

## 5. Base versus overlay rules

This is one of the most important repository conventions.

### 5.1 Shared base

The shared base owns:

- structure
- object composition
- neutral defaults
- fallback values that keep objects readable in isolation

The base should not own environment-qualified release decisions unless they are truly universal.

### 5.2 Overlays

Overlays own:

- validated image pins
- qualified runtime values
- environment-specific posture
- family-specific posture
- backend-specific posture
- DPF sizing and flavor
- DOCA template version pins
- operator values that differ by deployment context

This rule applies consistently to:

- `capm3`
- `cluster-api`
- `kyverno`
- `networking`
- `managed-addons`

## 6. GitOps application guidelines

### 6.1 Argo CD Applications

Every Argo CD `Application` should:

- be named clearly
- point to the canonical repo URL
- use the correct overlay path
- include the correct sync-wave annotation
- stay aligned with the `README.md` wave table

### 6.2 Sync-wave ordering

The intended ordering is:

- `management-cilium = -30`
- `cluster-api = -20`
- `capm3 = -15`
- `inventory = -10`
- `catalog = -5`
- `kyverno = -3`
- `services = -2`
- `onboarding = 0`

These values should remain identical in:

- manifest annotations
- README documentation
- architecture diagrams when present

### 6.3 ApplicationSet

`ApplicationSet` is the runtime selector for child-cluster addon attachment.

It should:

- remain explicit
- select overlays from labels
- fall back safely when labels are absent
- document the activation chain clearly

It is a runtime fan-out mechanism, not a static app-of-apps child.

## 7. Documentation guidelines

Documentation should read like human architecture documentation.

It should be:

- concise
- neutral
- direct
- operationally useful

It should explain:

- what the package does
- what each layer owns
- how activation works
- what is base versus overlay
- what is still an engineering item versus a structural item

It should not:

- narrate tool history
- refer to previous fixes
- read like release notes
- sound defensive or autogenerated
- use vague wording such as “more concrete follow-on”

### 7.1 Preferred documentation tone

Use language such as:

- This package defines...
- This layer owns...
- The overlay model is...
- The activation path is...
- The package should be read as...

Avoid language such as:

- now
- fixed
- corrected
- compared to previous
- this repo adds

## 8. Asset guidelines

For diagrams and other assets:

- do not regenerate unless explicitly requested
- do not rasterize SVGs
- do not wrap JPEGs in SVGs
- do not rename or reinterpret user-provided assets
- if a file is provided, treat it as opaque unless asked to transform it

If copying from an archive:

- copy files unchanged
- skip metadata files such as `__MACOSX` and `._*`

## 9. Consistency rules

Any change must preserve consistency across:

- manifests
- overlays
- docs
- bootstrap files
- repo URLs
- sync-wave tables
- activation-chain docs

Do not fix one layer and let another drift.

Before considering a package clean, verify at minimum:

- sync-wave annotations match `README.md`
- the canonical repo URL is used everywhere
- overlays referenced by `Application` and `ApplicationSet` exist
- overlay patches referenced by `kustomization.yaml` exist
- no stale package names remain
- docs do not reintroduce regression language

## 10. Baseline rules

When a package is declared baseline or source of truth, it should mean:

- no unrelated edits from that point
- future work is delta-only
- no silent refactors
- no asset changes unless requested
- no naming drift
- no reintroduction of stale wave values
- no removal of established overlay patterns

A baseline should be treated as frozen until a specific requested change is applied.

## 11. Practical summary

The final implementation standard for this package is:

- clear names
- explicit ownership
- neutral base
- overlay-owned environment decisions
- architect-style comments
- human documentation tone
- no regression language
- no asset manipulation unless requested
- strict consistency between manifests and docs
