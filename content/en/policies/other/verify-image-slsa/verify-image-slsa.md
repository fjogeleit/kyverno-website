---
title: "Verify SLSA Provenance (Keyless)"
category: Software Supply Chain Security
version: 1.8.3
subject: Pod
policyType: "verifyImages"
description: >
    Provenance is used to identify how an artifact was produced and from where it originated. SLSA provenance is an industry-standard method of representing that provenance. This policy verifies that an image has SLSA provenance and was signed by the expected subject and issuer when produced through GitHub Actions. It requires configuration based upon your own values.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/verify-image-slsa/verify-image-slsa.yaml" target="-blank">/other/verify-image-slsa/verify-image-slsa.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-slsa-provenance-keyless
  annotations:
    policies.kyverno.io/title: Verify SLSA Provenance (Keyless)
    policies.kyverno.io/category: Software Supply Chain Security
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.8.3
    kyverno.io/kyverno-version: 1.9.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/description: >-
      Provenance is used to identify how an artifact was produced
      and from where it originated. SLSA provenance is an industry-standard
      method of representing that provenance. This policy verifies that an
      image has SLSA provenance and was signed by the expected subject and issuer
      when produced through GitHub Actions. It requires configuration based upon
      your own values.
spec:
  validationFailureAction: Audit
  webhookTimeoutSeconds: 30
  rules:
    - name: check-slsa-keyless
      match:
        any:
        - resources:
            kinds:
              - Pod
      verifyImages:
      - imageReferences:
        - "myreg.org/path/repo:*"
        attestations:
        - predicateType: https://slsa.dev/provenance/v0.2
          attestors:
          - count: 1
            entries:
            - keyless:
                subject: "https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v*"
                issuer: "https://token.actions.githubusercontent.com"
                rekor:
                  url: https://rekor.sigstore.dev
          conditions:
          - all:
            # This expression uses a regex pattern to ensure the builder.id in the attestation is equal to the official
            # SLSA provenance generator workflow and uses a tagged release in semver format. If using a specific SLSA
            # provenance generation workflow, you may need to adjust the first input as necessary.
            - key: "{{ regex_match('^https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v[0-9].[0-9].[0-9]$','{{ builder.id}}') }}"
              operator: Equals
              value: true

```
