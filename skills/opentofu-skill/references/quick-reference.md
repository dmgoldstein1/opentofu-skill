# Quick Reference

> **Part of:** [terraform-skill](../SKILL.md)
> **Purpose:** OpenTofu command cheat sheets and decision flowcharts for macOS and Linux

This document provides quick lookup tables, command references, and decision flowcharts for rapid consultation during development.

---

## Table of Contents

1. [Command Cheat Sheet](#command-cheat-sheet)
2. [Decision Flowchart](#decision-flowchart)
3. [Version-Specific Guidance](#version-specific-guidance)
4. [Troubleshooting Guide](#troubleshooting-guide)
5. [Migration Paths](#migration-paths)

---

## Command Cheat Sheet

### Static Analysis

Use `tofu` commands throughout:

```bash
# Format and validate
tofu fmt -recursive -check
tofu validate

# Linting
tflint --init && tflint

# Security scanning
checkov -d .
```

### Native Tests (1.6+)

```bash
# Run all tests
tofu test

# Run tests in specific directory
tofu test -test-directory=tests/unit/

# Verbose output
tofu test -verbose
```

### Plan Validation

```bash
# Generate and review plan
tofu plan -out tfplan

# Convert plan to pretty JSON
tofu show -json tfplan | jq -r '.' > tfplan.json

# Check for specific changes
tofu show tfplan | grep "will be created"
```

### State Management

```bash
# View all resources in state
tofu state list

# Show specific resource details
tofu state show aws_instance.web

# Move/rename resource in state (refactoring)
tofu state mv aws_instance.old aws_instance.new
tofu state mv aws_instance.app module.compute.aws_instance.app

# Remove resource from state (keeps actual resource)
tofu state rm aws_instance.temporary

# Import existing resource into state
tofu import aws_instance.web i-1234567890abcdef0

# Import using import blocks (1.5+)
# Define in .tf: import { to = aws_instance.web, id = "i-123..." }
# Note: File must not exist — the CLI refuses to overwrite.
tofu plan -generate-config-out=imported.tf

# Detect configuration drift
tofu plan -refresh-only

# Update state to match reality (no infrastructure changes)
tofu apply -refresh-only

# Backup state to file
tofu state pull > backup-$(date +%Y%m%d).tfstate

# Restore state from backup (DANGEROUS)
tofu state push backup.tfstate

# Force unlock stuck state lock
# Default: prompts for y/N confirmation
tofu force-unlock LOCK_ID

# CI-friendly (skips prompt):
tofu force-unlock -force LOCK_ID
```

### State Backend Migration

```bash
# Migrate from local to remote backend
# 1. Add backend config to backend.tf
# 2. Run migration
tofu init -migrate-state

# Change backend without migrating state
tofu init -reconfigure

# Pass backend config at runtime
tofu init \
  -backend-config="key=prod/terraform.tfstate" \
  -backend-config="dynamodb_table=terraform-locks"

# Or use config file
tofu init -backend-config=backend-prod.hcl
```

---

## Decision Flowchart

### Testing Approach Selection

```
Need to test OpenTofu code?
│
├─ Just syntax/format?
│  └─ tofu validate + fmt
│
├─ Static security scan?
│  └─ trivy + checkov
│
├─ OpenTofu 1.6+?
│  ├─ Simple logic test?
│  │  └─ Native tofu test
│  │
│  └─ Complex integration?
│     └─ Terratest
│
└─ Pre-1.6?
   ├─ Go team?
   │  └─ Terratest
   │
   └─ Neither?
   └─ Plan to upgrade OpenTofu
```

### Module Development Workflow

```
1. Plan
   ├─ Define inputs (variables.tf)
   ├─ Define outputs (outputs.tf)
   └─ Document purpose (README.md)

2. Implement
   ├─ Create resources (main.tf)
   ├─ Pin versions (versions.tf)
   └─ Add examples (examples/simple, examples/complete)

3. Test
   ├─ Static analysis (validate, fmt, lint)
   ├─ Unit tests (native or Terratest)
   └─ Integration tests (examples/)

4. Document
   ├─ Update README with usage
   ├─ Document inputs/outputs
   └─ Add CHANGELOG

5. Publish
   ├─ Tag version (git tag v1.0.0)
   ├─ Push to registry
   └─ Announce changes
```

---

## Version-Specific Guidance

### OpenTofu 1.6+

- ✅ Native `tofu test` framework with `.tftest.hcl` files
- ✅ Consider migrating simple tests from Terratest
- ✅ Keep Terratest for complex integration
- ✅ Import blocks from 1.5 available for declarative imports with `-generate-config-out`

### OpenTofu 1.7+

- ✅ NEW: Mock providers for unit testing
- ✅ Reduce costs with mocking
- ✅ Use real integration tests for final validation
- ✅ Faster test iteration

---

## Troubleshooting Guide

### Issue: Tests fail in CI but pass locally

**Symptoms:**
- Tests pass on your machine
- Same tests fail in GitHub Actions/GitLab CI

**Common Causes:**
1. Different OpenTofu/provider versions
2. Different environment variables
3. Different AWS credentials/permissions

**Solution:**

```hcl
# versions.tf - Pin versions explicitly
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"  # Pin to major version
    }
  }
}
```

### Issue: Parallel tests conflict

**Symptoms:**
- Tests fail when run in parallel
- Error: "ResourceAlreadyExistsException"

**Cause:** Resource naming collisions

**Solution:**

```go
// Use unique identifiers
import "github.com/gruntwork-io/terratest/modules/random"

uniqueId := random.UniqueId()
bucketName := fmt.Sprintf("test-bucket-%s", uniqueId)
```

### Issue: High test costs

**Symptoms:**
- AWS bill increasing from tests
- Many orphaned resources in test account

**Solutions:**

1. **Use mocking for unit tests** (OpenTofu 1.7+)
   ```hcl
   mock_provider "aws" { ... }
   ```

2. **Implement resource TTL tags**
   ```go
   Vars: map[string]interface{}{
       "tags": map[string]string{
           "Environment": "test",
           "TTL":         "2h",
       },
   }
   ```

3. **Run integration tests only on main branch**
   ```yaml
   if: github.ref == 'refs/heads/main'
   ```

4. **Use smaller instance types**
   ```hcl
   instance_type = "t3.micro"  # Not "m5.large"
   ```

5. **Share test resources when safe**
   - VPCs, security groups (rarely change)
   - Don't share: instances, databases (change often)

### Issue: State lock is stuck

**Symptoms:**
```
Error: Error acquiring the state lock
Lock Info:
  ID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
  Who: user@hostname
  Created: 2026-01-20 12:00:00
```

**Common Causes:**
1. OpenTofu process crashed or was killed
2. Network interruption during operation
3. CI/CD job terminated unexpectedly

**Solution:**

```bash
# 1. Verify the operation is NOT actually running
# Check the host mentioned in lock info
ssh user@hostname "ps aux | grep terraform"

# Or check CI/CD job status
# GitHub Actions: Check workflow runs
# GitLab CI: Check pipeline jobs

# 2. Only if confirmed the operation is not running:
terraform force-unlock LOCK_ID

# 3. Document why you unlocked
echo "Force-unlocked due to CI job timeout" > unlock-notes.txt
```

**Prevention:**
```yaml
# GitHub Actions - Use concurrency control
concurrency:
  group: terraform-${{ github.ref }}
  cancel-in-progress: false  # Wait, don't cancel
```

### Issue: State file is corrupted or lost

**Symptoms:**
- Error: "state snapshot was created by OpenTofu v1.8.0"
- Error: "Failed to load state"
- State file missing or unreadable

**Solutions:**

**If versioning enabled (S3):**
```bash
# List versions
aws s3api list-object-versions \
  --bucket my-terraform-state \
  --prefix prod/terraform.tfstate

# Restore previous version
aws s3api get-object \
  --bucket my-terraform-state \
  --key prod/terraform.tfstate \
  --version-id PREVIOUS_VERSION_ID \
  terraform.tfstate.restored

# Push restored state
tofu state push terraform.tfstate.restored
```

**If no backup exists:**
```bash
# Recreate state by importing all resources
tofu import aws_vpc.main vpc-12345678
tofu import aws_subnet.private[0] subnet-abcd1234
# ... continue for all resources

# Or use import blocks (1.5+)
# In .tf file:
# import { to = aws_vpc.main, id = "vpc-12345678" }
# Note: File must not exist — the CLI refuses to overwrite.
tofu plan -generate-config-out=imported.tf
```

### Issue: Configuration drift detected

**Symptoms:**
```
Note: Objects have changed outside of OpenTofu
```

**Cause:** Manual changes in console or by other tools

**Solutions:**

```bash
# View drift
tofu plan -refresh-only

# Accept drift (update state to match reality)
tofu apply -refresh-only

# Or fix drift (update resources to match config)
tofu apply

# Prevent drift with detective controls
# - Enable CloudTrail
# - Use AWS Config rules
# - Regular tofu plan in CI
```

### Issue: Cannot migrate state between backends

**Symptoms:**
- `tofu init -migrate-state` fails
- Backend authentication errors

**Solutions:**

```bash
# Ensure credentials are configured
export AWS_PROFILE=terraform
# or
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...

# Try migration again
tofu init -migrate-state

# If still failing, manual migration:
# 1. Pull state from old backend
tofu state pull > old-state.json

# 2. Switch backend config
# Edit backend.tf

# 3. Initialize new backend
tofu init -reconfigure

# 4. Push state to new backend
tofu state push old-state.json
```

---

## Migration Paths

### From Manual Testing → Automated

**Phase 1:** Static analysis
```bash
tofu validate
tofu fmt -check
```

**Phase 2:** Plan review
```bash
tofu plan -out=tfplan
# Manual review
```

**Phase 3:** Automated tests
- Native tests (1.6+)
- OR Terratest

**Phase 4:** CI/CD integration
- GitHub Actions / GitLab CI
- Automated apply on main branch

### From Terratest → Native Tests (1.6+)

**Strategy:** Gradual migration

1. **Keep Terratest for:**
   - Complex integration tests
   - Multi-step workflows
   - Cross-provider tests

2. **Migrate to native tests:**
   - Simple unit tests
   - Logic validation
   - Mock-friendly tests

3. **During transition:**
   - Maintain both frameworks
   - Gradually increase native test coverage
   - Remove Terratest tests once replaced

**Example:** Mixed approach

```
tests/
├── unit/                    # Native tests
│   └── validation.tftest.hcl
└── integration/             # Terratest
    └── complete_test.go
```

### OpenTofu Upgrade Path

1. **Pin the runtime** — declare the target OpenTofu version in `required_version`.
2. **Standardize CI/CD** — use `tofu` consistently in `init`/`plan`/`apply` invocations.
3. **Refresh docs** — keep examples, workflow templates, and local commands aligned to the same OpenTofu version floor.

---

## Pre-Commit Checklist

### Formatting & Validation

Run these commands before every commit:

```bash
# Format all OpenTofu files
tofu fmt -recursive

# Validate configuration
tofu validate
```

### Naming Convention Review

- [ ] All identifiers use `_` not `-`
- [ ] No resource names repeat resource type (no `aws_vpc.main_vpc`)
- [ ] Single-instance resources named `this` or descriptive name
- [ ] Variables have plural names for lists/maps (`subnet_ids` not `subnet_id`)
- [ ] All variables have descriptions
- [ ] All outputs have descriptions
- [ ] Output names follow `{name}_{type}_{attribute}` pattern
- [ ] No double negatives in variable names

### Code Structure Review

- [ ] `count`/`for_each` at top of resource blocks (blank line after)
- [ ] `tags` as last real argument in resources
- [ ] `depends_on` after tags (if used)
- [ ] `lifecycle` at end of resource (if used)
- [ ] Variables ordered: description → type → default → sensitive → nullable → validation
- [ ] Only `#` comments used (no `//` or `/* */`)

### Modern Features Check

- [ ] Using `try()` not `element(concat())`
- [ ] Secrets use write-only arguments or external data sources (not in state)
- [ ] `nullable = false` set on non-null variables
- [ ] `optional()` used in object types where applicable (OpenTofu 1.3+)
- [ ] Variable validation blocks added where constraints needed
- [ ] Consider cross-variable validation for related variables (OpenTofu 1.9+)

### Architecture Review

- [ ] `terraform.tfvars` only at composition level (not in modules)
- [ ] Remote state configured (never local state)
- [ ] Resource modules don't hardcode values (use variables/data sources)
- [ ] `terraform_remote_state` used for cross-composition dependencies
- [ ] File structure follows standard: main.tf, variables.tf, outputs.tf, versions.tf

### Documentation Check

Required documentation for all modules:

- [ ] **README.md exists** with absolute links (registry compatibility)
- [ ] **All variables documented** in README with descriptions and types
- [ ] **All outputs documented** in README with descriptions
- [ ] **Usage examples provided** showing how to use the module
- [ ] **Version requirements specified** (OpenTofu version, provider versions)

---

## Version Management Quick Reference

### Constraint Syntax

| Syntax | Meaning | Use Case |
|--------|---------|----------|
| `"5.0.0"` | Exact version | Avoid (inflexible) |
| `"~> 5.0"` | Pessimistic (>= 5.0, < 6.0 — any 5.x) | Allow minor and patch updates within 5.x |
| `"~> 5.0.1"` | Pessimistic (>= 5.0.1, < 5.1.0 — 5.0.x patches) | Lock to 5.0.x patch updates only |
| `">= 5.0, < 6.0"` | Range | Any 5.x version |
| `">= 5.0"` | Minimum | Risky (breaking changes) |

### Strategy by Component

| Component | Recommendation | Example |
|-----------|----------------|---------|
| **OpenTofu** | Pin minor, allow patch | `required_version = "~> 1.9"` |
| **Providers** | Pin major, allow minor/patch | `version = "~> 5.0"` |
| **Modules (prod)** | Pin exact version | `version = "5.1.2"` |
| **Modules (dev)** | Allow patch updates | `version = "~> 5.1"` |

### Update Workflow

```bash
# Step 1: Lock versions initially
tofu init              # Creates .terraform.lock.hcl

# Step 2: Update to latest within constraints
tofu init -upgrade     # Updates providers

# Step 3: Review changes
tofu plan

# Step 4: Commit lock file
git add .terraform.lock.hcl
git commit -m "Update provider versions"
```

### Update Strategy

**Security patches:**
- Update immediately
- Test: dev → stage → prod
- Prioritize the OpenTofu runtime and provider updates

**Minor versions:**
- Regular maintenance (monthly/quarterly)
- Review changelog for breaking changes
- Test thoroughly before production

**Major versions:**
- Planned upgrade cycles
- Dedicated testing period
- May require code changes
- Phased rollout: dev → stage → prod

---

## Refactoring Quick Reference

### Common Refactoring Patterns

#### Pattern 1: Count to For_Each Migration

**When:** Need stable resource addressing or items might be reordered

```bash
# Step 1: Add for_each, keep count commented
# Step 2: Add moved blocks for each resource
# Step 3: Run tofu plan (should show "moved" not "destroy/create")
# Step 4: Apply changes
# Step 5: Remove commented count
```

**Key principle:** Use `moved` blocks to preserve existing resources

#### Pattern 2: Legacy to Modern OpenTofu

**0.12/0.13 → 1.x checklist:**

- [ ] Replace `element(concat(...))` → `try()`
- [ ] Add `nullable = false` where appropriate
- [ ] Use `optional()` in object types (1.3+)
- [ ] Add `validation` blocks
- [ ] Migrate secrets to write-only arguments (1.11+)
- [ ] Use `moved` blocks for refactoring (1.1+)
- [ ] Add cross-variable validation (1.9+)

#### Pattern 3: Secrets Remediation

**Goal:** Move secrets out of OpenTofu state

```bash
# Step 1: Create secret in AWS Secrets Manager (outside OpenTofu)
aws secretsmanager create-secret --name prod-db-password --secret-string "..."

# Step 2: Update OpenTofu to use data sources
# Step 3: Use write-only argument (OpenTofu 1.11+)
# Step 4: Remove random_password resource or variable
# Step 5: Apply and verify secret not in state
tofu show | grep -i password  # Should not appear
```

### Refactoring Decision Tree

```
What are you refactoring?

├─ Resource addressing (count[0] → for_each["key"])
│  └─ Use: moved blocks + for_each conversion
│
├─ Secrets in state
│  └─ Use: AWS Secrets Manager + write-only arguments (1.11+)
│
├─ Legacy OpenTofu syntax inherited from older HCL
│  └─ Use: Modern feature checklist above
│
└─ Module structure (rename, reorganize)
   └─ Use: moved blocks to preserve resources
```

### Migration Best Practices

**Before refactoring:**
1. Backup state file
2. Test in development first
3. Review tofu plan carefully
4. Document what changed and why

**During refactoring:**
1. One change at a time
2. Verify each step with tofu plan
3. Use moved blocks, not destroy/recreate
4. Keep git history clean with logical commits

**After refactoring:**
1. Verify idempotency (plan shows no changes)
2. Test in staging before production
3. Update documentation
4. Communicate changes to team

**For detailed refactoring patterns, see:** [Code Patterns: Refactoring Patterns](code-patterns.md#refactoring-patterns)

---

## Common Patterns

### Resource Naming

```hcl
# ✅ Good: Descriptive, contextual
resource "aws_instance" "web_server" { }
resource "aws_s3_bucket" "application_logs" { }

# ❌ Bad: Generic
resource "aws_instance" "main" { }
resource "aws_s3_bucket" "bucket" { }
```

### Variable Naming

```hcl
# ✅ Good: Context-specific
var.vpc_cidr_block
var.database_instance_class

# ❌ Bad: Generic
var.cidr
var.instance_class
```

### File Organization

```
Standard module structure:
├── main.tf          # Primary resources
├── variables.tf     # Input variables
├── outputs.tf       # Output values
├── versions.tf      # Provider versions
└── README.md        # Documentation
```

---

**Back to:** [Main Skill File](../SKILL.md)
