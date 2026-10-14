# Infrastructure as Code Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers Terraform and OpenTofu (`*.tf`, `*.tfvars`, `*.hcl`, Terragrunt). The same principles apply to AWS CloudFormation and CDK, Azure Bicep and ARM, and Pulumi.

## Authorities
- HashiCorp Terraform style guide and recommended practices
- CIS Benchmarks for AWS, Azure and GCP
- Cloud providers' well-architected frameworks, security pillar

## Tooling to look for and run
- `terraform fmt -check -recursive`, `terraform validate`
- `tflint`
- Policy scanners: `checkov`, `trivy config` (tfsec has been merged into Trivy)
- Other tools: `cfn-lint` / `cfn-nag` for CloudFormation; the Bicep linter for Bicep
- **Never** run `terraform apply` or `destroy`. Run `terraform plan` only if the user agrees, because it needs credentials and reads live state.

## Security
- `[C]` Secrets in `.tf` files, `*.tfvars` or variable defaults; `terraform.tfstate` committed (state holds secrets in plain text).
- `[C]` Security groups or firewalls open to `0.0.0.0/0` or `::/0` on SSH (22), RDP (3389) or database ports.
- `[C]` Storage made public: S3 buckets without the public access block, Azure storage with `allow_blob_public_access`, public GCS ACLs. Also flag databases with `publicly_accessible = true`.
- `[C]` IAM policies with `Action = "*"` and `Resource = "*"`, or `iam:PassRole` on `*`; wildcard principals in resource policies.
- `[H]` Encryption at rest disabled or left at default where the org requires customer-managed keys (EBS, RDS, S3, disks); TLS not enforced on load balancers or storage.
- `[H]` Remote state backend without encryption or locking (S3 without encryption or a lock; no state locking).
- `[M]` Audit logging off: CloudTrail, VPC flow logs, access logs, diagnostic settings.
- `[M]` Long-lived access keys created for CI where OIDC federation is available.

## Correctness and safety
- `[C]` A change that **forces replacement** of a stateful resource (database, bucket, volume): a renamed resource address without a `moved` block, or a change to an immutable attribute. That means data loss.
- `[H]` Stateful resources without `lifecycle { prevent_destroy = true }` or deletion protection, where siblings have it.
- `[H]` `count` used for collections whose items can be inserted or removed. The indexes shift and resources get recreated. Use `for_each` with stable keys.
- `[H]` Provider or module versions not constrained (`required_providers` with no version, a module `source` without `?ref=` or `version`).
- `[M]` Hardcoded account IDs, regions, AMI IDs or CIDRs that should be variables or data sources; `depends_on` used to cover up missing references.
- `[M]` `ignore_changes` hiding drift without a comment explaining why.

## Cost and reliability
- `[M]` Single-AZ databases or a single instance for production workloads, where siblings use multi-AZ; no backups or retention.
- `[M]` Oversized instances, or resources with no tags (when the project uses cost-allocation tags).

## Maintainability
- `[L]` Variables without a `description` or `type`; outputs without descriptions; very large root modules that should be split; naming that doesn't match the repo's convention.
