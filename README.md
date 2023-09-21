# Bit Tasks for Git CI/CD Pipelines

Example GitLab Pipeline jobs for common Bit and Git CI/CD workflows.

### GitLab Support with Bit Docker Image
You can leverage seamless integration of GitLab support through the [Bit Docker image](https://github.com/bit-tasks/bit-docker-image). Select from these available images:

- **Latest Stable:** 
  ```
  bitsrc/stable:latest
  ```
  
- **Nightly:** 
  ```bash
  bitsrc/nightly:latest
  ```

## Setup Guide

1. **Initialize Configuration File:** Create `.gitlab-ci.yml` in your GitLab repository's root and with the code shown below.
2. **Workspace Navigation:** Move to the appropriate directory if your workspace isn't at the root of your Git repository. For instance, use `cd ws-dir`.
3. **Script Initialization:** Begin with `gitlab.bit.init`, as subsequent scripts will depend on it.
4. **CI/CD Variables Setup:** Define new CI/CD variables like:
   - `GITLAB_TOKEN`: [Project Access Token](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html) or [Personal Access Token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)
   - `BIT_CONFIG_USER_TOKEN`: [Bit Config](https://bit.dev/reference/config/bit-config/)
   - `GIT_USER_NAME`
   - `GIT_USER_EMAIL`
   
Ensure these variables are correctly configured within your GitLab CI pipeline.

> **Tip:** By setting the variables in [GitLab project CI/CD variables](https://docs.gitlab.com/ee/ci/variables/), they'll automatically be accessible inside the script `.gitlab-ci.yml`.

### Automating Component Release

| Task                        | Example                         | 
|-----------------------------|---------------------------------|
| Initialize Bit             | [bit-init/.gitlab-ci.yml](/gitlab-pipelines/bit-init/.gitlab-ci.yml)          |
| Bit Verify Components  | [verify/.gitlab-ci.yml](/gitlab-pipelines/verify/.gitlab-ci.yml)                |
| Bit Tag and Export        | [tag-export/.gitlab-ci.yml](/gitlab-pipelines/tag-export/.gitlab-ci.yml)  |
| Bit Merge Request Build  | [merge-request/.gitlab-ci.yml](/gitlab-pipelines/merge-request/.gitlab-ci.yml) |
| Bit Lane Cleanup        | [lane-cleanup/.gitlab-ci.yml](/gitlab-pipelines/lane-cleanup/.gitlab-ci.yml) |
| Commit Bitmap           | [commit-bitmap/.gitlab-ci.yml](/gitlab-pipelines/commit-bitmap/.gitlab-ci.yml) |

  :arrow_down: [Download Files](https://github.com/bit-tasks/github-action-examples/raw/main/downloads/automating-component-releases.zip)

### Update Workspace Components, External Dependencies and Envs

| Task                        | Example                         |
|-----------------------------|---------------------------------|
| Dependency Update           | [dependency-update/.gitlab-ci.yml](/gitlab-pipelines/dependency-update/.gitlab-ci.yml)   |

  :arrow_down: [Download Files](https://github.com/bit-tasks/github-action-examples/raw/main/downloads/dependency-update.zip)

### Sync Git Branches with Bit Lanes

| Task                        | Example                         |
|-----------------------------|---------------------------------|
| Branch Lane                 | [branch-lane/.gitlab-ci.yml](/gitlab-pipelines/branch-lane/.gitlab-ci.yml)  |

  :arrow_down: [Download Files](https://github.com/bit-tasks/github-action-examples/raw/main/downloads/branch-lane.zip)

## Usage Documentation

### 1. Bit Initialization: `gitlab.bit.init`

```bash
gitlab.bit.init
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.init)

The Bit Docker image comes with the latest Bit version pre-installed. Still, the `gitlab.bit.init` script provides:

- Verification and possible installation of the relevant version as specified in the `engine` block of your `workspace.jsonc`.
- Initialization of default `org` and `scope` variables for further tasks.
- Execution of the `bit install` command inside the workspace.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

build-job:
  stage: build
  script: 
    - |
      cd my-workspace-dir
      gitlab.bit.init
  rules:
     - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```

### 2. Bit Verification: `gitlab.bit.verify`

```bash
gitlab.bit.verify
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.verify)

#### Parameters (Optional)

- `--skip-build`: This parameter, like `gitlab.bit.verify --skip-build`, prevents the `bit build` command execution.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

build-job:
  stage: build
  script: 
    - |
      cd my-workspace-dir
      gitlab.bit.init
      gitlab.bit.verify
  rules:
     - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```

### 3. Bit Commit Bitmap: `gitlab.bit.commit-bitmap`

```bash
gitlab.bit.commit-bitmap --skip-ci
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.commit-bitmap)

#### Parameters (Optional)

- `--skip-push`: Avoids pushing changes; useful for tests.
- `--skip-ci`: Prevents re-triggering CI on code push, avoiding potential loops.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

build-job:
  stage: build
  script: 
    - |
      cd my-workspace-dir
      gitlab.bit.init
      gitlab.bit.commit-bitmap --skip-ci
  rules:
     - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```

### 4. Bit Merge Request: `gitlab.bit.merge-request`

```bash
gitlab.bit.merge-request
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.merge-request)

Execute this script when a Merge Request is created. It verifies the components and create a lane in [bit.cloud](https://bit.cloud) for previewing and testing components.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

merge-request-job:
  stage: build
  script: 
    - |
      cd my-workspace-dir
      gitlab.bit.init
      gitlab.bit.merge-request
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

### 5. Bit Lane Cleanup: `gitlab.bit.lane-cleanup`

```bash
gitlab.bit.lane-cleanup
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.lane-cleanup)

Execute this script when a Merge Request is approved (GitLab doesn't have a Merge Request merged or closed event). You need to update $CI_MERGE_REQUEST_APPROVALS to match with your configuration.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

merge-request-closed-job:
  stage: build
  script: 
    - |
      cd my-workspace-dir
      gitlab.bit.init
      gitlab.bit.lane-cleanup
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_APPROVALS == "1" && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
```

### 6. Bit Tag and Export: `gitlab.bit.tag-export`

```bash
gitlab.bit.tag-export
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.tag-export)

Tag component versions using labels on Merge Requests or within Merge Request/Commit titles. Use version keywords `major`, `minor`, `patch`, and `pre-release`.

> **Note:** If a Merge Request is merged, track it via its `merge commit` in the target branch. For the action to detect the version keyword, the `merge commit` should be the recent one in the commit history.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

release-components-job:
  stage: build
  script: 
    - |
      cd my-workspace-dir
      gitlab.bit.init
      gitlab.bit.tag-export
  rules:
    - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```

### 7. Bit Branch and Lane: `gitlab.bit.branch-lane`

```bash
gitlab.bit.branch-lane
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.branch-lane)

Execute this script when a new branch is created in Git. It will create a lane in [bit.cloud](https://bit.cloud) for each new Branch and keep the lane in sync with the components modified in Git.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

build-job:
  stage: build
  script: 
    - |
      cd test-ws
      gitlab.bit.init
      gitlab.bit.branch-lane
  rules:
    - if: '$CI_PIPELINE_SOURCE == "push"'
```

### 8. Bit Dependency Update: `gitlab.bit.dependency-update`

```bash
gitlab.bit.dependency-update --allow "envs, workspace-components"
```
*Source:* [script details](https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.dependency-update)

Run this script as a [scheduled pipeline](https://docs.gitlab.com/ee/ci/pipelines/schedules.html), which will create a merge request to the specified branch with the updated dependencies.

#### Parameters (Optional)

- `--allow`: Allow different types of dependencies. Options `all`, `external-dependencies`, `workspace-components`, `envs`. You can also use a combination of one or two values, e.g. `gitlab.bit.dependency-update --allow "external-dependencies, workspace-components"`. Default `all`.
- `--branch`: Branch to check for dependency updates. Default `main`.

#### Example
```yaml
image: bitsrc/stable:latest

variables:
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git_user_email”

check-updates:
  stage: build
  script: 
    - |
      cd test-ws
      gitlab.bit.init
      gitlab.bit.dependency-update --allow "all" --branch "main"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

## Contributor Guide

To contribute, make updates to scripts starting with `gitlab.bit.` in the [Bit Docker Image Repository](https://github.com/bit-tasks/bit-docker-image).

To create zip files use the below commands.

```bash
chmod +x zip_files.sh
bash ./zip_files.sh
```
