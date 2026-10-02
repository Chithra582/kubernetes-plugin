# DUTIES — Kubernetes Jenkins Agent Plugin

## Core Agent Duties

### 1. Dynamic Agent Provisioning & Scaling
- Monitor Jenkins controller build queue demands and match pending jobs with matching labels.
- Calculate required pod compute capacity and verify against cloud concurrency caps (`containerCap`).
- Assemble Kubernetes Pod resource manifests and submit them to the cluster API asynchronously.

### 2. Template Synthesis & Inheritance Management
- Resolve parent-child template hierarchies between global Jenkins cloud templates and job-level `podTemplate` blocks.
- Merge custom container definitions, volume mounts, environment variables, annotations, and labels.
- Parse and overlay raw inline YAML specifications (`yaml` parameter) atop GUI-defined pod templates.

### 3. Inbound Connection & Lifecycle Supervision
- Generate unique agent node names and secure authentication secret tokens for the inbound jnlp connection.
- Monitor Kubernetes pod phases (`Pending`, `ContainerCreating`, `Running`, `Completed`, `Failed`, `Terminating`).
- Detect pod evictions, OOM kills, and image pull failures; emit diagnostic events to Jenkins build console logs.

### 4. Cluster Resource Hygiene & Clean Teardown
- Dispatch immediate Pod deletion requests upon build completion, cancellation, or slave timeout.
- Clean up transient emptyDir volumes and unbind temporary PersistentVolumeClaims.
- Maintain accurate cloud agent metrics and report cluster provisioning health.
