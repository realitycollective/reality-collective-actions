# Reality Collective Actions

A collection of reusable composite GitHub Actions for Unity Package Manager (UPM) projects, built and maintained by the Reality Collective.

[![Test Composite Actions](https://github.com/realitycollective/reality-collective-actions/actions/workflows/test-actions.yml/badge.svg)](https://github.com/realitycollective/reality-collective-actions/actions/workflows/test-actions.yml)

## Overview

This repository contains a suite of composite GitHub Actions designed to streamline CI/CD workflows for Unity UPM packages. These actions automate version extraction, Unity environment validation, multi-target builds, and release management.

> [!NOTE]
> We recommend using [Buildalon](https://github.com/buildalon) Actions to manage Unity Hub/Editor setup and activation, as well as useful tools like "create-unity-project" to setup a temporary Unity project for testing.
>
> Although we have developed custom actions for actually setting up UPM builds and running them on various versions of Unity/OS concurrently, as Buildalon actions cannot do this out of the box.

### Key Features

- **UPM Setup**: Configure Unity projects with UPM packages and dependencies.
- **Multi-Target Building**: Build Unity projects against multiple build targets on various Operating systems.
- **Version Extraction**: Read package and Unity versions from UPM packages and Unity projects.
- **Release Management**: Automate version incrementing and git tag creation.
- **Branch Synchronization**: Keep branches in sync across repositories.

---

## Table of Contents

- [Actions](#actions)
  - [Setup Unity UPM Build](#setup-unity-upm-build)
  - [Run Unity Multi-Target Build](#run-unity-multi-target-build)
  - [Get Package Version From Package](#get-package-version-from-package)
  - [Get Unity Version From Package](#get-unity-version-from-package)
  - [Get Unity Version From Project](#get-unity-version-from-project)
  - [Tag Release](#tag-release)
  - [Upversion And Tag Release](#upversion-and-tag-release)
  - [Refresh Branch](#refresh-branch)
- [Usage Examples](#usage-examples)
- [Contributing](#contributing)
- [License](#license)

---

## Actions

### Setup Unity UPM Build

**Action:** `setup-unity-upm-build`

Prepares a Unity project by checking out a UPM package and its dependencies.

#### Description

This action orchestrates the setup of a Unity project for UPM package validation. It checks out the UPM package being tested into the Packages directory and optionally clones additional dependencies from other repositories. This is typically used as a preparation step before running Unity builds or tests.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `unity-project-path` | Path to the test Unity project (must already exist) | Yes | N/A |
| `dependencies` | JSON array of dependency objects: `[{"DependencyBranch": "domain/repo.git"}]` | No | N/A |
| `repository` | (Optional) owner/repo of the UPM package to validate (defaults to current repository) | No | N/A |

#### Outputs

This action does not produce outputs.

#### Environment Variables

- `GITHUB_TOKEN`: Required for cloning private dependency repositories
- `GITHUB_ACTOR`: Used for git authentication with dependencies

#### Example Usage

```yaml
- name: Setup Unity Project
  uses: realitycollective/reality-collective-actions/setup-unity-upm-build@v1
  with:
    unity-project-path: './TestProject'
    dependencies: |
      [
        {"utilities": "github.com/realitycollective/com.realitycollective.utilities.git"}
      ]
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### Run Unity Multi-Target Build

**Action:** `run-unity-multitarget-build`

Builds a Unity project against multiple build targets in sequence.

#### Description

This action accepts a comma-separated list of Unity build targets and runs a Unity build for each target specified. It validates the Unity installation, executes builds in batch mode with appropriate platform-specific handling, and uploads build logs as artifacts. The action will fail fast if any target build fails.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `unity-editor-path` | The path to the Unity editor executable | Yes | N/A |
| `unity-project-path` | The path to the Unity project | Yes | N/A |
| `build-targets` | Comma-separated list of build targets (e.g., "StandaloneWindows64,StandaloneLinux64,Android") | Yes | N/A |

#### Outputs

This action does not produce outputs, but uploads build logs as artifacts.

#### Artifacts

- `unity-build-log-{run-number}-{run-attempt}-{os}-{unity-version}-{targets}`: Contains all Unity build logs for debugging

#### Example Usage

```yaml
- name: Run Multi-Target Build
  uses: realitycollective/reality-collective-actions/run-unity-multitarget-build@v1
  with:
    unity-editor-path: 'C:\Program Files\Unity\Hub\Editor\2021.3.15f1\Editor\Unity.exe'
    unity-project-path: './TestProject'
    build-targets: 'StandaloneWindows64,StandaloneLinux64,Android'
```

---

### Get Package Version From Package

**Action:** `get-package-version-from-package`

Extracts the version number from a Unity Package Manager (UPM) `package.json` file.

#### Description

This action reads a UPM package.json file and extracts the package version. It's useful for determining the current version of a package before build processes, version incrementing, or tagging releases.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version-file-path` | Path to the UPM package.json file | No | `package.json` |

#### Outputs

| Name | Description |
|------|-------------|
| `packageversion` | The version string extracted from the package.json file (e.g., "1.0.0", "2.1.3-pre.1") |

#### Example Usage

```yaml
- name: Get Package Version
  id: package-version
  uses: realitycollective/reality-collective-actions/setup-unity-upm-buildget-package-version-from-package@v1
  with:
    version-file-path: 'package.json'

- name: Display Package Version
  run: echo "Current package version is ${{ steps.package-version.outputs.packageversion }}"
```

---

### Get Unity Version From Package

**Action:** `get-unity-version-from-package`

Retrieves the Unity Editor version requirement from a UPM `package.json` file.

#### Description

This action reads the Unity version specified in a UPM package.json file. The Unity version is stored in the "unity" field and indicates the minimum Unity version required for the package to function.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version-file-path` | Path to the UPM package.json file | No | `package.json` |

#### Outputs

| Name | Description |
|------|-------------|
| `unityversion` | The Unity version string from the package.json (e.g., "2021.3", "2022.2") |

#### Example Usage

```yaml
- name: Get Unity Version from Package
  id: unity-version
  uses: realitycollective/reality-collective-actions/get-unity-version-from-package@v1
  with:
    version-file-path: 'package.json'

- name: Display Unity Version
  run: echo "Required Unity version is ${{ steps.unity-version.outputs.unityversion }}"
```

---

### Get Unity Version From Project

**Action:** `get-unity-version-from-project`

Extracts the Unity Editor version from a Unity project's `ProjectVersion.txt` file.

#### Description

This action reads a Unity project's ProjectSettings/ProjectVersion.txt file to determine which version of Unity Editor was used to create or last open the project. This is useful when working with Unity projects (as opposed to standalone UPM packages).

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version-file-path` | Path to the Unity project's ProjectVersion.txt file | No | `**/ProjectSettings/ProjectVersion.txt` |

#### Outputs

| Name | Description |
|------|-------------|
| `unityversion` | The Unity version string extracted from ProjectVersion.txt (e.g., "2021.3.15f1") |

#### Example Usage

```yaml
- name: Get Unity Version from Project
  id: project-version
  uses: realitycollective/reality-collective-actions/get-unity-version-from-project@v1
  with:
    version-file-path: 'ProjectSettings/ProjectVersion.txt'

- name: Use Unity Version
  run: echo "Project uses Unity ${{ steps.project-version.outputs.unityversion }}"
```

---

### Tag Release

**Action:** `tag-release`

Creates a git tag for a specific version in the repository.

#### Description

This action creates and pushes a git tag to mark a release. It validates that the tag doesn't already exist and creates the tag with a "v" prefix (e.g., "v1.0.0"). The commit message includes "[skip ci]" to prevent triggering additional workflow runs.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `version` | The version number to tag (without the "v" prefix, e.g., "1.0.0") | Yes | N/A |

#### Outputs

This action does not produce outputs.

#### Environment Variables

- `GIT_PAT`: A Personal Access Token with `repo` scope required to push tags

#### Example Usage

```yaml
- name: Create Release Tag
  uses: realitycollective/reality-collective-actions/tag-release@v1
  with:
    version: '1.0.0'
  env:
    GIT_PAT: ${{ secrets.GIT_PAT }}
```

#### Example with Version from Package

```yaml
- name: Get Package Version
  id: version
  uses: realitycollective/reality-collective-actions/setup-unity-upm-buildget-package-version-from-package@v1

- name: Tag Release
  uses: realitycollective/reality-collective-actions/tag-release@v1
  with:
    version: ${{ steps.version.outputs.packageversion }}
  env:
    GIT_PAT: ${{ secrets.GIT_PAT }}
```

---

### Upversion And Tag Release

**Action:** `upversion-and-tag-release`

Automatically increments the package version and optionally creates a release tag.

#### Description

This action reads the current version from package.json, increments it according to the specified build type (major, minor, patch, pre-release, etc.), commits the change, and optionally creates a git tag. It follows semantic versioning principles and includes "[skip ci]" in commit messages.

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

- `GIT_PAT`: A Personal Access Token with `repo` scope required to push commits and tags

#### Example Usage

```yaml
- name: Increment Pre-Release Version
  id: upversion
  uses: realitycollective/reality-collective-actions/upversion-and-tag-release@v1
  with:
    build-type: 'pre-release'
    createTag: true
  env:
    GIT_PAT: ${{ secrets.GIT_PAT }}

- name: Display New Version
  run: echo "New version is ${{ steps.upversion.outputs.packageversion }}"
```

#### Example with Different Build Types

```yaml
# Release a new major version
- name: Major Version Release
  uses: realitycollective/reality-collective-actions/upversion-and-tag-release@v1
  with:
    build-type: 'major'
    target-branch: 'main'
    createTag: true
  env:
    GIT_PAT: ${{ secrets.GIT_PAT }}

# Create a patch release
- name: Patch Release
  uses: realitycollective/reality-collective-actions/upversion-and-tag-release@v1
  with:
    build-type: 'patch'
    createTag: true
  env:
    GIT_PAT: ${{ secrets.GIT_PAT }}
```

---

### Refresh Branch

**Action:** `refresh-branch`

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
  uses: realitycollective/reality-collective-actions/refresh-branch@v1
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
        uses: actions/checkout@v5
        with:
          fetch-depth: 0
          
      - name: Sync Feature Branch
        uses: realitycollective/reality-collective-actions/refresh-branch@v1
        with:
          target-branch: 'feature/experimental'
          source-branch: 'development'
        env:
          GIT_PAT: ${{ secrets.GIT_PAT }}
```

---

## Usage Examples

### Complete Build Pipeline Example

Here's a complete example workflow that creates a Unity project to test a UPM package with several Unity versions on different OS's:

```yaml
name: Build UPM Package

on:
  push:
    branches: [ main, development ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  setup-and-build:
    name: Test Unity UPM Build
    runs-on: ${{ matrix.os }}
    if: always()
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        unity-version:
          - 6000.0.x
          - 6000
        include:
          - os: ubuntu-latest
            build-targets: StandaloneLinux64, Android, iOS
            build-args: ""
          - os: windows-latest
            build-targets: StandaloneWindows64, Android
            build-args: ""
          - os: macos-latest
            build-targets: StandaloneOSX, Android, iOS, VisionOS
            build-args: ""
    
    steps:
      - uses: buildalon/unity-setup@v2
        id: unity-setup
        with:
          version-file: "None"
          unity-version: ${{ matrix.unity-version }} # overrides version in version-file
          build-targets: ${{ matrix.build-targets }}

      - uses: buildalon/activate-unity-license@v2
        if: runner.environment == 'github-hosted'
        with:
          license: "Personal" # Choose license type to use [ Personal, Professional, Floating ]
          username: ${{ secrets.UNITY_USER}}
          password: ${{ secrets.UNITY_ACC}}

      - uses: buildalon/create-unity-project@v2
        id: create-unity-project
        with:
          project-name: P

      - name: Setup Unity UPM Build
        uses: realitycollective/reality-collective-actions/setup-unity-upm-build@v1
        with:
          unity-project-path: ${{ steps.create-unity-project.outputs.project-path }}
          dependencies: '[{"development": "github.com/realitycollective/com.realitycollective.buildtools.git"},{"development": "github.com/realitycollective/com.realitytoolkit.core.git"},{"development": "github.com/realitycollective/com.realitycollective.utilities.git"},{"development": "github.com/realitycollective/com.realitycollective.service-framework.git"}]'

      - name: Run Unity Build
        uses: realitycollective/reality-collective-actions/run-unity-multitarget-build@v1
        with:
          unity-editor-path: ${{ steps.unity-setup.outputs.unity-editor-path }}
          unity-project-path: ${{ steps.create-unity-project.outputs.project-path }}
          build-targets: ${{ matrix.build-targets }}
```

### Version Management Example

```yaml
name: Release Management

on:
  workflow_dispatch:
    inputs:
      release-type:
        description: 'Release type'
        required: true
        type: choice
        options:
          - pre-release
          - patch
          - minor
          - major

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout with Full History
        uses: actions/checkout@v5
        with:
          fetch-depth: 0
          token: ${{ secrets.GIT_PAT }}
      
      - name: Increment Version and Tag
        id: version
        uses: realitycollective/reality-collective-actions/upversion-and-tag-release@v1
        with:
          build-type: ${{ github.event.inputs.release-type }}
          createTag: true
        env:
          GIT_PAT: ${{ secrets.GIT_PAT }}
      
      - name: Display New Version
        run: echo "Released version ${{ steps.version.outputs.packageversion }}"
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
6. Include script versioning in each action

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Support

For issues, questions, or contributions, please visit the [GitHub repository](https://github.com/realitycollective/reality-collective-actions) or join the [Reality Collective Discord](https://discord.gg/hF7TtRCFmB).
