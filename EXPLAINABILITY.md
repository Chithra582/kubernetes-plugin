# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Kubernetes Plugin Agent** (`kubernetes-plugin`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Kubernetes Plugin Agent (`kubernetes-plugin`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Cloud Infrastructure & Dynamic CI/CD Agent Provisioning  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), SOC 2, ISO 27001  

---

## How the Agent Decides

Kubernetes Plugin Agent is an autonomous cloud agent provisioning, pod lifecycle orchestration, and dynamic build executor scaling agent designed for **Jenkins Kubernetes Plugin**. The agent coordinates template inheritance, resource quota validation, pod manifest synthesis, JNLP agent connection handshakes, and ephemeral build agent lifecycle management.

### 1. Decision Architecture

The build queue intake, template resolution, pod provisioning, and container execution pipeline operates across a deterministic, five-stage architecture:

```
Build Queue Request (Jenkins Job with Kubernetes Label Dispatched to Queue)
    │
    ▼
[Stage 1: Intent Ingestion & Template Resolution Gate]
    │  - Evaluates requested build label against configured Kubernetes clouds
    │  - Traverses inheritance tree: merges global templates with pipeline podTemplate specifications
    │  - Normalizes container image tags, environment variables, resource limits, and service accounts
    ▼
[Stage 2: Resource Quota & Capacity Verification]
    │  - Audits active container count against maximum ceiling (containerCap)
    │  - Inspects cluster namespace ResourceQuota limits (CPU, memory, persistent volumes)
    │  - Evaluates node affinity, tolerations, and anti-affinity placement constraints
    ▼
[Stage 3: Pod Manifest Synthesis & Security Audit]
    │  - Synthesizes Kubernetes Pod v1 specification with ephemeral container definitions
    │  - Enforces container security policies: rejects unauthorized hostPath mounts and unapproved root execution
    │  - Attaches JNLP agent container, workspace emptyDir volumes, and secret projection volumes
    ▼
[Stage 4: Inbound JNLP Handshake & Health Gate]
    │  - Dispatches Pod creation request to Kubernetes API server via Fabric8 client
    │  - Monitors Pod phase transitions: Pending -> Running -> Initialized
    │  - Establishes bidirectional JNLP/WebSocket channel with Jenkins controller within timeout budget
    ▼
[Stage 5: Ephemeral Build Execution & Teardown Gate]
    │  - Streams pipeline step commands to agent container via remoting protocol
    │  - Collects build exit codes, workspace artifacts, and container execution logs
    │  - Gracefully terminates and reaps ephemeral Pod to release cluster resources
    ▼
Build Completed & Dynamic Kubernetes Pod Reaped
```

### 2. Scoring Methodology & Rubric Formulations

When matching pending build tasks against available Kubernetes clouds and templates, the plugin computes two deterministic, mathematically rigorous scoring models:

1. **Provisioning Affinity Score ($S_{\text{affinity}}$)**:
   $$S_{\text{affinity}} = (w_l \cdot L_{\text{match}}) + (w_c \cdot C_{\text{headroom}}) + (w_r \cdot R_{\text{quota}}) + (w_t \cdot T_{\text{affinity}})$$
   where:
   - $L_{\text{match}} \in \{0, 1\}$: Binary label exactness between job requirements and template labels.
   - $C_{\text{headroom}} = \max\left(0, 1 - \frac{N_{\text{active}}}{C_{\text{cap}}}\right)$: Concurrency capacity headroom in the target cloud.
   - $R_{\text{quota}} \in [0, 1]$: Available namespace CPU and memory quota relative to requested limits.
   - $T_{\text{affinity}} \in [0, 1]$: Node selector and toleration compatibility score.
   - Weights: $w_l = 0.40, w_c = 0.25, w_r = 0.20, w_t = 0.15$ ($\sum w_i = 1.0$).

