# Versioning and Releases

## Semantic Versioning with Major Version Tags

The most important distribution practice is implementing semantic versioning with moving major version tags. This is the standard approach used by GitHub's official actions.

### How It Works

1. **Create specific release tags** for each version (e.g., `v1.0.0`, `v1.0.1`, `v1.2.0`)

2. **Maintain major version tags** that point to the latest compatible release:

```bash
# After releasing v1.2.0, update the v1 tag
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```

This allows users to reference your action as `uses: sherpa-sh/sherpa-action@v1` and automatically receive backwards-compatible updates, while still having the option to pin to specific versions like `v1.0.1` for stability.

## Version Increment Guidelines

| Change Type | Version Bump | Example | When to Use |
|-------------|--------------|---------|-------------|
| **Patch** | `v1.0.x` | `v1.0.0` → `v1.0.1` | Bug fixes and minor improvements that are fully backwards-compatible |
| **Minor** | `v1.x.0` | `v1.0.1` → `v1.1.0` | New features that remain backwards-compatible |
| **Major** | `vx.0.0` | `v1.1.0` → `v2.0.0` | Breaking changes that modify inputs, outputs, or behavior in incompatible ways |

## Beta and Pre-release Tags

For major version transitions, use beta tags like `v2-beta` to allow early adopters to test breaking changes before the official release.

```bash
# Create a beta tag
git tag v2.0.0-beta
git push origin v2.0.0-beta
```

Users can test with:
```yaml
uses: sherpa-sh/sherpa-action@v2.0.0-beta
```

Remove the `-beta` suffix when ready for production.

## Release Checklist

- [ ] All tests passing
- [ ] CHANGELOG updated
- [ ] Version tag created (`v1.2.3`)
- [ ] Major version tag updated (`v1`)
- [ ] GitHub Release created with release notes

## Creating a Release

```bash
# 1. Create the specific version tag
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0

# 2. Update the major version tag
git tag -fa v1 -m "Update v1 tag to v1.2.0"
git push origin v1 --force

# 3. Create a GitHub Release via the web UI or gh CLI
gh release create v1.2.0 --title "v1.2.0" --notes "Release notes here"
```
