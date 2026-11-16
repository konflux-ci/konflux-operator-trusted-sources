# Konflux Operator Trusted Sources

This repository stores Conforma (Enterprise Contract) policy data used by the
[Konflux operator](https://github.com/konflux-ci/konflux-ci)'s default
`EnterpriseContractPolicy`. The operator references this data via a git ref in
its policy manifest.

## Data files

The `data/` directory contains rule data consumed by Conforma during policy
evaluation.

| File | Purpose |
|------|---------|
| `trusted_task_rules.yaml` | [ADR 0053](https://github.com/konflux-ci/architecture/blob/main/ADR/0053-trusted-task-model.md) rule-based trust model — allow/deny patterns and version constraints for Tekton tasks. Sets `trusted_task_rules_enabled: true`. |
| `trusted_task_rules_deprecated.yaml` | Allow/deny rules for deprecated task locations (`integration-service-catalog`, `konflux-vanguard`) during migration to `tekton-catalog`. |
| `rule_data.yml` | General rule configuration — allowed registries, required labels, informative tests, RPM signature keys. |
| `required_tasks.yml` | Required tasks for the `docker-build-oci-ta-min` pipeline, including `-min` task variants. |

### Trust model

The new trust model (ADR 0053) replaces the old digest-based "acceptable
bundles" approach. Instead of matching exact image digests, task trust is
determined by OCI registry patterns and version constraints defined in
`trusted_task_rules.yaml`. The `trusted_task_rules_enabled: true` flag
activates this model.

Tasks from `quay.io/konflux-ci/tekton-catalog/` and
`quay.io/konflux-ci/integration-service-catalog/` are trusted by default.
Version-based deny rules in the same file enforce minimum task versions and
expire old ones on scheduled dates.

### How the operator uses this data

The operator's default `EnterpriseContractPolicy` references this repository
as a git data source:

```yaml
sources:
- data:
  - github.com/konflux-ci/konflux-operator-trusted-sources//data?ref=<sha-or-branch>
  config:
    include:
    - '@redhat'
```

Conforma loads all YAML files from the `data/` directory and merges them into
the policy evaluation context.

### CI validation

A GitHub Actions workflow validates rule data format on pull requests using
the Conforma CLI with the `@policy_data` rule collection. See `ci-policy.yaml`
for the policy configuration.
