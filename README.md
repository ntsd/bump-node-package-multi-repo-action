# Bump Node Package Multiple Repository Action

A GitHub Action to bump node package version to multiple giving repositories.

Only support node package manager `npm`, `yarn`, or `pnpm`. determine by lock file `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`.

## Inputs

| field        | description                                                                                                                                                                                 | example                                                                                        |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| package      | `required` Package and version to bump                                                                                                                                                      | @zero-hub/client@1.0.0                                                                         |
| repositories | `required` Repositories to bump, separate by comma. use postfix `@` to specify the branch to bump. use `:` to specify the path of the package. eg, `username/repo@branch:./path_to_package` | hotcode-dev/zerohub-share@develop,hotcode-dev/zerohub-share@develop,hotcode-dev/zerohub-meet:. |
| gh_token     | GitHub token. Required when the destination repository is private.                                                                                                                          |                                                                                                |

|

## Examples Uses

### Dispatch Event

```yaml
name: bump package version
on:
  workflow_dispatch:
    inputs:
      package:
        description: "Package and version to bump. eg, `sdp-compact@0.0.6`"
        required: true
      repositories:
        description: "Repositories to bump, separate by comma."
        required: true
jobs:
  bump-package-version:
    runs-on: ubuntu-latest
    # permission contents and pull-requests is required
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v2
      - uses: ntsd/bump-node-package-multi-repo-action@v1
        with:
          package: ${{ github.event.inputs.package }}
          repositories: ${{ github.event.inputs.repositories }}
          gh_token: ${{ secrets.GITHUB_TOKEN }} # replace with your github token
```

> Because secrets.GITHUB_TOKEN not allow us to use the token for the other private repo, this should be replace with personal or organization GitHub Token. [Ref](https://github.com/orgs/community/discussions/46566)

### Use with Sematic Release

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v3
  - name: Semantic Release
    uses: cycjimmy/semantic-release-action@v3
    id: semantic # Need an `id` for output variables
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
  - name: Bump multiple repositories
    uses: ntsd/bump-node-package-multi-repo-action@v1
    if: steps.semantic.outputs.new_release_published == 'true'
    with:
      package: your_package@${{ steps.semantic.outputs.new_release_version }}
      repositories: "user/repo,user/repo2"
      gh_token: ${{ secrets.GITHUB_TOKEN }}
```
