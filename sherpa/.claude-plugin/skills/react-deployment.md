---
name: React Deployment
description: Steps required to build and deploy React applications (Vite, Create React App, or custom configurations) to cloud providers.
---

When building and deploying React applications, follow these steps to ensure proper deployment. React apps are typically Single Page Applications (SPAs) that compile to static files.

## Detection

Identify React applications by:
- `react` and `react-dom` in `package.json` dependencies
- Vite: `vite.config.js` or `vite.config.ts` with `@vitejs/plugin-react`
- Create React App: `react-scripts` in dependencies
- Custom: Look for JSX/TSX files with React imports

## Build Process

1. Read the build commands from `.sherpa.sh/build-info.md` file.
2. Run the build with `NODE_ENV=production` unless otherwise specified.

### Vite React

```bash
# Default build command
npm run build
# or
npx vite build
```

**Output directory:** `dist/` (configurable in `vite.config.js` via `build.outDir`)

Check `vite.config.js` for custom output directory:
```javascript
export default defineConfig({
  build: {
    outDir: 'build' // custom output directory
  }
})
```

### Create React App (CRA)

```bash
# Build command
npm run build
# or
npx react-scripts build
```

**Output directory:** `build/`

### Custom Webpack/Parcel/esbuild

- Check `package.json` scripts for the build command
- Output directory varies - check config files or `package.json`

## Deployment

React apps are SPAs that output static files. Always use CDN deployment.

1. Identify the output directory:
   - Vite: `dist/` (or custom `build.outDir`)
   - CRA: `build/`
   - Custom: Check build config

2. Upload the entire output folder to the CDN using your `cdn-uploading` skill.

## Client-Side Routing Configuration

React apps using React Router or similar need special handling for client-side routing.

### Important: SPA Fallback

Configure your CDN/server to serve `index.html` for all routes that don't match a static file. This ensures routes like `/dashboard` or `/users/123` work correctly after page refresh.

**Cloudflare Pages:** Automatic SPA support
**AWS S3 + CloudFront:** Configure custom error response to serve `index.html` for 404s
**Nginx:**
```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

## Environment Variables

React apps use environment variables at **build time**, not runtime.

### Vite
- Variables must be prefixed with `VITE_`
- Access via `import.meta.env.VITE_API_URL`
- Set before build: `VITE_API_URL=https://api.example.com npm run build`

### Create React App
- Variables must be prefixed with `REACT_APP_`
- Access via `process.env.REACT_APP_API_URL`
- Set before build: `REACT_APP_API_URL=https://api.example.com npm run build`

## Base Path / Public URL

If deploying to a subdirectory (not root):

### Vite
Set `base` in `vite.config.js`:
```javascript
export default defineConfig({
  base: '/my-app/'
})
```
Or via CLI: `npx vite build --base=/my-app/`

### Create React App
Set `homepage` in `package.json`:
```json
{
  "homepage": "/my-app"
}
```
Or set `PUBLIC_URL`: `PUBLIC_URL=/my-app npm run build`

## Monorepo Support

For monorepos using Turborepo, Nx, or pnpm workspaces:

1. Identify the app directory (e.g., `apps/web`, `packages/frontend`)
2. Run build from root: `turbo run build --filter=web` or `nx build web`
3. Output will be in the app's output directory (e.g., `apps/web/dist`)

## Common Issues

### Assets not loading
- Check `base` or `PUBLIC_URL` configuration
- Ensure all assets use relative paths or respect the base path

### Routes returning 404
- Configure SPA fallback on CDN/server
- Ensure `index.html` is served for unknown routes

### Environment variables undefined
- Ensure correct prefix (`VITE_` or `REACT_APP_`)
- Variables must be set at build time, not runtime
- Rebuild after changing environment variables

## When to Use This Skill

- Deploying React apps built with Vite
- Deploying Create React App projects
- Deploying custom React configurations with Webpack, Parcel, or esbuild
- Any React SPA that compiles to static files