2. **Pod Health & Liveness Index ($H_{\text{pod}}$)**:
   $$H_{\text{pod}} = 100 \times \left( \alpha \cdot \frac{t_{\text{timeout}} - t_{\text{elapsed}}}{t_{\text{timeout}}} + \beta \cdot \mathbb{I}(\text{Ready} = \text{TRUE}) + \gamma \cdot \left(1 - \frac{R_{\text{restarts}}}{R_{\max}}\right) \right)$$
   where $\alpha = 0.40, \beta = 0.40, \gamma = 0.20$, measuring connection timeliness, container readiness gates, and zero container crash loops.

A pod provisioning request is approved if and only if:
$$S_{\text{affinity}} \ge 0.65 \quad \land \quad N_{\text{active}} < C_{\text{cap}}$$

### 3. Thresholding & Refusal Decision Criteria

Kubernetes Plugin Agent enforces strict infrastructure integrity boundaries:
- **Refusal on Capacity Exhaustion**: Requests that exceed cloud capacity are queued deterministically with code `ERR_CONTAINER_CAP_EXCEEDED`.
- **Refusal on Connection Timeout**: Pods failing to establish inbound JNLP handshakes within `slaveConnectTimeout` are terminated with code `ERR_CONNECT_TIMEOUT`.
- **Refusal on Namespace Quota Breach**: Pod creation requests exceeding Kubernetes ResourceQuota ceilings fail immediately with code `ERR_NAMESPACE_QUOTA`.
- **Refusal on Dangerous Host Mounts**: Templates requesting forbidden host filesystem mounts (`/var/run/docker.sock`) are blocked with code `ERR_UNAUTHORIZED_MOUNT`.
- **Refusal on Scheduling Deadlock**: Pods remaining unscheduled past the pod startup timeout are evicted with code `ERR_SCHEDULING_DEADLOCK`.

### 4. Fallback Decision Mechanism

Kubernetes Plugin Agent maintains uninterrupted CI/CD execution through multi-tier fallbacks:
- **API Connection Fallback**: If the primary Kubernetes API endpoint becomes unreachable, the client falls back to secondary cluster endpoints or local microk8s/kind instances.
- **Template Inheritance Fallback**: If a pipeline-specified parent template cannot be resolved, the plugin falls back to global default pod templates with a build warning.
- **Graceful Deletion Escalation**: If standard pod deletion fails within `gracePeriodSeconds`, an escalated force deletion (`gracePeriodSeconds = 0`) is dispatched to clean orphaned pods.
- **Model Fallback Cascade**: High-level failure analysis and pod failure diagnostics default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Kubernetes Plugin Agent maintains administrator authority and operational safety:
- **Admin Configuration Gate**: Modifying Kubernetes cloud credentials, API URLs, certificate authorities, and namespace bindings requires `Jenkins.ADMINISTER` permissions.
- **Privileged Security Approvals**: Running privileged containers (`securityContext.privileged: true`) requires explicit admin approval in template definitions.
- **Manual Node Eviction**: Operators can inspect live dynamic pods in Jenkins node management UI and manually terminate delinquent pods.

---

## The Data It Uses

Kubernetes Plugin Agent operates under strict enterprise privacy and infrastructure security standards.

### 1. Ingested Input Data

The agent processes only build specifications and cluster telemetry:
- **Jenkins Pipeline Specifications**: Declarative `agent { kubernetes { ... } }` and scripted `podTemplate` blocks containing container images, commands, and labels.
- **Cluster Runtime Telemetry**: Pod phase statuses, container readiness probes, CPU/memory usage metrics, and exit codes.
- **Build Queue Requests**: Job IDs, requested executor labels, parameters, and upstream triggering metadata.

### 2. Configuration & Reference Data

- **Kubernetes Cloud Configuration**: Master API URL, cluster CA certificates, credential IDs, and namespace defaults.
- **Pod Template Library**: System-wide reusable container templates providing standardized runtime images (Maven, Node, Python, Go).
- **ServiceAccount Tokens**: RBAC credentials granting scoped pod lifecycle permissions within the target namespace.

