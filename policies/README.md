# Compliance Policies (Rego)

OPA/Conftest policies that run against `terraform plan -json` output. Each one maps to a NIST 800-53 control and blocks non-compliant resources before they get applied. GCP and AWS variants share the same control IDs.

## Policies

### SC-28 — Encryption at Rest
| File | Cloud | What it checks |
|------|-------|----------------|
| `sc28_encryption.rego` | GCP | Every `google_storage_bucket` has a CMEK encryption block. |
| `sc28_encryption_aws.rego` | AWS | Every `aws_s3_bucket` has a matching `aws_s3_bucket_server_side_encryption_configuration`. |
- **Severity:** high

### AC-3 — Access Enforcement
| File | Cloud | What it checks |
|------|-------|----------------|
| `ac3_no_public.rego` | GCP | Buckets enforce uniform access and public access prevention. Firewalls don't open 22/3389 to `0.0.0.0/0`. |
| `ac3_no_public_aws.rego` | AWS | Every `aws_s3_bucket` has a `aws_s3_bucket_public_access_block` with all four flags true. |
- **Severity:** critical

### CM-6 — Required Tags/Labels
| File | Cloud | What it checks |
|------|-------|----------------|
| `cm6_required_tags.rego` | GCP | Every taggable resource has labels: `project`, `environment`, `managed_by`, `compliance_scope`. |
| `cm6_required_tags_aws.rego` | AWS | Every taggable resource has tags: `Project`, `Environment`, `ManagedBy`, `ComplianceScope`. |
- **Severity:** medium

## Running

```bash
# unit tests
opa test -v policies/

# eval against a plan
opa eval -d policies -i <plan.json> data.compliance.<control>.deny --format=pretty

# conftest gate (used by CI)
conftest test --policy policies --namespace compliance.<control> <plan.json>
bash scripts/policy-gate.sh --workspace <path>
```
