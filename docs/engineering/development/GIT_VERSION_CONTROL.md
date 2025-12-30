# Git Version Control Guidelines

## Overview

This document defines the Git workflow and commit message standards for Nexus development. Following these guidelines ensures clean, maintainable history and facilitates code review.

## Commit Message Format

### Conventional Commits

Nexus follows the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

### Commit Types

| Type | Description | Examples |
|------|-------------|----------|
| `feat` | New feature | `feat: add user registration` |
| `fix` | Bug fix | `fix: handle null email validation` |
| `docs` | Documentation only | `docs: update README installation` |
| `style` | Code style changes (formatting) | `style: format with prettier` |
| `refactor` | Code refactoring | `refactor: extract validation logic` |
| `perf` | Performance improvements | `perf: optimize query caching` |
| `test` | Adding or updating tests | `test: add integration tests for auth` |
| `build` | Build system changes | `build: update webpack config` |
| `ci` | CI/CD changes | `ci: add GitHub Actions workflow` |
| `chore` | Other changes | `chore: update dependencies` |
| `revert` | Revert previous commit | `revert: feat(user): remove deprecated field` |

### Scopes

Scopes indicate the area of codebase affected:

```
feat(auth): add JWT token refresh
fix(database): handle connection timeout
refactor(user): extract password hashing
test(api): add endpoint tests
docs(cli): update help text
```

Common scopes:
- `auth` - Authentication system
- `user` - User management
- `cli` - CLI interface
- `api` - REST API
- `database` - Database layer
- `ai` - AI provider integration
- `migration` - Database migrations
- `agent` - AI agents
- `config` - Configuration files
- `test` - Testing framework

### Message Body

Use imperative mood, present tense:

```
Add blank line between subject and body

- Explain WHY the change is made
- Use imperative mood ("add" not "added")
- List related issues
```

### Message Footer

Optional footer for:

```
Refs #123
Closes #456
Breaking change: removes deprecated API
```

## Commit Message Examples

### Good Commit Messages

```bash
feat(auth): implement JWT token refresh mechanism

Add refresh token generation and validation to support
long-lived sessions. Tokens rotate on each refresh for
security.

- Add generateRefreshToken utility
- Add validateRefreshToken middleware
- Implement /auth/refresh endpoint
- Add refresh token tests

Closes #123
```

```bash
fix(user): prevent duplicate email registration

Add unique constraint check on email field to prevent
multiple users with same email address. Returns proper
error message for existing emails.

Refs #456
```

```bash
refactor(query): extract common where clause logic

Move WHERE clause construction to separate method to
reduce duplication across query builder methods.
Improves maintainability.

BREAKING CHANGE: QueryBuilder.where() now returns
fluent interface instead of void
```

### Bad Commit Messages

❌ `Update files`
❌ `Fix bug`
❌ `WIP`
❌ `Update code`
❌ `Merge branch feature`
❌ `Initial commit` (unless truly initial)

## Commit Checklist

Before committing, verify:

- [ ] All tests passing (`yarn test`)
- [ ] Type checking passes (`yarn typecheck`)
- [ ] Linting passes (`yarn lint`)
- [ ] Code formatted (`yarn format`)
- [ ] Commit message follows format
- [ ] Commit message uses imperative mood
- [ ] Commit message explains WHY, not just WHAT
- [ ] No unrelated changes in commit
- [ ] Sensitive data not included (API keys, passwords)
- [ ] Large files committed (check file size < 5MB)

## Branch Naming

### Feature Branches

```
feat/<feature-name>
feat/user-registration
feat/jwt-auth
```

### Bugfix Branches

```
fix/<bug-description>
fix/email-validation-error
fix/database-timeout
```

### Hotfix Branches

```
hotfix/<hotfix-description>
hotfix/security-patch-2024-01
hotfix/critical-bug-123
```

### Release Branches

```
release/<version>
release/v1.0.0
release/v1.1.0
```

## Git Workflow

### Feature Development Flow

```bash
# Start from latest main
git checkout main
git pull

# Create feature branch
git checkout -b feat/new-feature

# Make changes, commit frequently
git add .
git commit -m "feat: add new feature"

# Rebase to keep history clean
git fetch origin
git rebase origin/main

# Push and create PR
git push -u origin feat/new-feature
```

### Commit Frequency

**Small, focused commits** are better:

- ✅ One logical change per commit
- ✅ Max 150 lines per commit (matches step limits)
- ✅ Reviewable in < 5 minutes
- ✅ Clear, descriptive messages
- ❌ Large commits with multiple features
- ❌ "Work in progress" commits
- ❌ Committing broken code

### Squashing Commits

Before merging, consider squashing related commits:

```bash
# Interactive rebase
git rebase -i HEAD~5

# Mark commits to squash (s) or keep (pick)
# Edit combined commit message
```

When to squash:
- Multiple small commits for single feature
- "WIP" or "fix typo" commits during development
- Commits that build on each other

## Commit Message Enforcement

### Pre-commit Hook

Husky pre-commit hook validates:

1. Commit message format (conventional commits)
2. Test suite passing
3. Linting passing
4. No large files (> 5MB)

### Commitlint Configuration

```javascript
// commitlint.config.js
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert'],
    ],
    'subject-case': [0],
    'subject-max-length': [2, 'always', 72],
    'body-max-line-length': [2, 'always', 100],
  },
};
```

## Git History Best Practices

### Clean History

- **Rewrite history** on feature branches before merging
- **Never force push** to main/master
- **Use merge commits** only when necessary
- **Prefer rebase** over merge for feature branches

```bash
# Good: Rebase feature branch
git checkout feat/feature
git rebase main

# Bad: Merge feature into main
git checkout main
git merge feat/feature
```

### Commit Significance

Each commit should:
- Be **releasable** independently
- Pass all tests
- Be **reversible** (can revert cleanly)
- Explain its purpose clearly

## Version Tags

Follow semantic versioning for releases:

```bash
# Tag release
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# Format: v<MAJOR>.<MINOR>.<PATCH>
# v1.0.0 - Initial release
# v1.1.0 - New feature (backward compatible)
# v1.1.1 - Bug fix (backward compatible)
# v2.0.0 - Breaking changes
```

## .gitignore Best Practices

**Never commit**:
- API keys and secrets (`.env`, `secrets/`)
- Build artifacts (`dist/`, `build/`)
- Dependencies (`node_modules/`)
- IDE files (`.idea/`, `.vscode/`)
- OS files (`.DS_Store`, `Thumbs.db`)
- Test coverage reports
- Large files (> 100KB, use Git LFS)
- Temporary files

**Always commit**:
- Configuration templates (`.env.example`)
- Documentation
- Type definitions
- Build scripts
- Test fixtures

## Code Review Guidelines

### PR Description Format

```markdown
## Summary
Brief description of changes

## Changes
- List of major changes
- Impact of changes

## Testing
- Tests added/updated
- Manual testing performed

## Checklist
- [ ] Tests passing
- [ ] Linting passing
- [ ] Documentation updated
- [ ] No breaking changes (or noted)
```

### Review Criteria

Reviewers check:
- Code follows project conventions
- Tests are comprehensive
- Commit messages are clear
- No security vulnerabilities
- Performance impact considered
- Documentation updated
- Breaking changes documented

## References

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)

---

**Remember**: Clear commit messages improve code review speed, enable easy rollback, and create a maintainable project history.
