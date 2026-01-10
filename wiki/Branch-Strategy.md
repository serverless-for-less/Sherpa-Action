# Branch Strategy

## Main Branch as Stable Source

Your `main` (or `master`) branch should always represent a stable, production-ready version of your action. All development should happen on feature branches before merging to main, following these principles:

- **Create feature branches with descriptive names** like `feature/add-caching` or `bugfix/token-permissions`

- **Keep branches short-lived** and merge frequently to reduce conflicts

- **Use pull requests for all changes** with peer review and automated testing

- **Never deploy directly from the default branch** - users should reference specific versions instead

## Release Branches (Optional)

For complex actions, consider using release branches like `release/v1` or `release/v2` to stabilize code before tagging. This allows you to:

- Test and verify releases independently from active development

- Apply hotfixes to older major versions without affecting newer releases

- Create a clear separation between development and release-ready code

## Branch Naming Conventions

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New functionality | `feature/aws-support` |
| `bugfix/` | Bug fixes | `bugfix/token-permissions` |
| `hotfix/` | Urgent production fixes | `hotfix/security-patch` |
| `release/` | Release stabilization | `release/v2` |

## Pull Request Guidelines

1. Reference any related issues in the PR description
2. Keep PRs focused on a single change
3. Ensure all CI checks pass before requesting review
4. Squash commits when merging to keep history clean
