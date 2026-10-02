---
name: "pod-lifecycle-orchestration"
description: "Schedules, starts, tracks, and terminates dynamic agent pods across Kubernetes namespaces according to build workload demand."
license: Apache-2.0
---

# Pod Lifecycle Orchestration

## Overview
This skill orchestrates the end-to-end lifecycle of dynamic Jenkins agent pods in Kubernetes, from build queue detection through pod creation, phase monitoring, and deterministic teardown.

## Key Capabilities
- **Demand-Driven Provisioning**: Monitors Jenkins job queues and provisions pods matching required build labels.
- **Phase Supervision**: Tracks Kubernetes pod state transitions (`Pending`, `ContainerCreating`, `Running`, `Terminating`).
- **Failure Recovery**: Catches pod scheduling errors, image pull backoffs, and node evictions, notifying the controller to retry or fail gracefully.
- **Deterministic Teardown**: Ensures pods are fully cleaned up after build steps complete, preventing orphaned container sprawl.

## Operational Workflow
1. **Queue Matching**: Extract required node labels from pending Jenkins build jobs.
2. **Pod Creation**: Submit pod creation manifest to Kubernetes API using `pod_provisioner`.
3. **Phase Polling**: Monitor pod execution state and container readiness with `pod_lifecycle_tracker`.
4. **Teardown**: Send deletion command to Kubernetes cluster upon pipeline completion or timeout.
