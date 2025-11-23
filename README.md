# Reality Collective Actions

A collection of reusable composite GitHub Actions for Unity Package Manager (UPM) projects, built and maintained by the Reality Collective.

[![Test Composite Actions](https://github.com/realitycollective/reality-collective-actions/actions/workflows/test-actions.yml/badge.svg)](https://github.com/realitycollective/reality-collective-actions/actions/workflows/test-actions.yml)

## Overview

This repository contains a comprehensive suite of composite GitHub Actions designed to streamline the CI/CD pipeline for Unity UPM packages. These actions automate common tasks such as version management, Unity build validation, package testing, and release management.

### Key Features

- **Version Management**: Extract and manage package and Unity versions from UPM packages
- **Unity Installation Validation**: Verify Unity Editor installations across platforms
- **Automated Building**: Build and test UPM packages with single or multiple Unity versions
- **Release Automation**: Automatically increment versions and create git tags
- **Branch Management**: Keep branches synchronized across repositories

---

## Table of Contents

- [Actions](#actions)
  - [Get Package Version From Package](#get-package-version-from-package)
  - [Get Unity Version From Package](#get-unity-version-from-package)
  - [Get Unity Version From Project](#get-unity-version-from-project)
  - [Validate Unity Install](#validate-unity-install)
  - [Run Unity UPM Build](#run-unity-upm-build)
  - [Run Unity UPM Build Multi-Version](#run-unity-upm-build-multi-version)
  - [Tag Release](#tag-release)
  - [Upversion And Tag Release](#upversion-and-tag-release)
  - [Refresh Branch](#refresh-branch)
- [Usage Examples](#usage-examples)
- [Contributing](#contributing)
- [License](#license)

---

## Actions

### Get Package Version From Package

**Action:** `getpackageversionfrompackage`

Extracts the version number from a Unity Package Manager (UPM) `package.json` file.

#### Description

This action reads a UPM package.json file and extracts the package version. It's useful for determining the current version of a package before build processes, version incrementing, or tagging releases.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version-file-path` | Path to the UPM package.json file. If not specified, searches for package.json in the repository root. | No | `package.json` |

#### Outputs

| Name | Description |
|------|-------------|
| `packageversion` | The version string extracted from the package.json file (e.g., "1.0.0", "2.1.3-pre.1") |

#### Example Usage

```yaml
- name: Get Package Version
  id: package-version
  uses: realitycollective/reality-collective-actions/getpackageversionfrompackage@main
  with:
    version-file-path: 'package.json'

- name: Display Package Version
  run: echo "Current package version is ${{ steps.package-version.outputs.packageversion }}"
```

---

### Get Unity Version From Package

**Action:** `getunityversionfrompackage`

Retrieves the Unity Editor version requirement from a UPM `package.json` file.

#### Description

This action reads the Unity version specified in a UPM package.json file. The Unity version is typically stored in the "unity" field and indicates the minimum Unity version required for the package to function.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version-file-path` | Path to the UPM package.json file. Must contain a "unity" field with the Unity version (e.g., "2021.3"). | No | `package.json` |

#### Outputs

| Name | Description |
|------|-------------|
| `unityversion` | The Unity version string from the package.json (e.g., "2021.3", "2022.2") |

#### Example Usage

```yaml
- name: Get Unity Version from Package
  id: unity-version
  uses: realitycollective/reality-collective-actions/getunityversionfrompackage@main
  with:
    version-file-path: 'package.json'

- name: Display Unity Version
  run: echo "Required Unity version is ${{ steps.unity-version.outputs.unityversion }}"
```

---

### Get Unity Version From Project

**Action:** `getunityversionfromproject`

Extracts the Unity Editor version from a Unity project's `ProjectVersion.txt` file.

#### Description

This action reads a Unity project's ProjectSettings/ProjectVersion.txt file to determine which version of Unity Editor was used to create or last open the project. This is useful when working with Unity projects (as opposed to standalone UPM packages).

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version-file-path` | Path to the Unity project's ProjectVersion.txt file. | No | `**/ProjectSettings/ProjectVersion.txt` |

#### Outputs

| Name | Description |
|------|-------------|
| `unityversion` | The Unity version string extracted from ProjectVersion.txt (e.g., "2021.3.15f1") |

#### Example Usage

```yaml
- name: Get Unity Version from Project
  id: project-version
  uses: realitycollective/reality-collective-actions/getunityversionfromproject@main
  with:
    version-file-path: 'ProjectSettings/ProjectVersion.txt'

- name: Use Unity Version
  run: echo "Project uses Unity ${{ steps.project-version.outputs.unityversion }}"
```

---

### Validate Unity Install

**Action:** `validateunityinstall`

Validates that a specific Unity Editor version is installed on the build agent.

#### Description

This action checks whether Unity Hub and a specific Unity Editor version are installed on the build machine. It can install Unity Hub if not present and will search for the closest matching Unity version if the exact version is not found. Works across Windows, macOS, and Linux platforms.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `unityversion` | The Unity version to validate (e.g., "2021.3", "2022.2.5f1"). Can be a short version (major.minor) or full version. | Yes | N/A |

#### Outputs

| Name | Description |
|------|-------------|
| `unityeditorversion` | The full Unity Editor version that was found and validated |
| `unityeditorinstalled` | Returns `1` if the Unity Editor is installed, `0` otherwise |

#### Example Usage

```yaml
- name: Validate Unity Installation
  id: validate-unity
  uses: realitycollective/reality-collective-actions/validateunityinstall@main
  with:
    unityversion: '2021.3'

- name: Check Installation Result
  run: |
    echo "Unity version found: ${{ steps.validate-unity.outputs.unityeditorversion }}"
    echo "Installation status: ${{ steps.validate-unity.outputs.unityeditorinstalled }}"
```

---

### Run Unity UPM Build

**Action:** `rununityUPMbuild`

Builds and tests a Unity UPM package with a single specified Unity version.

#### Description

This comprehensive action creates a temporary Unity project, installs the UPM package, resolves dependencies, and runs Unity in batch mode to validate the package. It supports all major platforms (Windows, macOS, Linux) and can handle package dependencies from other repositories.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `unityVersion` | The Unity version to use for building (e.g., "2021.3", "2022.2.5f1") | Yes | N/A |
| `dependencies` | JSON array of dependencies with their git URLs. Format: `[{"dependency-name": "github.com/owner/repo.git"}]` | No | N/A |

#### Outputs

This action does not produce outputs, but uploads build logs as artifacts.

#### Environment Variables

- `GITHUB_TOKEN`: Required for cloning private dependency repositories
- `GITHUB_ACTOR`: Used for git authentication with dependencies

#### Artifacts

- `unity-build-log-{build-target}`: Contains all Unity build logs for debugging

#### Example Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        build-target: [StandaloneLinux64, StandaloneWindows64]
    steps:
      - uses: actions/checkout@v4
      
      - name: Build UPM Package
        uses: realitycollective/reality-collective-actions/rununityUPMbuild@main
        with:
          unityVersion: '2021.3'
          dependencies: |
            [
              {"utilities": "github.com/realitycollective/com.realitycollective.utilities.git"}
            ]
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### Advanced Usage with Dependencies

```yaml
- name: Build with Multiple Dependencies
  uses: realitycollective/reality-collective-actions/rununityUPMbuild@main
  with:
    unityVersion: '2022.3'
    dependencies: |
      [
        {"utilities": "github.com/realitycollective/com.realitycollective.utilities.git"},
        {"service-framework": "github.com/realitycollective/com.realitycollective.service-framework.git"}
      ]
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### Run Unity UPM Build Multi-Version

**Action:** `rununityUPMbuildmultiversion`

Builds and tests a Unity UPM package across multiple Unity versions using a build matrix.

#### Description

This action is similar to `rununityUPMbuild` but is designed to work with GitHub Actions matrix strategies. It allows you to test your package against multiple Unity versions simultaneously. The action automatically handles version resolution and can filter by minimum Unity version.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `dependencies` | JSON array of dependencies with their git URLs. Format: `[{"dependency-name": "github.com/owner/repo.git"}]` | No | N/A |
| `min-version` | Minimum Unity version to build against (e.g., "2021.3"). Versions below this will be skipped. | No | N/A |

#### Outputs

This action does not produce outputs, but uploads build logs as artifacts.

#### Environment Variables

- `GITHUB_TOKEN`: Required for cloning private dependency repositories
- `GITHUB_ACTOR`: Used for git authentication with dependencies

#### Matrix Variables

This action expects the following matrix variables:
- `matrix.unityVersion`: The Unity version to test with
- `matrix.build-target`: The Unity build target (e.g., StandaloneLinux64)

#### Artifacts

- `unity-build-log-{unityVersion}-{build-target}`: Contains Unity build logs for each version/target combination

#### Example Usage

```yaml
jobs:
  build-multiversion:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        unityVersion: ['2021.3', '2022.3', '2023.2']
        build-target: [StandaloneLinux64]
    steps:
      - uses: actions/checkout@v4
      
      - name: Build UPM Package Multi-Version
        uses: realitycollective/reality-collective-actions/rununityUPMbuildmultiversion@main
        with:
          min-version: '2021.3'
          dependencies: |
            [
              {"utilities": "github.com/realitycollective/com.realitycollective.utilities.git"}
            ]
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### Tag Release

**Action:** `tagrelease`

Creates a git tag for a specific version in the repository.

#### Description

This action creates and pushes a git tag to mark a release. It includes validation to prevent duplicate tags and can force-update existing tags. The tag is created with a "v" prefix (e.g., "v1.0.0").

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version` | The version number to tag (without the "v" prefix). Example: "1.0.0" | Yes | N/A |

#### Outputs

This action does not produce outputs.

#### Environment Variables

- `GITHUB_TOKEN`: Required with sufficient permissions to push tags (use a PAT with `repo` scope if needed)

#### Example Usage

```yaml
- name: Create Release Tag
  uses: realitycollective/reality-collective-actions/tagrelease@main
  with:
    version: '1.0.0'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### Example with Version from Package

```yaml
- name: Get Package Version
  id: version
  uses: realitycollective/reality-collective-actions/getpackageversionfrompackage@main

- name: Tag Release
  uses: realitycollective/reality-collective-actions/tagrelease@main
  with:
    version: ${{ steps.version.outputs.packageversion }}
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### Upversion And Tag Release

**Action:** `upversionandtagrelease`

Automatically increments the package version and optionally creates a release tag.

#### Description

This action reads the current version from package.json, increments it according to the specified build type (major, minor, patch, pre-release, etc.), commits the change, and optionally creates a git tag. It follows semantic versioning principles.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `build-type` | Type of version increment: `major`, `minor`, `patch`, `patch-release`, `pre-release`, or `build` | No | `pre-release` |
| `target-branch` | The branch to apply the version update to | No | `${{ github.ref }}` |
| `createTag` | Whether to create a git tag after updating the version (true/false) | No | `false` |

#### Outputs

| Name | Description |
|------|-------------|
| `packageversion` | The new version string after incrementing (e.g., "1.1.0", "2.0.0-pre.5") |

#### Version Increment Types

- **major**: Increments major version (1.0.0 → 2.0.0)
- **minor**: Increments minor version (1.0.0 → 1.1.0)
- **patch**: Increments patch version (1.0.0 → 1.0.1)
- **patch-release**: Resets to current patch without increment (1.0.0-pre.5 → 1.0.0)
- **pre-release**: Increments pre-release version (1.0.0-pre.1 → 1.0.0-pre.2)
- **build**: Increments build metadata (1.0.0-pre.1+1 → 1.0.0-pre.1+2)

#### Environment Variables

- `GITHUB_TOKEN`: Required with sufficient permissions to push commits and tags

#### Example Usage

```yaml
- name: Increment Pre-Release Version
  id: upversion
  uses: realitycollective/reality-collective-actions/upversionandtagrelease@main
  with:
    build-type: 'pre-release'
    createTag: true
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- name: Display New Version
  run: echo "New version is ${{ steps.upversion.outputs.packageversion }}"
```

#### Example with Different Build Types

```yaml
# Release a new major version
- name: Major Version Release
  uses: realitycollective/reality-collective-actions/upversionandtagrelease@main
  with:
    build-type: 'major'
    target-branch: 'main'
    createTag: true
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

# Create a patch release
- name: Patch Release
  uses: realitycollective/reality-collective-actions/upversionandtagrelease@main
  with:
    build-type: 'patch'
    createTag: true
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### Refresh Branch

**Action:** `refreshbranch`

Synchronizes a target branch with changes from a source branch.

#### Description

This action pulls changes from a source branch and pushes them to a target branch, helping keep feature branches up-to-date with the main branch or synchronizing branches across your repository. The commit message includes "[skip ci]" to prevent triggering additional workflow runs.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `target-branch` | The branch to update with changes from the source branch | Yes | N/A |
| `source-branch` | The branch to pull changes from | Yes | N/A |

#### Outputs

This action does not produce outputs.

#### Environment Variables

- `GIT_PAT`: A Personal Access Token with `repo` scope to push to the target branch

#### Example Usage

```yaml
- name: Refresh Development Branch
  uses: realitycollective/reality-collective-actions/refreshbranch@main
  with:
    target-branch: 'development'
    source-branch: 'main'
  env:
    GIT_PAT: ${{ secrets.GIT_PAT }}
```

#### Scheduled Branch Sync

```yaml
name: Sync Branches

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday at midnight
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Sync Feature Branch
        uses: realitycollective/reality-collective-actions/refreshbranch@main
        with:
          target-branch: 'feature/experimental'
          source-branch: 'development'
        env:
          GIT_PAT: ${{ secrets.GIT_PAT }}
```

---

## Usage Examples

### Complete CI/CD Pipeline Example

Here's a complete example workflow that uses multiple actions together:

```yaml
name: Build and Release

on:
  push:
    branches: [ main, development ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  validate-and-build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        unityVersion: ['2021.3', '2022.3']
        build-target: [StandaloneLinux64, StandaloneWindows64, StandaloneOSX]
        exclude:
          - os: ubuntu-latest
            build-target: StandaloneWindows64
          - os: ubuntu-latest
            build-target: StandaloneOSX
          - os: windows-latest
            build-target: StandaloneLinux64
          - os: windows-latest
            build-target: StandaloneOSX
          - os: macos-latest
            build-target: StandaloneWindows64
          - os: macos-latest
            build-target: StandaloneLinux64
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Get Package Version
        id: package-version
        uses: realitycollective/reality-collective-actions/getpackageversionfrompackage@main
      
      - name: Get Unity Version
        id: unity-version
        uses: realitycollective/reality-collective-actions/getunityversionfrompackage@main
      
      - name: Validate Unity Install
        uses: realitycollective/reality-collective-actions/validateunityinstall@main
        with:
          unityversion: ${{ steps.unity-version.outputs.unityversion }}
      
      - name: Build UPM Package
        uses: realitycollective/reality-collective-actions/rununityUPMbuild@main
        with:
          unityVersion: ${{ matrix.unityVersion }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  release:
    needs: validate-and-build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GIT_PAT }}
      
      - name: Increment Version
        id: version
        uses: realitycollective/reality-collective-actions/upversionandtagrelease@main
        with:
          build-type: 'patch'
          createTag: true
        env:
          GITHUB_TOKEN: ${{ secrets.GIT_PAT }}
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ steps.version.outputs.packageversion }}
          release_name: Release ${{ steps.version.outputs.packageversion }}
          draft: false
          prerelease: false
```

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

### Development Guidelines

1. Ensure all actions use PowerShell Core for cross-platform compatibility
2. Add comprehensive error handling and validation
3. Update documentation for any input/output changes
4. Test on all platforms (Windows, macOS, Linux) when applicable
5. Follow existing code style and conventions

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Support

For issues, questions, or contributions, please visit the [GitHub repository](https://github.com/realitycollective/reality-collective-actions) or join the [Reality Collective Discord](https://discord.gg/hF7TtRCFmB).
