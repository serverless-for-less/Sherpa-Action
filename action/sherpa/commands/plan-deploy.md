---
description: Plan a deployment to target infrastructure.
---

Create a deployment plan that consists of infrastructure definitions and what parts of the build project code will get deployed where, and the method for doing it.

1. Ask the user what provider they want to deploy to. Ensure we can access the mcp server for that provider. (We currently support cloudflare and AWS native to this Sherpa plugin). If you don't have access tell the user how to fix it, then end the chat and don't continue.
2. Use your `manage-infrastructure` skill, the `.sherpa.sh/build-info.md`, and if they exist `.sherpa.sh/deployment-plan.md`,`.sherpa.sh/infrastructure.md`, and the latest `.sherpa.sh/executionlog` to decide what infrastructure needs to be created/updated.Examine the infrastructure and deployment commands in the executionlog. Decide if you should just repeat them. Usually, you want to repeat them.
3. Make a plan of what you are going to create/update and what code and assets you will deploy and where they will go. Use the skills you have for the framework and for the cloud provider. Think very hard on this step.
4. Save this to the file `.sherpa.sh/deployment-plan.md`. Use Semantic Versioning at the top of the file, update it if the file already exists.
5. Think hard and check the plan for any mistakes. Common mistakes are logic, provider, and framework specific issues. If you see mistakes, correct them.