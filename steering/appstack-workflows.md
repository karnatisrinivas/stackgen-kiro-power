# StackGen AppStack Workflows

Step-by-step guidance for creating and managing StackGen appStacks through Kiro. An appStack is the core unit of infrastructure in StackGen — a named, versioned topology of cloud resources tied to a project and environment.

---

## AppStack Creation Workflow

**IMPORTANT**: Never call write MCP tools (`create_appstack`, `add_resource_*`, `update_resource`, `create_env_profile`, etc.) until the user explicitly confirms the plan in Step 4.

Use `stackgen-user` for appStack operations and `stackgen-admin` for projects and governance.

---

### Step 1 — Analyze the Codebase

Inspect the workspace root to infer application type and deployment hints:

| Signal | Likely Stack |
|--------|-------------|
| `package.json` + express/fastify/nest deps | Node.js API |
| `package.json` + next.js | Node.js (Next.js/static) |
| `requirements.txt` / `pyproject.toml` / `Pipfile` | Python |
| `go.mod` | Go |
| `pom.xml` / `build.gradle` | Java / JVM |
| `Dockerfile` | Container (note base image) |
| `serverless.yml` / SAM template | Serverless |
| `terraform/` / `cdk.json` / `pulumi.yaml` | Existing IaC (complement, don't duplicate) |

Record and present to the user:
- Detected language/runtime and framework
- Repo folder name (default appStack name candidate)
- Existing IaC hints — note when StackGen should complement rather than replace
- Suggested workload pattern (e.g. Node API → ECS/Fargate + ALB + ECR)

Do not infer AWS account IDs, subscription IDs, or secrets from environment files.

---

### Step 2 — Confirmation Questions (Required)

Gather all required answers before drafting the plan. Do not proceed until complete.

#### Required

1. **Cloud provider** — `aws` | `azure` | `gcp`
2. **Primary region** — e.g. `us-east-1`, `westeurope`, `us-central1` (must match provider)
3. **StackGen project** — call `get_stackgen_projects` on `stackgen-admin`, list names, let user pick (or ask them to create one in the StackGen UI if none fit)
4. **AppStack name** — default: `<repo-folder>-<env>`; must be unique within the project
5. **Environment** — `dev` | `staging` | `prod` (used for naming, labels, and env profiles)

#### Recommended (ask if not stated)

6. **Start from template?** — call `get_appstacks` with `labels: ["template"]` filtered by `cloud_provider` and workload type; if user picks one, use its UUID as `appstack_ref_id`
7. **Resource packs vs individual resources** — prefer `add_resource_pack_to_appstack` when a pack matches the workload (get pack UUIDs from `get_supported_resource_types` on the new appStack)
8. **Multi-region?** — only ask if user mentions HA or global requirements

#### Optional

9. Short description for the appStack
10. Labels (e.g. `kiro-generated`, `nodejs`, `dev`, team names)

---

### Step 3 — Draft Plan (No Writes)

Present a structured plan for user review:

```markdown
## Proposed AppStack

- **Name:** <appstack-name>
- **Project:** <project-name>
- **Cloud:** <aws|azure|gcp>
- **Region:** <region>
- **Source:** Template `<template-name>` (uuid: ...) OR greenfield
- **Workload:** <runtime> <framework> → <suggested pattern>
- **Resources to add:**
  - <pack-name> (pack) OR
  - <resource-type> (`<identifier>`)
- **Env profile:** `<env>` with <region variables>
- **Labels:** <labels>
```

Note: if using a template, `cloud_provider` may be inherited — confirm with user if needed.

---

### Step 4 — Explicit Confirmation

Ask:

> Reply **yes** to create this appStack in StackGen, or tell me what to change.

Only proceed on clear approval (`yes`, `create it`, `go ahead`, `looks good`, etc.). If the user asks for changes, loop back to Step 3 with revisions.

---

### Step 5 — Create via MCP

Execute in this order:

1. **Create the appStack** — `create_appstack` on `stackgen-user`:
   - Required: `name`, `project_name`, `cloud_provider` (unless inherited from template)
   - Optional: `appstack_ref_id` (template UUID), `description`, `labels`

2. **Add infrastructure** — in order:
   - Call `get_supported_resource_types` on the new appStack UUID to get valid packs and resource types
   - Prefer `add_resource_pack_to_appstack` when a pack fits the workload (use pack **UUID**, not display name)
   - Otherwise `add_resource_to_appstack` per resource with valid `resource_type` and `identifier`
   - Call `connect_resources` to define dependencies (e.g. ECS service → RDS database)

3. **Configure resources** — for each resource needing non-default settings:
   - Call `get_resource_type_configurations` to discover valid configuration keys
   - Apply via `update_resource` with the resource UUID and key-value pairs

4. **Create environment profile** — `create_env_profile` on the appStack:
   - Set region-specific variables (e.g. `AWS_REGION`, `AZURE_LOCATION`)
   - Apply environment-specific values for the chosen env (`dev`/`staging`/`prod`)

5. **Verify** — call `get_appstack_resources` and confirm resources are present
   - Share the `topology_page` URL from `get_appstacks` if returned

On errors: report the MCP error message and suggest fixes (wrong UUID, insufficient PAT scope, invalid resource type, resource limit exceeded).

---

### Step 6 — Repo Integration (Optional)

After successful creation, offer (only if user requests):

- **Download IaC**: Use the `iac_download_url` field from `get_appstacks`
- **Push to Git**: Call `push_appstack_to_git` on `stackgen-admin` if Git integration is configured
- **Run Terraform**: Offer to trigger `create_appstack_action_run` on `stackgen-user` for plan or apply — always confirm the action type before executing

---

## AppStack Inspection Workflow

When the user asks to inspect, list, or understand existing infrastructure:

1. Call `get_stackgen_projects` on `stackgen-admin` to identify the relevant project
2. Call `get_appstacks` on `stackgen-user` filtered by project name or other criteria
3. For details on a specific appStack, call `get_appstack_resources` with the appStack UUID
4. Present resource types, names, identifiers, and connection relationships
5. Share the `topology_page` URL for visual inspection in the StackGen UI

---

## AppStack Update Workflow

When the user asks to modify an existing appStack:

1. Call `get_appstacks` and `get_appstack_resources` to understand current state
2. Present what currently exists before proposing changes
3. Draft proposed changes (resources to add, remove, or reconfigure)
4. Require explicit confirmation before any write call
5. Execute: `add_resource_to_appstack`, `update_resource`, or `connect_resources` as needed
6. Verify with `get_appstack_resources`

---

## MCP Tool Quick Reference

| Action | Server | Tool |
|--------|--------|------|
| List projects | stackgen-admin | `get_stackgen_projects` |
| List templates | stackgen-user | `get_appstacks` with `labels: ["template"]` |
| Create appStack | stackgen-user | `create_appstack` |
| List existing appStacks | stackgen-user | `get_appstacks` |
| Get appStack resources | stackgen-user | `get_appstack_resources` |
| List packs and types | stackgen-user | `get_supported_resource_types` |
| Add resource pack | stackgen-user | `add_resource_pack_to_appstack` |
| Add single resource | stackgen-user | `add_resource_to_appstack` |
| Connect resources | stackgen-user | `connect_resources` |
| Get resource config options | stackgen-user | `get_resource_type_configurations` |
| Configure resource | stackgen-user | `update_resource` |
| Create env profile | stackgen-user | `create_env_profile` |
| Run Terraform action | stackgen-user | `create_appstack_action_run` |
| Push IaC to Git | stackgen-admin | `push_appstack_to_git` |
