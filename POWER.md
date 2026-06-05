# StackGen Infrastructure Power — Complete Documentation

## Overview

The StackGen Infrastructure Power connects Kiro's AI agent directly to your StackGen workspace, enabling natural-language infrastructure provisioning, drift detection, and SRE workflow automation across AWS, Azure, and GCP. No Terraform expertise or manual configuration files required.

## Key Capabilities

- AppStack creation and management from natural language descriptions
- Cloud infrastructure provisioning across AWS, Azure, and GCP
- Topology visualization and dependency mapping
- Drift detection and remediation between desired and actual state
- Policy enforcement and compliance checking
- Environment profile management (dev/staging/prod)
- IaC download and Git integration
- Project and secret management

## Authentication

Two MCP servers handle distinct responsibilities, both authenticated via Personal Access Token (PAT):

- **stackgen-admin**: Projects, policies, custom modules, and secrets management
- **stackgen-user**: AppStacks, topology, snapshots, and infrastructure resources

PAT credentials are stored locally in `mcp.json` and never sent to external AI providers.

## MCP Server Architecture

### stackgen-admin (Projects & Governance)
- `get_stackgen_projects` — list and filter projects
- `get_custom_modules` — retrieve reusable infrastructure modules
- `push_appstack_to_git` — integrate IaC with source control
- `manage_secrets` — project-level secret management
- Policy management and enforcement tools

### stackgen-user (Infrastructure & AppStacks)
- `create_appstack` — provision a new infrastructure topology
- `get_appstacks` — list appStacks, including templates
- `get_appstack_resources` — inspect deployed resources
- `get_supported_resource_types` — list valid resource types for a cloud/region
- `add_resource_pack_to_appstack` — add a grouped resource pack (e.g. ECS + ALB + ECR)
- `add_resource_to_appstack` — add an individual resource
- `connect_resources` — define dependencies and relationships between resources
- `update_resource` — configure resource-level settings
- `get_resource_type_configurations` — retrieve configurable fields for a resource type
- `create_env_profile` — create environment-specific variable profiles
- `create_appstack_action_run` — trigger Terraform plan/apply operations
- Topology and snapshot management tools

## AppStack Creation Workflow

AppStacks are the core unit of infrastructure in StackGen — a named, versioned topology of cloud resources tied to a project and environment. Creating one from Kiro follows this workflow:

1. **Codebase analysis** — detect runtime, framework, and deployment hints from workspace files
2. **Confirmation questions** — collect cloud provider, region, project, name, and environment
3. **Template selection** — optionally start from an existing StackGen template
4. **Draft plan** — present the full plan for user review before any writes
5. **Explicit confirmation** — wait for clear user approval
6. **MCP execution** — create the appStack, add resources, configure environments, verify topology
7. **Repo integration** — optionally download IaC or push to Git

## Supported Workload Patterns

| Runtime | Typical Resources |
|---------|-------------------|
| Node.js API | ECS/Fargate, ALB, ECR, RDS/DynamoDB |
| Node.js (Next.js/static) | CloudFront, S3, Route53 |
| Python | ECS/Fargate, ALB, ECR, RDS |
| Go | ECS/Fargate or Lambda, API Gateway |
| Java | ECS/Fargate, ALB, ECR, RDS |
| Container (generic) | ECS/Fargate or App Service |
| Serverless | Lambda, API Gateway, DynamoDB, S3 |

## Recommended Approach

Describe your infrastructure need in plain language. Kiro's agent will analyze your codebase, ask targeted confirmation questions, and present a plan before making any changes. No writes occur without explicit user approval.
