---
title: "Prevent cr8escape (CVE-2022-0811) in CEL expressions"
category: Other in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    A vulnerability "cr8escape" (CVE-2022-0811) in CRI-O the container runtime engine underpinning Kubernetes allows attackers to escape from a Kubernetes container and gain root access to the host. The recommended remediation is to disallow sysctl settings with + or = in their value.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/prevent-cr8escape/prevent-cr8escape.yaml" target="-blank">/other-cel/prevent-cr8escape/prevent-cr8escape.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: prevent-cr8escape
  annotations:
    policies.kyverno.io/title: Prevent cr8escape (CVE-2022-0811) in CEL expressions
    policies.kyverno.io/category: Other in CEL 
    policies.kyverno.io/severity: high
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      A vulnerability "cr8escape" (CVE-2022-0811) in CRI-O the container runtime engine
      underpinning Kubernetes allows attackers to escape from a Kubernetes container
      and gain root access to the host. The recommended remediation is to disallow
      sysctl settings with + or = in their value.
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: restrict-sysctls-cr8escape
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
          expressions:
            - expression: >-
                object.spec.?securityContext.?sysctls.orValue([]).all(sysctl, 
                !has(sysctl.value) || (!sysctl.value.contains('+') && !sysctl.value.contains('='))) 
              message: "characters '+' or '=' are not allowed in sysctls values"      


```
