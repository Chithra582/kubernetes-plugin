---
name: "inbound-agent-connectivity"
description: "Injects environment variables, monitors JNLP/inbound agent connection health, and manages retention timeout policies."
license: Apache-2.0
---

# Inbound Agent Connectivity

## Overview
This skill establishes and maintains secure bidirectional communication between the Jenkins controller and inbound container agents running inside Kubernetes pods.

## Key Capabilities
- **Inbound Variable Injection**: Automates injection of `JENKINS_URL`, `JENKINS_SECRET`, and `JENKINS_AGENT_NAME` into the `jnlp` inbound container.
- **Connection Timeout Guardrails**: Enforces connection deadlines (`slaveConnectTimeout`) to quickly recycle unresponsive or misconfigured pods.
- **WebSocket & TCP Protocol Support**: Supports direct TCP Remoting as well as WebSocket tunneling for network topologies with reverse proxies or ingress controllers.
- **Retention & Idle Management**: Manages pod retention policies (`idleMinutes`, `runOnce`) to reap idle agents or preserve pods for post-build debugging.

## Operational Workflow
1. **Node Handshake Setup**: Allocate node name and generate unique JNLP secret token on Jenkins controller.
2. **Container Injection**: Inject connection parameters into inbound agent container specification.
3. **Heartbeat Supervision**: Monitor connection establishment and socket liveness via `pod_lifecycle_tracker`.
4. **Timeout Enforcement**: If connection fails within timeout window, flag agent as failed and trigger immediate pod termination.
