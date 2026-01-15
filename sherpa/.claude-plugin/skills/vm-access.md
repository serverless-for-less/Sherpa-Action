---
name: vm-access
description: SSH into VMs for backend deployments. Handles connection setup and command execution using SSH keys from environment variables.
---

Access VMs using SSH. Keys are provided via environment variables—never hardcode credentials.

## When to Use This Skill

- Connecting to VMs for any backend deployment
- Running commands on remote servers
- Use alongside framework-specific skills (e.g., `nodejs-backend`) for full deployments

## SSH Access Pattern

### Setup

The SSH private key is provided via `$SSH_PRIVATE_KEY` environment variable. Before SSH:

```bash
# Write key to temp file with correct permissions
echo "$SSH_PRIVATE_KEY" > /tmp/deploy_key
chmod 600 /tmp/deploy_key
```

### Connecting

```bash
# First connection (accept host key)
ssh -i /tmp/deploy_key -o StrictHostKeyChecking=accept-new user@$VM_IP

# Run a single command
ssh -i /tmp/deploy_key user@$VM_IP "whoami"

# Run multiple commands with heredoc
ssh -i /tmp/deploy_key user@$VM_IP << 'EOF'
  cd /app
  git pull
  # ... more commands
EOF
```

### Cleanup

```bash
rm -f /tmp/deploy_key
```

## Reading VM Info

VM details are stored in `.sherpa.sh/infrastructure.md`. Read the IP address from there:

```markdown
## VM: my-app-server
- **Provider:** AWS EC2
- **Instance ID:** i-0abc123def456
- **IP Address:** 54.123.45.67
- **Region:** us-east-1
- **Size:** t3.small
```

## Best Practices

1. **Never hardcode keys** - Always use `$SSH_PRIVATE_KEY` from environment
2. **Use temp files** - Write key to `/tmp/deploy_key`, delete after use
3. **Set permissions** - Always `chmod 600` on key file
4. **Accept host keys** - Use `-o StrictHostKeyChecking=accept-new` on first connect
5. **Use heredocs** - For multi-command deployments, use `<< 'EOF'` syntax

## Security Considerations

- SSH keys in `$SSH_PRIVATE_KEY` are never written to logs or `.sherpa.sh/` files
- Only reference keys by environment variable name in documentation
- Clean up temp key files immediately after use
- Use ed25519 keys for better security (smaller, faster, more secure than RSA)