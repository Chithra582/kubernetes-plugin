---
name: "pod-template-synthesis"
description: "Merges declarative and scripted pipeline podTemplate specifications, container definitions, and YAML manifests with cluster defaults."
license: Apache-2.0
---

# Pod Template Synthesis

## Overview
This skill implements the multi-level inheritance and merging engine that combines cloud-level defaults, pipeline declarative `podTemplate` blocks, and inline raw YAML manifests into unified Kubernetes Pod specifications.

## Key Capabilities
- **Hierarchical Inheritance**: Merges parent cloud templates with child job templates using deterministic precedence rules.
- **Container Overrides**: Updates container images, commands, and environment variables when matching container names, while preserving default inbound JNLP containers.
- **Raw YAML Overlay**: Merges multi-document or partial raw YAML manifests directly into generated pod specs.
- **Volume & Mount Resolution**: Combines host paths, emptyDirs, configMaps, and PVC volume definitions seamlessly.

## Operational Workflow
1. **Template Collection**: Retrieve default cloud template and job-specific `podTemplate` declarations.
2. **Hierarchy Resolution**: Traverse template inheritance trees from root defaults to child definitions.
3. **Spec Synthesis**: Execute `pod_template_merger` to merge containers, volumes, security contexts, and metadata.
4. **Validation**: Verify container resource requests, image formats, and required port definitions.
