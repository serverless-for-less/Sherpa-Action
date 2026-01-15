---
name: manage-infrastructure-mcp
description: Create, modify, query, and destroy infrastructure resources using MCP servers. Leverages MCP APIs for tag-based resource reconciliation and GitHub Actions workflow generation. No Terraform needed—AI-driven, tag-based infrastructure management.
Supports: Cloudflare, AWS
---

Manage infrastructure through **tag-based reconciliation** via MCP servers to:

1. **Query live state** from providers automatically
2. **Reason about differences** between actual and intended state
3. **Execute surgical changes** through MCP tools
4. **Tag everything** for attribution and governance
5. **Generate GitHub Actions workflows** for repeatability.

When managing infra always:

1. Check: Call MCP to list relevant provider resources using filters (env, tags, type).
2. Compare: Infer intended state from your request (and any config/docs). 
3. Mark: Diff actual vs intended: Decide which items to create / update / delete / tag-only.
4. Plan: Build a human-readable plan of all changes.
5. Confirm: Ask you to confirm (and whether this is dry-run or real).
6. Execute On approval, call MCP to:
    a. CREATE missing resources.
    b. MODIFY drifted resources.
    c. DELETE resources you explicitly approve to remove.
7. Tag: On each touched resource, ensure governance tags are set/updated (ManagedBy, Intent, Owner, Environment).

## How to tag:

1. Always put `ManagedBy: "Sherpa.sh"`
2. Always put the `Environment: {BRANCH NAME}`
3. Always put `DeploymentId: {github repo id}-{github workflow run id}`
4. Always put `Created: {utc timestamp}` on new resources
5. Always put `Updated: {utc timestamp}` on updated resources
6. Optional: put `SSHKeyName`: `sherpa-deploy` when accessing via SSH.

## When to Use This Skill

- **Creating infrastructure**: Ask Claude to provision instances, databases, DNS records, workers, etc. with auto-tagging
- **Discovering state**: Query what resources exist, which are managed by automation, and which are orphaned
- **Modifying resources**: Scale databases, update DNS, change firewall rules, adjust tags—Claude uses MCP tools
- **Generating workflows**: Claude creates reusable GitHub Actions workflows triggered on schedule or manually
- **Tagging unmanaged resources**: Find resources without governance tags and apply them
- **Detecting drift**: Query for resources that don't match intended config and reconcile automatically

### VM Creation Flow

1. **Create VM** via provider MCP server, injecting `$SSH_PUBLIC_KEY` for access
2. **Wait for ready** - poll until instance is running and SSH port is open
3. **Store details** in `.sherpa.sh/infrastructure.md`:
   - Instance ID, IP address, region, size
   - Security group and key pair names
4. **Access VM** using the `vm-access` skill for deployment

### VM-Specific Tags

In addition to standard governance tags, VMs include:

| Tag | Example | Purpose |
|-----|---------|---------|