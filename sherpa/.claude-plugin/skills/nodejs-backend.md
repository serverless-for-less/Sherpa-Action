---
name: nodejs-backend
description: Deploy Node.js applications (Next.js SSR, Express, etc.) to VMs. Handles Node installation, PM2 process management, and deployment workflows.
---

Deploy Node.js backend applications to VMs. Use with the `vm-access` skill for SSH connectivity.

## Initial Setup (First Deploy)

Install Node.js, PM2, and set up the application:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  # Install Node.js 20.x
  curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
  sudo apt-get install -y nodejs

  # Install PM2 globally
  sudo npm install -g pm2

  # Clone and build
  git clone https://github.com/$REPO.git /app
  cd /app
  npm install
  npm run build

  # Start with PM2
  pm2 start npm --name "app" -- start
  pm2 save
  pm2 startup
EOF
```

## Subsequent Deploys

Pull latest code and restart:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app
  git pull origin main
  npm install
  npm run build
  pm2 restart app
EOF
```

## PM2 Commands

```bash
# View running processes
pm2 list

# View logs
pm2 logs app

# Restart application
pm2 restart app

# Stop application
pm2 stop app

# Delete from PM2
pm2 delete app

# Monitor resources
pm2 monit
```

## Next.js Specific

For Next.js with SSR, ensure the start script runs the production server:

```bash
# In package.json, "start" should be: "next start"
pm2 start npm --name "nextjs-app" -- start

# Or start directly
pm2 start node_modules/.bin/next --name "nextjs-app" -- start -p 3000
```

## Environment Variables

Set environment variables for the Node.js app:

```bash
# Create .env file on the VM
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cat > /app/.env << 'ENVEOF'
DATABASE_URL=postgres://...
NODE_ENV=production
ENVEOF
EOF

# Or pass via PM2 ecosystem file
pm2 start ecosystem.config.js
```

## Nginx Reverse Proxy (Optional)

For production, put Nginx in front of Node.js:

```bash
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  sudo apt-get install -y nginx

  sudo tee /etc/nginx/sites-available/app << 'NGINX'
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
NGINX

  sudo ln -sf /etc/nginx/sites-available/app /etc/nginx/sites-enabled/
  sudo rm -f /etc/nginx/sites-enabled/default
  sudo nginx -t && sudo systemctl reload nginx
EOF
```

## When to Use This Skill

- Deploying Next.js apps with server-side rendering (SSR)
- Deploying Express.js APIs
- Deploying any Node.js application that needs to run persistently
- Managing Node.js processes with PM2
