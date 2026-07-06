# Evidence Chain of Custody

## How the pipeline proves each property

**Authenticity** — Cosign signs every evidence bundle using a keyless workflow tied to GitHub's OIDC token. The signature proves the bundle came from my pipeline in this repo, not from someone else. The `sig.bundle` file contains the certificate and signature. Verification uses `cosign verify-blob` against GitHub's OIDC issuer.

**Integrity** — A SHA-256 hash is computed at signing time and stored alongside the bundle as a `.sha256` sidecar. If even one byte changes, the hash won't match. The verify script recomputes the hash and compares — any mismatch means the evidence was tampered with.

**Timeliness** — Cosign logs every signature to Sigstore's Rekor transparency log. That log entry has a timestamp that can't be backdated. This proves when the evidence was signed, not just that it was signed.

**Preservation** — Signed bundles are uploaded to an S3 vault with Object Lock enabled (GOVERNANCE mode, 1-day retention for lab; COMPLIANCE mode with longer retention for production). Object Lock prevents deletion or overwrite during the retention period. Even an admin can't remove the evidence until retention expires.

## Verification

Run `scripts/verify-evidence.sh <run_id> --vault <bucket>` against any pipeline run. It checks all four properties and returns `CHAIN INTACT` or a specific failure reason.
