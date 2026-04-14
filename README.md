# Semantic Release Creator Action
[![CI](https://github.com/mherman22/semantic-release-creator-action/actions/workflows/ci.yml/badge.svg)](https://github.com/mherman22/semantic-release-creator-action/actions/workflows/ci.yml) [![Simple Test Suite](https://github.com/mherman22/semantic-release-creator-action/actions/workflows/test.yml/badge.svg)](https://github.com/mherman22/semantic-release-creator-action/actions/workflows/test.yml) [![Release](https://github.com/mherman22/semantic-release-creator-action/actions/workflows/release.yml/badge.svg)](https://github.com/mherman22/semantic-release-creator-action/actions/workflows/release.yml)

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

### With Configuration File

```yaml
- name: Create release
  uses: mherman22/create-semantic-release-action@v1
  with:
    release-tag: ${{ steps.bump.outputs.release-tag }}
    changelog: ${{ steps.changelog.outputs.release-notes }}
    stage-label: ${{ steps.bump.outputs.stage-label }}
    config-file: 'release-config.json'  # ✅ Use custom configuration
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
| `config-file` | Path to configuration file (JSON) | No | - |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `release-url` | URL of the created GitHub release | `https://github.com/owner/repo/releases/tag/v1.2.3` |
| `release-id` | ID of the created GitHub release | `123456789` |

## Configuration

You can customize the action's behavior using a JSON configuration file. This allows you to:
- Define custom file paths for version files
- Customize commit message templates
- Support additional file types beyond Maven and Node.js
- Configure versioning behavior

### Configuration File Format

Create a `config.json` file in your repository:

```json
{
  "files": {
    "maven": {
      "path": "backend/pom.xml"
    },
    "nodejs": {
      "path": "frontend/package.json"
    },
    "custom": [
      {
        "path": "Chart.yaml",
        "versionPattern": "version:\\s*([^\\n]*)",
        "updateTemplate": "version: {version}"
      }
    ]
  },
  "commits": {
    "releaseTemplate": "🚀 Release {version}",
    "nextDevTemplate": "🔧 Prepare next development iteration {version}"
  },
  "tags": {
    "prefix": "v",
    "annotated": true
  },
  "nextDev": {
    "maven": {
      "appendSnapshot": true
    },
    "nodejs": {
      "appendSnapshot": false
    }
  }
}
```

### Configuration Options

| Section | Field | Description | Default |
|---------|-------|-------------|---------|
| `files.maven.path` | Maven POM file location | Path to pom.xml | `pom.xml` |
| `files.nodejs.path` | Node.js package file location | Path to package.json | `frontend/package.json` |
| `files.custom[]` | Additional version files | Array of custom file configurations | `[]` |
| `commits.releaseTemplate` | Release commit message | Template with `{version}` placeholder | `chore(release): bump version to {version}` |
| `commits.nextDevTemplate` | Next dev commit message | Template with `{version}` placeholder | `chore: prepare next development iteration {version}` |
| `tags.prefix` | Tag prefix | Prefix for release tags | `` |
| `nextDev.maven.appendSnapshot` | Maven SNAPSHOT suffix | Add `-SNAPSHOT` to Maven versions | `true` |
| `nextDev.nodejs.appendSnapshot` | Node.js SNAPSHOT suffix | Add `-SNAPSHOT` to Node.js versions | `false` |

### Example Configurations

**Helm Chart Support:**
```json
{
  "files": {
    "custom": [
      {
        "path": "Chart.yaml",
        "versionPattern": "version:\\s*([^\\n]*)",
        "updateTemplate": "version: {version}"
      }
    ]
  }
}
```

**Docker Label Support:**
```json
{
  "files": {
    "custom": [
      {
        "path": "Dockerfile",
        "versionPattern": "LABEL version=\"([^\"]*)\"\n",
        "updateTemplate": "LABEL version=\"{version}\""
      }
    ]
  }
}
```

**Custom Commit Messages:**
```json
{
  "commits": {
    "releaseTemplate": "🚀 Release version {version}",
    "nextDevTemplate": "🔧 Next iteration: {version}"
  }
}
```

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

## Testing

This action includes comprehensive test suites to ensure reliability across different project types and scenarios.

### Test Coverage

The test suite covers:

- ✅ **Maven-only projects** (`pom.xml` only)
- ✅ **Node.js-only projects** (`package.json` only) 
- ✅ **Hybrid projects** (both `pom.xml` and `package.json`)
- ✅ **Different branches** (`main`, `develop`, custom branches)
- ✅ **All release stages** (`alpha`, `beta`, `final`, `snapshot`)
- ✅ **Next development versions** (for final releases)
- ✅ **Error handling** (missing files, malformed configs)
- ✅ **Edge cases** (special characters, unicode, large changelogs)

### Running Tests Locally

The action uses GitHub Actions workflows for testing. To run tests:

1. **Fork the repository**
2. **Enable Actions** in your fork
3. **Run test workflows** via Actions tab:
   - `Test Suite` - Core functionality tests
   - `Integration Tests` - Real GitHub API tests (requires PAT_TOKEN secret)

### Test Structure

```
.github/workflows/
├── test.yml              # Core functionality tests
├── integration-test.yml  # Real API integration tests
└── ci.yml                # Continuous integration
```

### Mock vs Real Testing

**Mock Tests (test.yml):**
- Fast execution (< 5 minutes)
- No external dependencies
- Tests action logic and file handling
- Runs on every push/PR

**Integration Tests (integration-test.yml):**
- Real GitHub API calls
- Creates/deletes test repositories
- Tests actual release creation
- Runs daily or on manual trigger
- Requires `PAT_TOKEN` secret with repo permissions

### Adding New Tests

When adding new functionality, please:

1. Add unit tests to `test.yml`
2. Consider integration tests for GitHub API changes
3. Test both success and failure scenarios
4. Include edge cases and error conditions

### Test Project Examples

The test suite creates various project types:

**Maven Project:**
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>test-maven</artifactId>
    <version>1.0.0</version>
</project>
```

**Node.js Project:**
```json
{
  "name": "test-nodejs",
  "version": "2.0.0",
  "description": "Test Node.js project"
}
```

**Hybrid Project:**
```
project/
├── pom.xml              # Backend version
└── frontend/
    └── package.json     # Frontend version
```

### Common Test Scenarios

| Scenario | Input | Expected Output |
|----------|-------|----------------|
| Maven release | `pom.xml` with `1.0.0` → `v1.1.0` | Updated `<version>v1.1.0</version>` |
| Node.js release | `package.json` with `2.0.0` → `v2.1.0` | Updated `"version": "v2.1.0"` |
| Final release | `stage-label: final`, `next-dev-version: 2.0.0-SNAPSHOT` | Two commits: release + next dev |
| Missing files | No `pom.xml` or `package.json` | Graceful failure with clear error |
| Custom branch | `branch: main` | Push to `main` instead of `develop` |

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
