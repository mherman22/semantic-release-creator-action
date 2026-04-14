# Semantic Release Creator Action

A GitHub Action that handles Git operations and GitHub release publishing for semantic releases. Takes pre-generated changelog content and creates complete releases with proper Git tagging and development cycle management.

## Features

- ✅ **Git Operations** - Automatic commits and tagging
- ✅ **GitHub Releases** - Creates releases using provided changelog content
- ✅ **Development Cycle** - Prepares next dev version for final releases  
- ✅ **Flexible** - Works with any project structure
- ✅ **Safe** - Handles tag conflicts and edge cases

## What This Action Does vs Doesn't Do

**✅ This action handles:**
- Git commits for version changes
- Git tagging and pushing
- GitHub release creation using provided content
- Next development version preparation (for final releases)

**❌ This action does NOT:**
- Generate changelog content (use a separate changelog action first)
- Analyze commits for version bumping (use a version bump action first)
- Determine release versions (versions must be provided as inputs)

## Usage

### Basic Usage

```yaml
- name: Create release
  uses: mherman22/create-semantic-release-action@v1
  with:
    release-tag: ${{ steps.bump.outputs.release-tag }}
    changelog: ${{ steps.changelog.outputs.release-notes }}
    stage-label: ${{ steps.bump.outputs.stage-label }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### With Next Development Version

```yaml
- name: Create release
  uses: mherman22/create-semantic-release-action@v1
  with:
    release-tag: ${{ steps.bump.outputs.release-tag }}
    changelog: ${{ steps.changelog.outputs.release-notes }}
    next-dev-version: ${{ steps.bump.outputs.next-dev-version }}
    stage-label: ${{ steps.bump.outputs.stage-label }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### With Custom Branch (for repositories using 'main' branch)

```yaml
- name: Create release
  uses: mherman22/create-semantic-release-action@v1
  with:
    release-tag: ${{ steps.bump.outputs.release-tag }}
    changelog: ${{ steps.changelog.outputs.release-notes }}
    stage-label: ${{ steps.bump.outputs.stage-label }}
    branch: 'main'  # ✅ Push to main instead of develop
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `release-tag` | The release tag to create | Yes | - |
| `changelog` | The changelog/release notes content | Yes | - |
| `stage-label` | Release stage label (alpha, beta, final, snapshot) | Yes | - |
| `github-token` | GitHub token for creating releases | Yes | - |
| `next-dev-version` | Next development version (for final releases) | No | - |
| `repository` | Repository name (owner/repo) | No | `${{ github.repository }}` |
| `backend-file` | Path to Maven pom.xml file | No | `pom.xml` |
| `frontend-file` | Path to Node.js package.json file | No | `frontend/package.json` |
| `branch` | Branch to push changes to | No | `develop` |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `release-url` | URL of the created GitHub release | `https://github.com/owner/repo/releases/tag/v1.2.3` |
| `release-id` | ID of the created GitHub release | `123456789` |

## Behavior

### Git Operations
1. **Commits version changes** - Stages and commits updated version files
2. **Creates annotated tag** - Creates Git tag with release message
3. **Pushes to origin** - Pushes both commits and tags

### Next Development Version (Final Releases Only)
1. **Updates backend** - Sets next SNAPSHOT version in pom.xml  
2. **Updates frontend** - Sets next semantic version in package.json
3. **Commits changes** - Creates "next development iteration" commit
4. **Pushes changes** - Updates the develop branch

### GitHub Release
1. **Creates release** - Uses the provided tag and changelog
2. **Sets prerelease** - Marks alpha/beta as prerelease, final as stable
3. **Returns metadata** - Provides release URL and ID for further processing

## Workflow Integration

This action is designed to be the **final step** in a release automation workflow:

1. **Step 1**: Version bumping (e.g., `semantic-version-bump-action`)
2. **Step 2**: Changelog generation (e.g., `johnyherangi/create-release-notes`)  
3. **Step 3**: Release creation (this action)

## Complete Workflow Example

```yaml
name: Automated Release
on:
  workflow_dispatch:
    inputs:
      version_type:
        description: 'Version bump type'
        type: choice
        options: [fix, minor, major]
        default: fix
      release_stage:
        description: 'Release stage'
        type: choice
        options: [alpha, beta, final]
        default: alpha

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Bump version
        id: bump
        uses: mherman22/semantic-version-bump-action@v1
        with:
          version-type: ${{ inputs.version_type }}
          release-stage: ${{ inputs.release_stage }}

      - name: Generate changelog
        id: changelog
        uses: johnyherangi/create-release-notes@main
        with:
          head-ref: ${{ github.ref }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Create release
        uses: mherman22/create-semantic-release-action@v1
        with:
          release-tag: ${{ steps.bump.outputs.release-tag }}
          changelog: ${{ steps.changelog.outputs.release-notes }}
          next-dev-version: ${{ steps.bump.outputs.next-dev-version }}
          stage-label: ${{ steps.bump.outputs.stage-label }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Contributing

1. Fork the repository
2. Create a feature branch  
3. Make your changes
4. Add tests if needed
5. Submit a pull request

## Support

If you encounter any issues or have feature requests, please [open an issue](../../issues).