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
2. Execute the deployment plan by uploading and running the application code and static assets based on the deployment plan

Step 4: Document
4. Write the commands that worked to build and deploy into `.sherpa.sh/executionlog.md` (overwrite whats existing) for later use so you can repeat the process. IMPORTANT! NEVER write secrets, credentials, passwords, or API keys into this document. Refer to them by all UPPERCASE variables.