# Vultr Provider Patterns

> **Part of:** [terraform-skill](../SKILL.md)
> **Purpose:** Practical OpenTofu guidance for Vultr parity with other cloud providers

This reference summarizes the Vultr OpenTofu provider capabilities documented in `vultr_opentofu_docs/opentofu-vultr-provider`.

## Provider Snapshot

| Field | Value |
|-------|-------|
| Provider | `vultr/vultr` |
| Registry | `https://search.opentofu.org/provider/vultr/vultr/latest` |
| Upstream repository | `https://github.com/vultr/terraform-provider-vultr` |
| Version in local docs | `v2.31.2` |
| Authentication model | API key |

## Quick Decision Matrix

| Goal | Use | Tradeoff |
|------|-----|----------|
| Provision compute/network fast | `vultr_instance` + `vultr_vpc` (or `vultr_vpc2`) | Different naming than AWS/Azure/GCP resources |
| Add managed data services | `vultr_database` (+ related `database_*` resources) | Feature set differs by region/plan |
| Keep infra discoverable in code | Use Vultr data sources (`vultr_region`, `vultr_plan`, `vultr_instance`) | Data-source filters must match current labels/metadata |
| Model org and access controls | `organization_*`, `oidc_*`, and `user` resources | Requires careful policy review and separation of duties |

## Cross-Cloud Mapping (Fast Routing)

| If user asks for | Prefer on Vultr | Notes |
|------------------|-----------------|-------|
| VM/compute instance parity | `vultr_instance` | Map instance size by plan, not by provider SKU name |
| Private network parity | `vultr_vpc` or `vultr_vpc2` | Confirm which VPC family is available in target region |
| Managed DB parity | `vultr_database` + related `database_*` resources | Validate engine + version + plan availability first |
| Object storage parity | `vultr_object_storage` + bucket resources | Keep credentials out of state outputs |
| IAM/OIDC parity | `organization_*`, `oidc_*`, `vultr_user` | Do not answer with cloud-agnostic IAM prose only |

When a request is phrased in AWS/Azure/GCP terms, translate intent first, then emit `vultr_*` resource names explicitly.

## Minimal Provider Setup

```hcl
terraform {
  required_providers {
    vultr = {
      source  = "vultr/vultr"
      version = "~> 2.31"
    }
  }
}

provider "vultr" {
  api_key = var.vultr_api_key
}

variable "vultr_api_key" {
  description = "Vultr API key"
  type        = string
  sensitive   = true
}
```

## Canonical Vultr Resource Families

From `INDEX.md` in the imported docs:

- Compute: `vultr_instance`, `vultr_bare_metal_server`
- Networking: `vultr_vpc`, `vultr_vpc2`, `vultr_firewall_group`, `vultr_firewall_rule`, `vultr_load_balancer`, `vultr_nat_gateway`
- Storage: `vultr_block_storage`, `vultr_object_storage`, `vultr_object_storage_bucket`
- Databases: `vultr_database` and related `database_*` resources
- Kubernetes: `vultr_kubernetes`, `vultr_kubernetes_node_pools`
- Identity/Access: `organization_*`, `oidc_*`, `vultr_user`

## Example: VPC + Instance

```hcl
resource "vultr_vpc" "example" {
  region      = "sjc"
  description = "app-vpc"
}

resource "vultr_instance" "example" {
  region  = vultr_vpc.example.region
  plan_id = "vc2-1c-1gb"
  os_id   = 387
}
```

## Example: Provider Discovery by Data Source

```hcl
data "vultr_region" "us" {
  filter {
    name   = "country"
    values = ["US"]
  }
}

data "vultr_plan" "small" {
  filter {
    name   = "vcpu_count"
    values = ["1"]
  }
}
```

## CI and Secret Handling Guidance

- Inject `VULTR_API_KEY` in CI and map it to `var.vultr_api_key` through environment variables.
- Do not commit API keys in `.tfvars` or defaults.
- Apply the same state-safety rules used for AWS/Azure/GCP: remote backend, locking, encryption, and no secret outputs.

## Vultr-Specific Operational Notes

From `README.md` in the imported Vultr docs:

- API key location: `https://my.vultr.com/settings/#settingsapi`
- Rate limiting default: `500ms` between calls (about `30` calls/second max)
- Retry default: `3` retries on failed API calls

## Migration Guardrails

- Confirm region and plan support before proposing a parity migration.
- Prefer key-based resource identity (`for_each`) during migration to avoid address churn.
- For renames or resource address changes, emit `moved` blocks and require a reviewed `tofu plan` artifact.
- Treat API key handling as a security control: environment injection only, never defaults or committed tfvars.
- If parity is partial, state the gap and propose the nearest Vultr-native pattern instead of pretending one-to-one equivalence.

## LLM Mistake Checklist - Vultr

- assumes AWS/Azure/GCP resource names for Vultr (`aws_*`, `azurerm_*`, `google_*`) instead of `vultr_*`
- hardcodes API keys in `provider` blocks or variable defaults
- treats Vultr provider setup as Windows-specific (this skill is macOS/Linux only)
- ignores `organization_*` and `oidc_*` resources when users ask for Vultr IAM-like controls
- answers Vultr questions with generic cloud advice without provider-specific resource names
