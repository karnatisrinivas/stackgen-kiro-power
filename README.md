# StackGen Power for Kiro

The StackGen Kiro Power lets you create and provision compliant infrastructure based on your organizational governance. It also helps you manage drift in your cloud environments across AWS — directly from Kiro using natural language in agent chat.

## About StackGen

[StackGen](https://stackgen.com) is an infrastructure management platform that enables teams to provision, visualize, and govern cloud infrastructure across AWS, Azure, and GCP using a topology-first approach. StackGen generates Terraform from visual appStack definitions and enforces policy guardrails across environments.

With this power, Kiro can create infrastructure directly from your codebase, manage existing appStacks, and run Terraform operations — all through conversation.

## Features

- **Codebase-Aware Provisioning**: Kiro analyzes your repo (Node.js, Python, Go, Java, containers) and proposes the right infrastructure
- **Multi-Cloud Support**: AWS, Azure, and GCP with provider-native resource types
- **AppStack Management**: Create, inspect, update, and version infrastructure topologies
- **Template-Based Scaffolding**: Start from StackGen-managed templates for common workload patterns
- **Environment Profiles**: Separate dev/staging/prod configuration with variable profiles
- **Drift Detection**: Detect and remediate differences between desired and actual cloud state
- **IaC Integration**: Download generated Terraform or push directly to your Git repo
- **Policy Enforcement**: Apply compliance rules and governance policies to infrastructure

## Installation

### Prerequisites

1. **StackGen Account**: Active StackGen workspace at your organization's instance URL
2. **Personal Access Token (PAT)**: A PAT with Read + Write scope from your StackGen workspace settings

### Kiro Power Setup

1. Open the **Powers** panel in Kiro
2. Click **Add power from GitHub**
3. Enter this repository URL
4. After installation, open `mcp.json` in the power directory and replace both placeholder values:
   - `https://not-configured` → your StackGen workspace URL (e.g. `https://app.stackgen.com`)
   - `Bearer not-configured` → `Bearer <your-PAT>`
5. Reload Kiro to activate both MCP servers

> **Security note**: Your PAT is stored locally in `mcp.json` only. It is never sent to Anthropic or any external AI provider.

### Verifying Setup

After reload, Kiro should have access to StackGen tools. Ask Kiro:
> "List my StackGen projects"

If tools are unavailable, verify `mcp.json` has no remaining `not-configured` values and reload again.

## What Can You Do With StackGen and Kiro?

### 1 — Provision Infrastructure From Your Repo
- Create an appStack for this Node.js API on AWS us-east-1
- Scaffold ECS + RDS infrastructure for my Python service in the staging project
- Set up serverless infrastructure for this Lambda function on AWS

### 2 — Manage Existing AppStacks
- List all appStacks in the production project
- Show me the topology for the checkout-api appStack
- What resources are in the payments-service appStack?

### 3 — Detect and Remediate Drift
- Is there drift between the desired and actual state for checkout-api?
- Show me what changed in production since the last snapshot
- Run a Terraform plan for the checkout-api appStack

### 4 — Manage Projects and Policies
- List all StackGen projects I have access to
- What policies are applied to the production project?
- Push the checkout-api IaC to our GitHub repository

### 5 — Environment Configuration
- Create a prod environment profile for checkout-api in us-east-1
- Update the RDS instance type for the staging environment
- What configuration options are available for ALB resources?

## MCP Server Reference

| Server | Responsibilities |
|--------|-----------------|
| `stackgen-admin` | Projects, policies, custom modules, secrets, Git integration |
| `stackgen-user` | AppStacks, topology, resources, environment profiles, Terraform actions |

## Steering Files

This power includes structured steering content that guides Kiro's AI on when and how to use StackGen:

- [steering/stackgen-steering-index.md](steering/stackgen-steering-index.md) — when to invoke StackGen tools
- [steering/setup-guide.md](steering/setup-guide.md) — PAT configuration and troubleshooting
- [steering/appstack-workflows.md](steering/appstack-workflows.md) — appStack creation and management workflows
- [steering/infrastructure-patterns.md](steering/infrastructure-patterns.md) — cloud provider resource patterns

## Resources

- **StackGen Docs**: [docs.stackgen.com](https://docs.stackgen.com)
- **StackGen App**: [app.stackgen.com](https://app.stackgen.com)
- **Support**: [support@stackgen.com](mailto:support@stackgen.com)
