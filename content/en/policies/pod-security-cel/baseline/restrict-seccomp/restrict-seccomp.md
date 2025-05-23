---
title: "Restrict Seccomp in CEL expressions"
category: Pod Security Standards (Baseline) in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    The seccomp profile must not be explicitly set to Unconfined. This policy,  requiring Kubernetes v1.19 or later, ensures that seccomp is unset or  set to `RuntimeDefault` or `Localhost`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-cel/baseline/restrict-seccomp/restrict-seccomp.yaml" target="-blank">/pod-security-cel/baseline/restrict-seccomp/restrict-seccomp.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-seccomp
  annotations:
    policies.kyverno.io/title: Restrict Seccomp in CEL expressions
    policies.kyverno.io/category: Pod Security Standards (Baseline) in CEL
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      The seccomp profile must not be explicitly set to Unconfined. This policy, 
      requiring Kubernetes v1.19 or later, ensures that seccomp is unset or 
      set to `RuntimeDefault` or `Localhost`.
spec:
  background: true
  validationFailureAction: Audit
  rules:
    - name: check-seccomp
      match:
        any:
        - resources:
            kinds:
              - Pod
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          variables:
            - name: allContainers
              expression: "(object.spec.containers + (has(object.spec.initContainers) ? object.spec.initContainers : []) + (has(object.spec.ephemeralContainers) ? object.spec.ephemeralContainers : []))"
            - name: allowedProfileTypes
              expression: "['RuntimeDefault', 'Localhost']"
          expressions:
            - expression: >- 
                (object.spec.?securityContext.?seccompProfile.?type.orValue('Localhost') 
                in variables.allowedProfileTypes) && 
                (variables.allContainers.all(container, 
                container.?securityContext.?seccompProfile.?type.orValue('Localhost') 
                in variables.allowedProfileTypes))
              message: >-
                Use of custom Seccomp profiles is disallowed. The field
                spec.containers[*].securityContext.seccompProfile.type must be unset or set to `RuntimeDefault` or `Localhost`.

```
