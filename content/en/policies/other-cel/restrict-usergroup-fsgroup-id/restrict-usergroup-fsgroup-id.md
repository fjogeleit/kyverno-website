---
title: "Validate User ID, Group ID, and FS Group in CEL expressions"
category: Sample in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    All processes inside a Pod can be made to run with specific user and groupID by setting `runAsUser` and `runAsGroup` respectively. `fsGroup` can be specified to make sure any file created in the volume will have the specified groupID. This policy validates that these fields are set to the defined values.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/restrict-usergroup-fsgroup-id/restrict-usergroup-fsgroup-id.yaml" target="-blank">/other-cel/restrict-usergroup-fsgroup-id/restrict-usergroup-fsgroup-id.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: validate-userid-groupid-fsgroup
  annotations:
    policies.kyverno.io/title: Validate User ID, Group ID, and FS Group in CEL expressions
    policies.kyverno.io/category: Sample in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      All processes inside a Pod can be made to run with specific user and groupID
      by setting `runAsUser` and `runAsGroup` respectively. `fsGroup` can be specified
      to make sure any file created in the volume will have the specified groupID.
      This policy validates that these fields are set to the defined values.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: validate-userid-groupid-fsgroup
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
          - expression: "object.spec.?securityContext.?runAsUser.orValue(1) == 1000"
            message: "User ID should be 1000."
          - expression: "object.spec.?securityContext.?runAsGroup.orValue(1) == 3000"
            message: "Group ID should be 3000."
          - expression: "object.spec.?securityContext.?fsGroup.orValue(1) == 2000"
            message: "fs Group should be 2000."


```
