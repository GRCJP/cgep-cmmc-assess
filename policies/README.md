# Compliance Policies (Rego)

Three OPA policies that run against `terraform plan -json` output. Each one maps to a NIST 800-53 control and blocks non-compliant resources before they get applied.

## Policies

### SC-28 — Encryption at Rest (`sc28_encryption.rego`)
- **Severity:** high
- **What it checks:** Every `google_storage_bucket` has a CMEK encryption block.
- **Remediation:** Add `encryption { default_kms_key_name = ... }` pointing to a KMS key you control.

### AC-3 — Access Enforcement (`ac3_no_public.rego`)
- **Severity:** critical
- **What it checks:** Buckets have `uniform_bucket_level_access = true` and `public_access_prevention = "enforced"`. Firewall rules don't open ports 22 or 3389 to `0.0.0.0/0`.
- **Remediation:** Lock down bucket access settings. For firewalls, narrow `source_ranges` or remove the rule.

### CM-6 — Required Labels (`cm6_required_tags.rego`)
- **Severity:** medium
- **What it checks:** Every taggable resource has four labels: `project`, `environment`, `managed_by`, `compliance_scope`.
- **Remediation:** Add the missing labels to the resource.

## Running

```bash
opa test -v policies/
opa eval -d policies -i <plan.json> data.compliance.<control>.deny --format=pretty
```
