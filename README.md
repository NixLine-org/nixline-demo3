# NixLine Demo3 - Upstream Consumption with Auto-Approved PRs

This repository demonstrates **pure upstream consumption** of [NixLine](https://github.com/NixLine-org/nixline-baseline) with automated policy updates via auto-approved pull requests.

Shows how organizations can:
- Use NixLine baseline as upstream without forking
- Receive automatic policy updates through GitHub's auto-merge
- Maintain zero maintenance overhead for governance policies
- Configure branch protection with auto-approval workflows

## Key Features Demonstrated

**Pure Upstream Consumption**
- No local configuration files or flakes
- Direct baseline calls via `nix run github:NixLine-org/nixline-baseline#sync`
- Zero maintenance fork-free architecture

**Automated Policy Updates**
- Weekly policy sync via GitHub Actions
- Auto-approved PRs for policy changes
- Branch protection with automated merging
- Instant propagation of baseline updates

**Enterprise-Ready Governance**
- Audit trail via pull request history
- Automated testing before policy application
- Configurable approval workflows
- Compliance-friendly change tracking

## Quick Start

This demo repository shows NixLine pure upstream consumption. Policy files are materialized from the baseline without any local configuration.

**Initial Setup (if policies are out of sync):**

If CI is failing with "Validation failed", this means policies need to be synced from the baseline. The automated workflow will handle this:

```bash
# Option 1: Trigger policy sync workflow manually (recommended)
gh workflow run "Policy Sync" --repo NixLine-org/nixline-demo3

# Option 2: Manual sync (for testing only)
nix run github:NixLine-org/nixline-baseline#sync
```

**What happens with the automated workflow:**
1. Policy sync workflow detects out-of-sync policies
2. Creates a PR with updated policy files
3. Auto-approval workflow approves the PR automatically
4. Auto-merge occurs when CI checks pass
5. Subsequent CI runs will pass with synchronized policies

**Manual commands for testing:**
```bash
# Check if policies are current
nix run github:NixLine-org/nixline-baseline#check

# Preview what would change
nix run github:NixLine-org/nixline-baseline#sync -- --dry-run

# Manual sync (creates local changes, not recommended for production)
nix run github:NixLine-org/nixline-baseline#sync
```

## How It Works

This repository demonstrates the **pure upstream consumption** pattern with automated PR workflows:

**Direct Baseline Integration**
- Uses `nix run github:NixLine-org/nixline-baseline#command` for all operations
- No local flake.nix or configuration files
- Baseline repository serves as single source of truth

**Auto-Approved Policy Updates**
- `.github/workflows/policy-sync.yml` runs weekly policy checks
- Creates PRs for any policy changes from baseline
- Auto-approval workflow merges changes after CI passes
- Maintains audit trail while eliminating manual overhead

## Automated Workflows

**Policy Sync with Auto-Approval (`policy-sync.yml`)**
```yaml
name: Policy Sync
on:
  schedule:
    - cron: '0 14 * * 0'  # Weekly on Sunday
  workflow_dispatch:

jobs:
  sync:
    uses: NixLine-org/.github/.github/workflows/nixline-policy-sync-pr.yml@stable
    with:
      consumption_pattern: direct
      baseline_repo: NixLine-org/nixline-baseline
      baseline_ref: stable
      create_pr: true
      auto_approve: true
```

**Auto-Approval Workflow (`auto-approve.yml`)**
```yaml
name: Auto Approve Policy Updates
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  auto-approve:
    if: github.actor == 'github-actions[bot]' && contains(github.event.pull_request.title, 'Policy Sync')
    runs-on: ubuntu-latest
    steps:
      - name: Auto approve
        uses: hmarr/auto-approve-action@v3
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Branch Protection Configuration

For enterprise environments requiring change approval, configure branch protection:

```yaml
# Required status checks
- nixline-ci

# Require pull request reviews before merging
require_pull_request_reviews: true

# Allow auto-merge when all requirements are met
allow_auto_merge: true

# Restrict pushes that bypass pull request requirements
enforce_admins: false
```

## Materialized Policy Files

This demo materializes these policies from the upstream baseline:

| Pack | Purpose | Materialized File |
|------|---------|-------------------|
| `editorconfig` | Code formatting standards | `.editorconfig` |
| `license` | Apache 2.0 license | `LICENSE` |
| `security` | Security policy | `SECURITY.md` |
| `codeowners` | Code ownership rules | `.github/CODEOWNERS` |
| `precommit` | Pre-commit hooks | `.pre-commit-config.yaml` |
| `dependabot` | Dependabot configuration | `.github/dependabot.yml` |

## Comparison: Demo1 vs Demo2 vs Demo3

| Feature | Demo1 (Basic) | Demo2 (Config-Driven) | Demo3 (Auto-Approved) |
|---------|---------------|----------------------|----------------------|
| Customization | Default policies | Full parameterization | Default policies |
| Updates | Manual sync | Manual sync | Automated PRs |
| Configuration | CLI only | TOML + CLI | None required |
| Maintenance | Low | Medium | Zero |
| Audit Trail | Git history | Git history | PR history |
| Enterprise Ready | Basic | Advanced | Full automation |

## Architecture Benefits

**Zero Maintenance Governance**
No configuration files to maintain. Automatic updates from upstream baseline. Complete hands-off policy management.

**Audit and Compliance**
Pull request history provides complete audit trail. Automated testing ensures policy integrity. Branch protection enables compliance workflows.

**Enterprise Integration**
Works with existing GitHub enterprise features. Supports custom approval workflows. Integrates with organization security policies.

**Instant Policy Propagation**
Baseline updates create PRs across all consumer repos. Auto-approval eliminates manual bottlenecks. Policy changes deploy within hours organization-wide.

## Advanced Configuration

**Custom Approval Requirements**
```yaml
# Require specific team approval for policy changes
auto-approve:
  if: |
    github.actor == 'github-actions[bot]' &&
    contains(github.event.pull_request.title, 'Policy Sync') &&
    github.event.pull_request.requested_teams contains '@myorg/security'
```

**Environment-Specific Policies**
```bash
# Production requires stricter policies
nix run github:NixLine-org/nixline-baseline#sync -- --packs security,license,codeowners

# Development allows more flexibility
nix run github:NixLine-org/nixline-baseline#sync -- --exclude dependabot
```

**Integration with External Systems**
```yaml
# Notify external systems of policy changes
- name: Notify compliance system
  if: success()
  run: |
    curl -X POST $COMPLIANCE_WEBHOOK \
      -H "Content-Type: application/json" \
      -d '{"repo": "${{ github.repository }}", "policies_updated": true}'
```

## Why This Pattern Matters

This approach solves the organizational governance automation challenge:

**Traditional Governance**: Policy updates require manual coordination across hundreds of repositories, creating bottlenecks and inconsistent application.

**NixLine Auto-Approval**: Policy updates from the centralized baseline automatically create tested, approved pull requests across all consumer repositories, ensuring instant organization-wide consistency.

The result is enterprise-grade governance automation with complete audit trails and zero maintenance overhead for development teams.