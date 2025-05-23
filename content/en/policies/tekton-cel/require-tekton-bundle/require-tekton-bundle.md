---
title: "Require Tekton Bundle in CEL expressions"
category: Tekton in CEL
version: 1.11.0
subject: TaskRun, PipelineRun
policyType: "validate"
description: >
    PipelineRun and TaskRun resources must be executed from a bundle
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//tekton-cel/require-tekton-bundle/require-tekton-bundle.yaml" target="-blank">/tekton-cel/require-tekton-bundle/require-tekton-bundle.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-tekton-bundle
  annotations:
    policies.kyverno.io/title: Require Tekton Bundle in CEL expressions
    policies.kyverno.io/category: Tekton in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: TaskRun, PipelineRun
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      PipelineRun and TaskRun resources must be executed from a bundle
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: check-bundle-pipelinerun
    match:
      any:
      - resources:
          kinds:
          - PipelineRun
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "object.spec.?pipelineRef.?bundle.orValue('') != ''"
            message: "A bundle is required."
  - name: check-bundle-taskrun
    match:
      any:
      - resources:
          kinds:
          - TaskRun
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "object.spec.?taskRef.?bundle.orValue('') != ''"
            message: "A bundle is required."


```
