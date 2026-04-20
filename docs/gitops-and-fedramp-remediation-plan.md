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
# GitOps and FedRAMP remediation plan

## Intent

This note captures the current package posture from two angles:

- GitOps operating discipline
- FedRAMP-oriented control readiness

The package is already structurally strong. The remaining work is not about repo hygiene. It is about moving from a well-composed reference platform to a controlled production platform that can stand up to regulated-environment review.

## Current posture

### What is already solid

The package already has the foundations of a good GitOps platform:

- declarative, Git-authored lifecycle
- explicit dependency ordering through sync waves
- pinned Helm chart versions
- Argo CD reconciliation with drift correction
- a consistent base-plus-overlay model
- a meaningful AppProject boundary
- a strong Kyverno admission baseline
- default-deny namespace network posture
- digest-based image integrity enforcement
- approved-registry enforcement

### What is not yet production-controlled

The package still behaves like an engineering-controlled platform rather than a regulated-production platform.

The main gap is not the absence of YAML. It is the absence of control boundaries around change, identity, cryptography, and evidence.

## GitOps findings

### 1. Production revisions should not track a mutable branch

Using `targetRevision: main` is acceptable for development, but it is not appropriate for a controlled production path.

A production deployment should reconcile to one of:

- a release tag
- an immutable commit SHA

### 2. Auto-prune is too aggressive on destructive infrastructure layers

`prune: true` is useful for application layers, but it is riskier for infrastructure-bound layers such as:

- inventory
- catalog
- Cluster API objects

### 3. AppProject scope is broader than it should be

A project boundary that allows:

- wildcard source repositories
- wildcard cluster-scoped resource types

is wider than necessary for this platform.

### 4. Bootstrap still needs repo-name consistency checks

Bootstrap and root templates must always reference the current canonical repository identity.

## FedRAMP-oriented findings

### 1. Audit architecture is not yet present

The package does not yet define:

- Kubernetes API audit logging
- Kyverno violation log handling
- runtime or host-level audit collection
- forwarding to a durable log system
- retention and tamper-protection posture

### 2. Identity and privileged access controls are incomplete

The package does not yet define:

- OIDC or SSO integration for cluster or Argo CD access
- MFA-backed privileged access
- an explicit privileged operator access model

### 3. Secret management is only documented, not operationalized

The package documents a minimal secret-management pattern, but the lifecycle is still missing:

- creation
- rotation
- revocation
- auditability
- ownership model

### 4. Encryption posture is still incomplete

There is no complete end-to-end statement for:

- etcd encryption at rest
- PersistentVolume encryption at rest
- east-west workload encryption in transit
- cryptographic boundary documentation

### 5. Supply-chain verification is still absent

The package enforces registries and digests, which is good, but that is not the full supply-chain posture.

The missing elements are:

- signed images
- provenance attestation
- SBOM-driven review
- policy verification of those artifacts

## Prioritized remediation plan

### Priority 1 — production GitOps control boundary

1. replace production `targetRevision: main` with immutable release refs
2. narrow `AppProject.sourceRepos`
3. narrow `AppProject` cluster-scoped resource permissions
4. gate destructive prune behavior on infrastructure layers
5. verify bootstrap and root-template repo identity everywhere

### Priority 2 — identity, approval, and secret lifecycle

1. define OIDC / SSO for Argo CD and cluster administration
2. require MFA-backed privileged access
3. operationalize secret lifecycle through an approved backend
4. define environment ownership for secret creation and rotation
5. document operator roles and privilege boundaries

### Priority 3 — audit and evidence

1. add Kubernetes API audit policy configuration
2. define Kyverno policy-event collection and forwarding
3. add durable log forwarding to a protected sink
4. define retention posture and access controls for logs
5. document evidence sources for operational review

### Priority 4 — encryption and network trust posture

1. add etcd encryption-at-rest configuration
2. define PV encryption expectations
3. define east-west encryption posture, such as Cilium mutual authentication
4. document trusted communication boundaries
5. map cryptographic expectations to the environment standard

### Priority 5 — supply-chain and release integrity

1. adopt image signing requirements
2. verify signatures in admission or deployment policy
3. define SBOM collection and review
4. define provenance requirements for promoted artifacts
5. document release qualification inputs for production overlays

## Architectural conclusion

This package already reads like a real platform package, not a loose manifest collection.

The remaining work is about control rigor:

- controlling what can change
- controlling who can operate
- proving what happened
- protecting what is stored and transmitted
- and verifying what software is allowed to run
