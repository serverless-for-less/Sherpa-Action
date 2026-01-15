---
description: Builds and deploys your project to a cloud provider of your choice.
---

Take the user context into account when building and deploying the project:
<usercontext>
"$ARGUMENTS"
</usercontext>

Be vocal, share the steps you are taking.

Step 1: Prep
1. If `.sherpa.sh/build-info.md` doesn't exist run the plugin command `/Sherpa/detect-build-info`.
2. If `.sherpa.sh/deployment-plan.md` doesn't exist run the plugin command `/Sherpa/plan-deployment`.

Step 2: Build 
3. Run `/Sherpa/build` to build the project
4. Run `/Sherpa/create-infrastructure` to ensure the needed infrastructure exists

Step 3: Deployment:
1. Read the files `.sherpa.sh/build-info.md` and `.sherpa.sh/deployment-plan`.
2. Execute the deployment plan by uploading and running the application code and static assets based on the deployment plan. **Always use the provider skill. Always use the MCP server. Never use Terraform.**

Step 4: Document (REQUIRED - DO NOT SKIP)
**You MUST complete this step before finishing.**

1. Write the commands that worked to build and deploy into `.sherpa.sh/executionlog.md` (overwrite what's existing) for later use so you can repeat the process. Include:
   - Build commands used
   - Deployment commands and API calls made
   - Any configuration changes applied
   - Timestamp of this deployment

2. Verify `.sherpa.sh/infrastructure.md` exists and is up to date (created by `/Sherpa/create-infrastructure`).

IMPORTANT! NEVER write secrets, credentials, passwords, or API keys into these documents. Refer to them by UPPERCASE variable names (e.g., $CLOUDFLARE_API_TOKEN).