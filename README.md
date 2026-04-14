# Create Semantic Release Action

A GitHub Action that handles the complete release creation process including Git tagging, version commits, next development version preparation, and GitHub release publishing.

## Features

- ✅ **Git Operations** - Automatic commits and tagging
- ✅ **GitHub Releases** - Creates releases with changelog
- ✅ **Development Cycle** - Prepares next dev version for final releases  
- ✅ **Flexible** - Works with any project structure
- ✅ **Safe** - Handles tag conflicts and edge cases

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