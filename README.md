# BitBucket Pull Request Resource

Tracks changes for all _git_ branches managed by BitBucket (Server or Cloud) for pull requests.

**This resource is meant to be used with [`version: every`](https://concourse.ci/get-step.html#get-version).**

## Origins of this project

We have had some issues with the existing resources that implemented BitBucket pull request handling
and therefore decided to roll our own. Most of the scripts are based on work from other GitHub
repositories:

- [cathive/concourse-bitbucket-pullrequest-resource](https://github.com/cathive/concourse-bitbucket-pullrequest-resource)
- [philicious/concourse-git-bitbucket-pr-resource](https://github.com/philicious/concourse-git-bitbucket-pr-resource)
- [zarplata/concourse-git-bitbucket-pr-resource](https://github.com/zarplata/concourse-git-bitbucket-pr-resource)

## Installation

Add the following `resource_types` entry to your pipeline:

```yaml
resource_types:
  - name: bitbucket-pr
    type: docker-image
    source:
      repository: bofink/concourse-bitbucket-pullrequest-resource
      tag: latest # You'll probably want to use a verionsed tag to ensure that your builds are reproducible
```

## Source Configuration

- `base_url`: _Required_. base URL of the bitbucket server, without a trailing slash.
  For example: `http://bitbucket.local`
- `username`: _Required_. username of the user which have access to repository.
- `password`: _Required_. password of that user
- `project`: _Required_. project for tracking
- `repository`: _Required_. repository for tracking
- `limit`: _Optional_. limit of tracked pull requests `default: 100`.
- `git`: _Required_. configuration is based on the [Git
  resource](https://github.com/concourse/git-resource). The `branch` configuration
  from the original resource is ignored.
- `bitbucket_type`: _Optional_. `cloud` for BitBucket Cloud or `server` for a self-hosted BitBucket Server. `default: server`

### Example

```yaml
resources:
- name: my-repo-with-pull-requests
  type: bitbucket-pr
  source:
    url: http://bitbucket.local
    username: some-username
    password: some-password
    project: cathive
    repository: concourse-bitbucket-pullrequest-resource
    git:
      uri: https://github.com/cathive/concourse-bitbucket-pullrequest-resource
      private_key: ((git-repo-key))

jobs:
  - name: my build
    plan:
      - get: my-repo-with-pull-requests
        trigger: true
        version: every
      - task: unit test
          ...
          inputs:
            - name: my-repo-with-pull-requests
          run:
          ...
        on_failure:
          put: my-repo-with-pull-requests
          params:
            state: FAILED
            name: "unit test"
            dir: my-repo-with-pull-requests
        on_success:
          put: my-repo-with-pull-requests
          params:
            state: SUCCESSFUL
            name: "unit test"
            dir: my-repo-with-pull-requests
```

## Behavior

### `check`: Check for changes to all pull requests.

The current open pull requests fetched from Bitbucket server for given
project and repository. Update time are compared to the last fetched pull request.

If any pull request are new or updated or removed, a new version is emitted.

### `in`: Fetch the commit that changed the pull request.

This resource delegates entirely to the `in` of the original Git resource, by
specifying `source.branch` as the branch that changed, and `version.ref` as the
commit on the branch.

All `params` and `source` configuration of the original resource will be
respected.

### `out`: Update build task status.

This updates the build status of the task.

#### Parameters

- `dir`: _Optional_. set to name of the resource if resource name is different than repository name

## Release

```
docker build -t bofink/concourse-bitbucket-pullrequest-resource:latest .
docker push bofink/concourse-bitbucket-pullrequest-resource:latest
```
