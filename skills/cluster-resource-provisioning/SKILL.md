---
name: "cluster-resource-provisioning"
description: "Manages Kubernetes credentials, service accounts, secrets, configmaps, persistent volumes, and node affinities for agent pods."
license: Apache-2.0
---

# Cluster Resource Provisioning

## Overview
This skill provisions and binds Kubernetes cluster-level resources, secrets, service accounts, and volume mounts necessary for agent pods to operate securely and efficiently.

## Key Capabilities
- **Credential & Secret Injection**: Mounts Kubernetes secrets and service account tokens into agent pods without exposing credentials in console logs.
- **Quota & Capacity Auditing**: Audits cluster namespace capacity and evaluates pending job queues against administrator container limits.
- **Volume & Storage Binding**: Attaches PersistentVolumeClaims and emptyDir volumes for workspace caching and build artifact storage.
- **Node Affinity & Tolerations**: Applies custom node selectors, tolerations, and affinity rules to route compute-heavy builds to dedicated node pools.

## Operational Workflow
1. **Quota Evaluation**: Invoke `cloud_scaler` to ensure available pod slots before scheduling.
2. **Secret Resolution**: Resolve required cluster credentials using `kube_secret_manager`.
3. **Storage Configuration**: Prepare volume attachments, claims, and mount paths for the workspace container.
4. **Placement Optimization**: Inject node affinities, tolerations, and priority classes into the final pod manifest.
