# StackGen Infrastructure Steering Guide

Guidance for provisioning and managing cloud infrastructure using StackGen MCP servers. Use these tools when the user needs to create, inspect, or manage infrastructure topologies across AWS, Azure, and GCP.

## Steering Files

| File | Description |
|------|-------------|
| [setup-guide.md](setup-guide.md) | PAT configuration, initial setup, and connection troubleshooting |
| [appstack-workflows.md](appstack-workflows.md) | AppStack creation workflow — codebase analysis through MCP execution |
| [infrastructure-patterns.md](infrastructure-patterns.md) | Cloud provider resource patterns for common workload types |

---

## When to Use StackGen MCP Tools

Use `stackgen-admin` and `stackgen-user` MCP tools when the user needs to:

- **Provision new infrastructure**: Create appStacks for a service, repo, or workload
- **Scaffold cloud resources**: Add ECS, RDS, Lambda, ALB, S3, and other resources to a topology
- **Manage existing appStacks**: List, inspect, update, or delete infrastructure topologies
- **Detect drift**: Compare desired infrastructure state against actual cloud state
- **Run Terraform operations**: Trigger plan or apply actions on an appStack
- **Manage environment profiles**: Create or update dev/staging/prod variable configurations
- **Enforce policies**: Apply or inspect governance rules on projects and appStacks
- **Integrate with Git**: Push generated IaC to a connected repository
- **Manage projects**: List, inspect, or organize infrastructure by StackGen project
- **Handle secrets**: Store and reference project-level secrets in infrastructure config

---

## When NOT to Use StackGen MCP Tools

Do not call StackGen tools when:
- The user is asking about application code, not infrastructure
- The user asks about non-cloud tooling (CI/CD pipelines, Docker builds, etc.) unless infrastructure provisioning is part of the ask
- The MCP servers show `not-configured` — run setup guidance first (see [setup-guide.md](setup-guide.md))

---

## Server Responsibilities

### stackgen-admin
- Projects and project metadata
- Policy management and enforcement
- Custom module registry
- Secret management
- Git integration (`push_appstack_to_git`)

### stackgen-user
- AppStack CRUD (`create_appstack`, `get_appstacks`, `delete_appstack`)
- Resource management (`add_resource_to_appstack`, `add_resource_pack_to_appstack`, `update_resource`, `connect_resources`)
- Resource discovery (`get_supported_resource_types`, `get_resource_type_configurations`)
- Environment profiles (`create_env_profile`)
- Topology and snapshots
- Terraform action runs (`create_appstack_action_run`)

---

## Key Principles

1. **Never write without explicit confirmation.** Always present a plan and wait for user approval before calling any write MCP tool (`create_appstack`, `add_resource_*`, `update_resource`, `create_env_profile`, etc.).
2. **Check setup before proceeding.** If StackGen tools are unavailable or `mcp.json` contains `not-configured`, guide the user through [setup-guide.md](setup-guide.md) before any other action.
3. **Analyze before proposing.** Inspect the workspace codebase to infer the appropriate workload pattern, then ask targeted confirmation questions.
4. **Prefer resource packs over individual resources** when a pack matches the workload — fewer API calls, better-tested topologies.
5. **Always verify after creation.** Call `get_appstack_resources` and share the `topology_page` URL after any appStack is created.
