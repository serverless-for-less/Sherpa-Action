---
name: Nextjs Deployment
description: Steps required to build and deploy a Nextjs project into a cloud provider.
---

When building and deploying Nextjs, never use the following:
- open-next
- cloudflare-next

You want to deploy
the actual application code that gets output into the .next folder.
Follow these steps.

1. Make sure you use the "next-devtools" MCP server for docs.
2. Determine the output folder from the next config file.
3. Run the build commands from the `.sherpa.sh/build-info.md` file. Use NODE_ENV=production unless otherwise specified by the user.
4. Detect the `output` mode in the next config.
5. If output is `export` { 
    a. upload the files in the `out` folder to the CDN. using your `cdn-uploading` skill.
} else { 
    a. Copy the `.next` folder to the server via scp.
    b. Run the app on the server:
        - If no `output` and no turborepo: `next start`
        - If no `output` and turborepo: `turbo run start --filter=web`
        - If `standalone` no turborepo: `node` with `.next/standalone/server.js`.
        - If `standalone` AND turborepo: `node` with `.next/standalone/[APP PATH]/server.js`. For example an app in app/web will be at .next/standalone/app/web/server.js
    c. Upload the static assets from `.next/static` (or the turborepo path when applicable) and the `public` folder in the nextjs project (Not the `.next` folder) to the CDN using your `cdn-uploading` skill.
}