---
name: Vue.js Deployment
description: Steps required to build and deploy Vue.js applications (Vite, Vue CLI) to cloud providers as static SPAs.
---

When building and deploying Vue.js applications, follow these steps. Vue apps are typically Single Page Applications (SPAs) that compile to static files. For Vue with SSR, see the `nuxt-deployment` skill instead.

## Detection

Identify Vue.js applications by:
- `vue` in `package.json` dependencies
- Vite: `vite.config.js` with `@vitejs/plugin-vue`
- Vue CLI: `@vue/cli-service` in devDependencies
- Look for `.vue` single-file components

## Build Process

1. Read the build commands from `.sherpa.sh/build-info.md` file.
2. Run the build with `NODE_ENV=production` unless otherwise specified.

### Vite Vue (Recommended)

```bash
# Default build command
npm run build
# or
npx vite build
```

**Output directory:** `dist/` (configurable in `vite.config.js` via `build.outDir`)

Check `vite.config.js` for custom configuration:
```javascript
export default defineConfig({
  plugins: [vue()],
  build: {
    outDir: 'build' // custom output directory
  }
})
```

### Vue CLI

```bash
# Build command
npm run build
# or
npx vue-cli-service build
```

**Output directory:** `dist/`

Custom output in `vue.config.js`:
```javascript
module.exports = {
  outputDir: 'build'
}
```

## Deployment

Vue SPAs output static files. Always use CDN deployment.

1. Identify the output directory:
   - Vite Vue: `dist/` (or custom `build.outDir`)
   - Vue CLI: `dist/` (or custom `outputDir`)

2. Upload the entire output folder to the CDN using your `cdn-uploading` skill.

## Client-Side Routing Configuration

Vue apps using Vue Router need special handling for HTML5 history mode.

### Important: SPA Fallback

Configure your CDN/server to serve `index.html` for all routes that don't match a static file.

**Cloudflare Pages:** Automatic SPA support
**AWS S3 + CloudFront:** Configure custom error response for 404s to serve `index.html`
**Nginx:**
```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### Hash Mode Alternative

If using hash mode (`createWebHashHistory()`), no server configuration needed - all routes use `/#/` prefix.

## Environment Variables

Vue apps use environment variables at **build time**.

### Vite Vue
- Variables must be prefixed with `VITE_`
- Access via `import.meta.env.VITE_API_URL`
- Set before build: `VITE_API_URL=https://api.example.com npm run build`

### Vue CLI
- Variables must be prefixed with `VUE_APP_`
- Access via `process.env.VUE_APP_API_URL`
- Set before build: `VUE_APP_API_URL=https://api.example.com npm run build`
- Supports `.env`, `.env.production`, `.env.local` files

## Base Path / Public Path

If deploying to a subdirectory (not root):

### Vite Vue
Set `base` in `vite.config.js`:
```javascript
export default defineConfig({
  base: '/my-app/',
  plugins: [vue()]
})
```

### Vue CLI
Set `publicPath` in `vue.config.js`:
```javascript
module.exports = {
  publicPath: '/my-app/'
}
```
Or via environment variable: `BASE_URL=/my-app/ npm run build`

## Vue Router Base Configuration

When using a base path, also configure Vue Router:

```javascript
const router = createRouter({
  history: createWebHistory('/my-app/'),
  routes: [...]
})
```

## Monorepo Support

For monorepos using Turborepo, Nx, or pnpm workspaces:

1. Identify the app directory (e.g., `apps/web`, `packages/frontend`)
2. Run build from root: `turbo run build --filter=web` or `nx build web`
3. Output will be in the app's output directory (e.g., `apps/web/dist`)

## PWA Support

If using `@vue/pwa` or `vite-plugin-pwa`:
- Service worker files will be in the output directory
- Ensure all PWA assets are uploaded to CDN
- Configure proper cache headers for service worker

## Common Issues

### Assets not loading
- Check `base` (Vite) or `publicPath` (Vue CLI) configuration
- Ensure router base matches deployment path

### Routes returning 404
- Configure SPA fallback on CDN/server
- Check if using history mode vs hash mode
- Verify router base path configuration

### Environment variables undefined
- Ensure correct prefix (`VITE_` or `VUE_APP_`)
- Variables must be set at build time
- Check `.env` file naming (`.env.production` for production builds)

### Build failing with TypeScript errors
- Run `vue-tsc --noEmit` to check types before build
- Ensure `tsconfig.json` includes all Vue files

## When to Use This Skill

- Deploying Vue.js SPAs built with Vite
- Deploying Vue CLI projects
- Any Vue.js application that compiles to static files
- For Vue with SSR/SSG, use the `nuxt-deployment` skill instead
