# Bit Tasks for Git CI/CD Pipelines

This document provides guidance on setting up GitLab Pipelines for Bit and Git CI/CD workflows, complete with examples.

GitLab support is seamlessly integrated into the [Bit Docker image](https://github.com/bit-tasks/bit-docker-image). You can opt for one of the images below:

- **Latest Stable:** 
  ```bash
  docker pull bitsrc/stable:latest
  ```
  
- **Nightly:** 
  ```bash
  docker pull bitsrc/nightly:latest
  ```

## **Setup Guide**

Follow the steps below to set up your Git CI/CD pipelines:

1. **Initialize Configuration File:** Create a `.gitlab-ci.yml` file in the root of your GitLab repository.
2. **Navigate to Workspace:** If your workspace isn't located at the root of your Git repository, navigate to the correct directory using, for example, `cd ws-dir`.
3. **Initialize Script:** Start with the `gitlab.bit.init` script. Subsequent scripts will depend on it.
4. **Set CI/CD Variables:** Establish new CI/CD variables such as:
   - `GITLAB_ACCESS_TOKEN`: [Project Access Token](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html)
   - `BIT_CONFIG_USER_TOKEN`: [Bit Config](https://bit.dev/reference/config/bit-config/)
   - `GIT_USER_NAME`
   - `GIT_USER_EMAIL`

   Ensure these variables are correctly referenced within your GitLab CI pipeline.

**Note:** If you set the variables in [GitLab project CI/CD variables](https://docs.gitlab.com/ee/ci/variables/), they become available in the script and not required to define in the `.gitlab-ci.yml`.

## Using Bit Init

```
gitlab.bit.init         # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.init
```

The latest bit version is pre-installed in the docker image. Yet the `gitlab.bit.init` script performs several steps useful for other scripts in your CI/CD pipeline Job.

1. Check `engine` block in your `workspace.jsonc` and if defined install the relevant version.
```
"teambit.harmony/bit": {
  "engine": "0.2.3",
  "engineStrict": true // warning or error if the version of the engine is not the same as the workspace
}
```
2. Initialize default `org` and `scope` variables for subsequent tasks.
3. Run `bit install` command inside the workspace.
   
### Example usage
```
image: bitsrc/stable:latest

variables:
  # Not required to define BIT_CONFIG_USER_TOKEN or GITLAB_ACCESS_TOKEN if added to GitLab project CI/CD variables
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git”_user_email

build-job:
  stage: build
  script: 
    - |      
      cd my-workspace-dir     # workspace is in this directory
      gitlab.bit.init         # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.init
  rules:
     - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```
## Using Bit Verify

### Example usage

```
gitlab.bit.verify         # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.verify
```

**Parameters**
1. `--skip-build` - You can use this parameter (e.g `gitlab.bit.verify --skip-build`) to avoid running `bit build` command.

```
image: bitsrc/stable:latest

variables:
  # Not required to define BIT_CONFIG_USER_TOKEN or GITLAB_ACCESS_TOKEN if added to GitLab project CI/CD variables
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git”_user_email

build-job:
  stage: build
  script: 
    - |      
      cd my-workspace-dir     # workspace is in this directory
      gitlab.bit.init         # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.init
      gitlab.bit.verify       # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.verify
  rules:
     - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```

## Using Bit Commit Bitmap

```
gitlab.bit.commit-back --skip-ci        # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.commit-back
```

**Parameters**
1. `--skip-push` - You can use this parameter (e.g `gitlab.bit.verify --skip-push`) to avoid pushing changes. This is mostly used for testing purposes.
2. `--skip-ci` - You can use this parameter (e.g `gitlab.bit.verify --skip-ci`) to avoid triggering the CI again upon code push that ends up in a loop. This is mostly used for testing purposes.


### Example usage

```
image: bitsrc/stable:latest

variables:
  # Not required to define BIT_CONFIG_USER_TOKEN or GITLAB_ACCESS_TOKEN if added to GitLab project CI/CD variables
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git”_user_email

build-job:
  stage: build
  script: 
    - |      
      cd my-workspace-dir                   # workspace is in this directory
      gitlab.bit.init                       # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.init
      gitlab.bit.commit-bitmap --skip-ci    # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.commit-bitmap
  rules:
     - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```

## Using Bit Merge Request

```
gitlab.bit.merge-request        # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.merge-request
```

Run this script, upon Merge Request is created.

### Example usage
```
image: bitsrc/stable:latest

variables:
  # Not required to define BIT_CONFIG_USER_TOKEN or GITLAB_ACCESS_TOKEN if added to GitLab project CI/CD variables
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git”_user_email

merge-request-job:
  stage: build
  script: 
    - |      
      cd my-workspace-dir        # workspace is in this directory
      gitlab.bit.init            # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.init
      gitlab.bit.merge-request   # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.merge-request
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

## Using Bit Tag and Export

```
gitlab.bit.tag-export        # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.tag-export
```

### Tag version

Specify the version tag for your components using the following methods. You can use any of these version keywords: `major`, `minor`, `patch`, and `pre-release`. 

- **Merge Request Labels:** Use the keyword directly as a label `major` or enclosed within square brackets `[major]`.
- **Merge Request or Commit Title:** Include the version keyword enclosed within square brackets `[major]` within your title text.

**Note:** Once a Merge Request is merged, it's tracked via its `merge commit` in the target branch. Therefore, the `merge commit` should be the last in the commit history for the action to read the version keyword from the Merge Request.

#### Git Commit

**Title:** Incorporate the version keyword in the title of your Git commit message.

**Note:** The version based on the latest commit title.


### Example usage
```
image: bitsrc/stable:latest

variables:
  # Not required to define BIT_CONFIG_USER_TOKEN or GITLAB_ACCESS_TOKEN if added to GitLab project CI/CD variables
  GIT_USER_NAME: “git_user_name”
  GIT_USER_EMAIL: “git”_user_email

release-components-job:
  stage: build
  script: 
    - |      
      cd my-workspace-dir        # workspace is in this directory
      gitlab.bit.init            # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.init
      gitlab.bit.tag-export      # https://github.com/bit-tasks/bit-docker-image/blob/main/scripts/gitlab.bit.tag-export
  rules:
    - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
```

### Automating Component Release

| Task                        | Example                         | Repo and Docs                                 |
|-----------------------------|---------------------------------|-----------------------------------------------|
| Initialize Bit             | [bit-init.yml](/gitlab-pipelines/bit-init.yml)          | [link](https://github.com/bit-tasks/init)    |
| Bit Verify Components  | [verify.yml](/gitlab-pipelines/verify.yml)                | [link](https://github.com/bit-tasks/verify)  |
| Bit Tag and Export        | [tag-export.yml](/gitlab-pipelines/tag-export.yml)  | [link](https://github.com/bit-tasks/tag-export) |
| Bit Pull Request Build  | [pull-request.yml](/gitlab-pipelines/pull-request.yml) | [link](https://github.com/bit-tasks/pull-request) |
| Commit Bitmap           | [commit-bitmap.yml](/gitlab-pipelines/commit-bitmap.yml) | [link](https://github.com/bit-tasks/commit-bitmap) |

  :arrow_down: [Download Files](https://github.com/bit-tasks/gitlab-pipeline-examples/raw/main/downloads/automating-component-releases.zip)

### Update Workspace Components, External Dependencies and Envs

*Coming Soon!*

### Sync Git Branches with Bit Lanes

*Coming Soon!*

## Contributor Guide

To contribute to GitLab, please make updates to the scripts that begin with `gitlab.bit.` found in the [Bit Docker Image Repository](https://github.com/bit-tasks/bit-docker-image).
