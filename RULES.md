# RULES — Kubernetes Jenkins Agent Plugin

## Operational Rules & Guardrails
1. **Mandatory Inbound Connection Verification**: Every provisioned agent pod must successfully connect to the Jenkins controller within the configured `slaveConnectTimeout` (default: 100 seconds); failure to establish connection triggers automatic pod termination and queue rescheduling.
2. **Deterministic Pod Inheritance**: When merging pod templates, container definitions sharing identical names must override fields hierarchically (Job Template > Cloud Parent Template > Global Defaults), while distinct container names must be appended.
3. **Strict Namespace Isolation**: Pods must strictly be scheduled within the configured Kubernetes namespace; cross-namespace pod creation without explicitly authorized multi-cloud configuration is forbidden.
4. **Mandatory Graceful Termination**: Completed or aborted builds must trigger graceful pod deletion with configurable termination grace periods, followed by force deletion if pods enter `Terminating` deadlock.
5. **Resource Request & Limit Enforcement**: Every container defined in a pod template should specify CPU and memory requests and limits to guarantee predictable Kubernetes scheduling and protect cluster node health.
6. **Credential Masking in Pod Specs**: Secret keys, Jenkins inbound secrets, and authentication bearer tokens must never be written as plain text in pod specs; they must be referenced via Kubernetes `SecretKeyRef` or injected directly via secure runtime channels.
7. **Concurrency Ceiling Enforcement**: The total number of concurrently running agent pods across all clouds must never exceed the administrator-defined `containerCap`; excess queue items must remain queued until running pods terminate.
