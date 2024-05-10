# CI/CD GitHub Actions

Handles continues integration / continues development

## Versioning

The project uses [Semantic versioning](https://semver.org/), and the version is determined using [Semantic release](https://semantic-release.gitbook.io/semantic-release/).

You can use the version of the release, to lock in the version of the action, that you want to use.

Please only use `main` as the version, if you want to use the latest version, as it may be unstable or updated in a way that breaks your workflow.

## Available actions

A wide range of actions are available, from Semver to linting.

- [Semver release](#semver-release)
- [Docker build and push](#docker-build-and-push)
- [PR format test - Conventional commits](#pr-format-test---conventional-commits)

### Semver release

This action will run Semantic Release to determine the next version of the package and create a new release.

#### Pre-requisites

Semver release pipeline requires a node project to be initiated in the repository.

<details>
<summary>Node project setup</summary>

Initialize a project with npm:

```bash
npm init -y
```

The following packages need to be installed:

```bash
npm install --save-dev @semantic-release/commit-analyzer @semantic-release/git @semantic-release/github @semantic-release/npm @semantic-release/release-notes-generator conventional-changelog-conventionalcommits
```

The following configuration needs to be added to a new file `.releaserc.json`:

```json
{
    "branches": [
        "main",
        {
            "name": "develop",
            "prerelease": "r"
        }
    ],
    "plugins": [
        [
            "@semantic-release/commit-analyzer",
            {
                "preset": "conventionalcommits"
            }
        ],
        [
            "@semantic-release/release-notes-generator",
            {
                "preset": "conventionalcommits"
            }
        ],
        [
            "@semantic-release/github",
            {
                "successComment": false
            }
        ],
        [
            "@semantic-release/npm",
            {
                "npmPublish": false
            }
        ],
        "@semantic-release/git"
    ]
}
```

*This file is the default configuration and can be changed to fit the needs of the project.*

A `.gitignore` file should be added to the repository to ignore the `node_modules` folder, created by NPM.

```gitignore
node_modules
```

</details>


#### Inputs

- `GH_TOKEN`: The GitHub token to use for authentication. Defaults to the default GitHub token.

#### Outputs

- `version`: The version that was released. This may be null if no release was created.

#### Example usage

**Usage of the action:**

```yaml
release:
    name: Release
    uses: the0mikkel/ci/.github/workflows/semver-release.yml@v1.0.0
```

### Docker build and push

This action will build a Docker image and push it to the GitHub Container Registry.  
Currently, it will push to the same image name as the repository name, but allows for specifying the tags.

It by defaults sets tags for latest, sha and version numbers.

#### Pre-requisites

The action requires a Dockerfile to be present in the repository.  
Where it is located can be specified in the action.

#### Inputs

- `dockerfile`: The path to the Dockerfile. Defaults to `./Dockerfile`.
- `context`: The build context for the Docker build. Defaults to `.`.
- `args`: Build arguments to pass to the Docker build. Defaults to an empty string.  
  Example: `VERSION=1.0.0`
- `semver`: The version to tag the image with. Leave empty, if you don't want to tag the image with the version. Defaults to an empty string.  
  Example: `1.0.0` or `${{ steps.release.outputs.version }}`
- `tags`: Additional tags to add to the image. Defaults to `type=raw,value=${{ github.sha }}`
- `registry`: What registry to use. Defaults to GitHub container registry `ghcr.io`
- `image_name`: Name of image. Defaults to GitHub repository `${{ github.repository }}`
- `registry_username`: Username to use for registry login. Defaults to Github actor.
- `registry_token`: Token to use for registry login. Defaults to GITHUB_TOKEN.

#### Example usage

**Usage of the action:**

```yaml
  docker:
    name: Docker
    needs: 
      - release
    uses: the0mikkel/ci/.github/workflows/docker.yml@v1.0.0
    with:
      semver: ${{ needs.release.outputs.version }}
```

### PR format test - Conventional commits

This action checks if the PR title fits the [Conventional commits](https://conventionalcommits.org) format.  
Perfect for ensuring the PR fits the Conventional commits format for [Semantic release](https://semantic-release.gitbook.io/semantic-release/).

#### Pre-requisites

This action requires no specific setup in the repository.

#### Inputs

There are not inputs for this action.

#### Example usage

**Usage of the action:**

```yaml
name: PR format test - Conventional commits

on:
  pull_request:
    types: [opened, synchronize, edited, reopened]

jobs:
  pr-linter:
    name: Pull Request title check
    uses: the0mikkel/ci/.github/workflows/pr-test-conventionalcommits.yml@v1.1.0
```
