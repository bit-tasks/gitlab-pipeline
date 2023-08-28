# Bit Tasks for Git CI/CD Pipelines

Integrate Bit tasks into your GitLab Pipelines.

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
For Git CI/CD pipelines, follow these instructions:

1. **Initialize Configuration File:** Create `.gitlab-ci.yml` in your GitLab repository's root.
2. **Workspace Navigation:** Move to the appropriate directory if your workspace isn't at the root of your Git repository. For instance, use `cd ws-dir`.
3. **Script Initialization:** Begin with `gitlab.bit.init`, as subsequent scripts will depend on it.
4. **CI/CD Variables Setup:** Define new CI/CD variables like:
   - `GITLAB_ACCESS_TOKEN`: [Project Access Token](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html)
   - `BIT_CONFIG_USER_TOKEN`: [Bit Config](https://bit.dev/reference/config/bit-config/)
   - `GIT_USER_NAME`
   - `GIT_USER_EMAIL`
   
Ensure these variables are correctly configured within your GitLab CI pipeline.

> **Tip:** By setting the variables in [GitLab project CI/CD variables](https://docs.gitlab.com/ee/ci/variables/), they'll automatically be accessible in the script without needing explicit definition in `.gitlab-ci.yml`.

## Task Scripts Usage

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

#### Parameters

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

#### Parameters

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

Execute this script when a Merge Request is created.

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

### 5. Bit Tag and Export: `gitlab.bit.tag-export`

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

### 6. Bit Branch and Lane: `gitlab.bit.branch-lane`

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

### Upcoming Features
- **Update Workspace Components, External Dependencies, and Environments**: Coming soon.

## Contributor Guide

To contribute, make updates to scripts starting with `gitlab.bit.` in the [Bit Docker Image Repository](https://github.com/bit-tasks/bit-docker-image).