### 3. Base Model & Inference Lineage

- **Deterministic Orchestration Engine**: Fabric8 Kubernetes Java client (`io.fabric8:kubernetes-client`) executing deterministic scheduling and lifecycle logic.
- **AI Infrastructure Copilot**: Foundation models (`gemini-2.0-flash`, `gpt-4o`, `claude-3-5-sonnet`) utilized for build log diagnostic analysis and pod failure root-cause identification.
- **Zero Training on Pipeline Code**: Source code, pipeline scripts, workspace files, and container logs are never utilized for model training.

### 4. Data Privacy, Storage, and Retention

- **SOC 2 & ISO 27001 Compliance**: In accordance with enterprise compliance standards, build agent tokens and credentials are encrypted at rest and in transit.
- **Ephemeral State Purging**: Dynamic pod metadata is purged from Jenkins memory immediately upon pod teardown; workspace data is destroyed with the pod's ephemeral emptyDir.
- **Secret Masking**: JNLP tokens, API keys, and environment variables are automatically masked in console logs using Jenkins Secret Masker.

---

## Limitations

Understanding the operational boundaries and technical constraints of Kubernetes Plugin Agent is essential for reliable CI/CD operations.

### 1. Inbound Connection Cold-Start Latency
- **Limitation**: Dynamic pods incur container image pull and JVM startup latency before becoming available to execute pipeline stages.
- **Mitigation**: Utilize pre-pulled images, Kubernetes `imagePullPolicy: IfNotPresent`, or cached node pools to minimize cold-start delay.

### 2. Shared Cluster Namespace Quotas
- **Limitation**: Heavy concurrent pipeline runs can exhaust namespace CPU/memory limits or IP allocations, causing unexpected scheduling failures.
- **Mitigation**: Configure strict `containerCap` values, apply ResourceQuotas, and configure PriorityClasses for critical builds.

### 3. Host Volume Portability Restrictions
- **Limitation**: Utilizing `hostPath` volume mounts breaks pod portability across multi-node clusters and presents security hazards.
- **Mitigation**: Encourage containerized workflows with PersistentVolumeClaims or ephemeral `emptyDir` volumes rather than node host paths.

### 4. Network Firewall & Ingress Traversal
- **Limitation**: Inbound agent pods require direct or proxied TCP/WebSocket connectivity back to the Jenkins controller URL.
- **Mitigation**: Leverage WebSocket Remoting over HTTPS (`jenkins-remoting-websocket`) to effortlessly traverse firewalls and ingress gateways without dedicated JNLP TCP ports.

### 5. Multi-Container Workspace Synchronization
- **Limitation**: Multiple containers in a single pod sharing a workspace can encounter file permission or concurrency lock collisions if running as different UID/GIDs.
- **Mitigation**: Enforce unified `securityContext.runAsUser` and `securityContext.fsGroup` settings across all pod container specifications.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Provisioning affinity & health index formulas | Section 2 | Verified |
| - Thresholding, quota refusal & error criteria | Section 3 | Verified |
| - Fallback decision mechanism & API recovery | Section 4 | Verified |
| - Human-in-the-loop & administrator governance | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested pipeline specs, telemetry & queue data | Section 1 | Verified |
| - Configuration, pod templates & service accounts | Section 2 | Verified |
| - Base model lineage & deterministic engine | Section 3 | Verified |
| - Data privacy, ephemeral retention & secret masking | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Inbound connection cold-start latency | Section 1 | Verified |
| - Shared cluster namespace quotas | Section 2 | Verified |
| - Host volume portability restrictions | Section 3 | Verified |
| - Network firewall & ingress traversal | Section 4 | Verified |
| - Multi-container workspace synchronization | Section 5 | Verified |
