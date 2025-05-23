---
title: "Check Tekton TaskRun Vulnerability Scan"
category: Tekton
version: 1.7.0
subject: TaskRun
policyType: "verifyImages"
description: >
    A signed bundle is required and a vulnerability scan made by Grype must return no vulnerabilities greater than 8.0.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//tekton/verify-tekton-taskrun-vuln-scan/verify-tekton-taskrun-vuln-scan.yaml" target="-blank">/tekton/verify-tekton-taskrun-vuln-scan/verify-tekton-taskrun-vuln-scan.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-tekton-taskrun-vuln-scan
  annotations:
    policies.kyverno.io/title: Check Tekton TaskRun Vulnerability Scan
    policies.kyverno.io/category: Tekton
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: TaskRun
    kyverno.io/kyverno-version: 1.7.2
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      A signed bundle is required and a vulnerability scan made by Grype must
      return no vulnerabilities greater than 8.0.
spec:
  validationFailureAction: Audit
  webhookTimeoutSeconds: 30
  rules:
  - name: check-signature
    match:
      any:
      - resources:
          kinds:
          - tekton.dev/v1beta1/TaskRun.status 
    imageExtractors:
      TaskRun:
        - name: "taskrunstatus"
          path: "/status/taskSpec/steps/*"
          value: "image"
          key: "name"
    verifyImages:
    - imageReferences:
      - "*"
      attestations:
      - predicateType: https://grype.anchore.io/scan
        attestors:
        - entries:
          - keys:
              publicKeys: |-
                -----BEGIN PUBLIC KEY-----
                MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEahmSvGFmxMJABilV1usgsw6ImcQ/
                gDaxw57Sq+uNGHW8Q3zUSx46PuRqdTI+4qE3Ng2oFZgLMpFN/qMrP0MQQg==
                -----END PUBLIC KEY-----          
        conditions:
        - any:
          - key: "{{ matches[].vulnerability[].cvss[?metrics.impactScore > '8.0'][] | length(@) }}"
            operator: Equals
            value: 0
          - key: "{{ source.target.userInput }}"
            operator: Equals
            value: "ghcr.io/tap8stry/git-init:v0.21.0@sha256:322e3502c1e6fba5f1869efb55cfd998a3679e073840d33eb0e3c482b5d5609b"
```
