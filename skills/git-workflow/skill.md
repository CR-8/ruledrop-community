Apply consistent Git conventions for commits, branches, and pull requests. Use this skill when writing commit messages, naming branches, structuring PRs, or setting up a team Git workflow.

## Commit Messages

Follow the Conventional Commits format:

```
<type>(<scope>): <short summary>

[optional body]

[optional footer]
```

**Types:**

| Type | When |
|------|------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code restructure, no behavior change |
| `perf` | Performance improvement |
| `test` | Adding or fixing tests |
| `chore` | Build, deps, tooling |
| `ci` | CI/CD changes |
| `revert` | Reverting a previous commit |

**Rules:**
- Summary line: 50 chars max, imperative mood ("add feature" not "added feature")
- No period at the end of the summary
- Body: wrap at 72 chars, explain *why* not *what*
- Reference issues in footer: `Closes #123`, `Fixes #456`

**Examples:**

```
feat(auth): add JWT refresh token rotation

Tokens now rotate on each refresh to limit the window of exposure
if a refresh token is stolen. Old tokens are invalidated immediately.

Closes #89
```

```
fix(api): return 404 when user not found instead of 500

The user lookup was throwing an unhandled exception when the ID
didn't exist. Added explicit not-found check before the DB query.
```

```
chore(deps): upgrade express from 4.18 to 4.19
```

## Branch Naming

```
<type>/<short-description>
<type>/<issue-id>-<short-description>
```

**Examples:**
```
feat/user-authentication
fix/42-login-redirect-loop
docs/api-reference-update
chore/upgrade-node-20
release/v2.1.0
```

Rules:
- Lowercase, kebab-case only
- No special characters except `/` and `-`
- Keep it short but descriptive (3–5 words max)
- Include issue ID when one exists

## Pull Requests

**Title** — same format as a commit message: `type(scope): summary`

**Description template:**

```markdown
## What
Brief description of what changed and why.

## How
Key implementation decisions. What approach was taken and why.

## Testing
How was this tested? What cases were covered?

## Screenshots (if UI change)
Before / After

## Checklist
- [ ] Tests added/updated
- [ ] Docs updated if needed
- [ ] No console.log or debug code left in
```

**PR size:** Keep PRs small and focused. One concern per PR. If a PR touches more than ~400 lines, consider splitting it. Reviewers can't give good feedback on massive diffs.

## Branching Strategy

### GitHub Flow (recommended for most teams)
- `main` is always deployable
- Feature branches off `main`, merged back via PR
- Deploy from `main` after merge

```
main ──────────────────────────────────────►
         └── feat/login ──► PR ──► merge
```

### Git Flow (for versioned releases)
- `main` — production releases only
- `develop` — integration branch
- `feature/*` — off develop
- `release/*` — stabilization before merge to main
- `hotfix/*` — off main, merged to both main and develop

Use GitHub Flow unless you have explicit versioned release cycles.

## Useful Patterns

**Squash on merge** — keep `main` history clean. One PR = one commit.

**Rebase feature branches** before merging to avoid merge commits:
```bash
git fetch origin
git rebase origin/main
```

**Atomic commits** — each commit should leave the codebase in a working state. Don't commit half-finished work to shared branches.

**Never force-push to shared branches** (`main`, `develop`). Force-push is fine on your own feature branch.

## Tags & Releases

Use semantic versioning: `MAJOR.MINOR.PATCH`

- `MAJOR` — breaking changes
- `MINOR` — new features, backward compatible
- `PATCH` — bug fixes, backward compatible

Tag releases: `git tag -a v1.2.0 -m "Release v1.2.0"`
