# Release Workflows Quick Reference

## Production Release (Manual)

**Workflow:** `.github/workflows/release.yml` (production-release job)  
**Trigger:** Manual via GitHub Actions UI  
**Purpose:** Automated semantic versioning and npm publishing

### How to Run

1. Navigate to **Actions** → **Release**
2. Click **Run workflow**
3. Select branch: `main`
4. (Optional) Check **Dry run** to preview without publishing
5. Click **Run workflow**

### What It Does

1. ✅ Runs tests to ensure quality
2. 📊 Analyzes commits using conventional commits
3. 🔢 Determines version bump (major.minor.patch)
4. 📝 Updates CHANGELOG.md
5. 📦 Publishes to npm with `latest` tag
6. 🏷️ Creates git tag (e.g., `v1.2.3`)
7. 📢 Creates GitHub Release with notes

### Requirements

- Commits must follow [Conventional Commits](https://www.conventionalcommits.org/)
- Secrets: `NPM_TOKEN` with publish permissions
- Must run from `main` branch

---

## Dev/Nightly Release (Manual)

**Workflow:** `.github/workflows/dev-release.yml` → calls `release.yml` (dev-release job)  
**Trigger:** Manual via GitHub Actions UI  
**Purpose:** Publish pre-release versions for testing

### How to Run

1. Navigate to **Actions** → **Dev Release**
2. Click **Run workflow**
3. Configure inputs:
   - **Branch:** `main`, `dev`, or `next`
   - **NPM tag:** `dev`, `next`, `alpha`, `beta`, `canary`, or `nightly`
   - **Custom NPM tag:** (Optional) e.g., `v4.1.x` - overrides dropdown
   - **Version suffix:** `dev`, `nightly`, `alpha`, `beta`, `canary`, or `next`
4. Click **Run workflow**

### What It Does

1. ✅ Runs tests from selected branch
2. 🏷️ Generates timestamped version (e.g., `0.0.1-dev.20260109123456.abc1234`)
3. 📦 Publishes to npm with specified tag
4. 🏷️ Creates git tag for reference
5. 📊 Shows summary with install command

### Version Format

```
{current-version}-{suffix}.{timestamp}.{commit-sha}
```

**Examples:**
- `0.0.1-dev.20260109123456.abc1234`
- `0.0.1-nightly.20260109123456`
- `1.2.3-alpha.20260109123456.def5678`

### Installing Dev Releases

```bash
# Latest dev version
npm install @stencil/vitest@dev

# Latest nightly version  
npm install @stencil/vitest@nightly

# Specific custom tag
npm install @stencil/vitest@v4.1.x

# Exact version
npm install @stencil/vitest@0.0.1-dev.20260109123456.abc1234
```

---

## Workflow Architecture

```
┌─────────────────────────────────────────────────────────┐
│  .github/workflows/release.yml (Trusted Workflow)       │
│  ─────────────────────────────────────────────────────  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  production-release (workflow_dispatch)          │  │
│  │  • Runs semantic-release                         │  │
│  │  • Auto-versioning via conventional commits      │  │
│  │  • Publishes to npm@latest                       │  │
│  │  • Creates GitHub Release                        │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  dev-release (workflow_call)                     │  │
│  │  • Manual versioning with timestamp              │  │
│  │  • Publishes to custom npm tag                   │  │
│  │  • Creates git tag only (no GH release)          │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
                         ▲
                         │ calls
                         │
┌─────────────────────────────────────────────────────────┐
│  .github/workflows/dev-release.yml                      │
│  ─────────────────────────────────────────────────────  │
│  • User-friendly UI for dev releases                    │
│  • Input validation and defaults                        │
│  • Calls release.yml's dev-release job                  │
└─────────────────────────────────────────────────────────┘
```

---

## Security

- **Trusted Publishing:** Uses npm provenance for supply chain security
- **Minimal Permissions:** Each job gets only required GitHub token permissions
- **Concurrency Control:** Prevents simultaneous releases from corrupting state
- **Branch Protection:** Production releases only from `main` via semantic-release config

---

## Troubleshooting

### Production Release Not Creating a Release

**Possible causes:**
1. No conventional commits since last release
2. Only `chore:`, `docs:`, or similar non-release commits
3. Dry run mode is enabled

**Solution:**
- Ensure commits use `feat:`, `fix:`, or breaking change markers
- Check semantic-release output for analysis results

### Dev Release Fails to Publish

**Possible causes:**
1. `NPM_TOKEN` not configured or expired
2. Package name already exists with that exact version
3. Network issues with npm registry

**Solution:**
- Verify `NPM_TOKEN` is set in repository secrets
- Check npm registry status
- Re-run the workflow

### Version Not Incrementing

**Production releases:**
- Semantic-release controls versioning based on commit messages
- Use `feat:` for minor, `fix:` for patch, `BREAKING CHANGE:` for major

**Dev releases:**
- Version is based on current package.json version + timestamp
- Update `package.json` version manually if needed

---

## Best Practices

1. **Use dev releases for testing** - Test features before merging to main
2. **Write conventional commits** - Required for automated versioning
3. **Test before releasing** - All workflows run tests automatically
4. **Use dry run** - Preview production releases before executing
5. **Document breaking changes** - Use `BREAKING CHANGE:` footer in commits
6. **Clean up old dev tags** - Remove obsolete npm tags periodically

---

## Related Documentation

- [Semantic Release Docs](https://semantic-release.gitbook.io/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [npm Provenance](https://docs.npmjs.com/generating-provenance-statements)
- [GitHub Actions workflow_call](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
