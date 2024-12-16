# Bump Node Package Multiple Repository Action

A Github Action to bump node package version to multiple giving repositories.

## Inputs

| field           | description                                                                                                                                                                      | example                                                                                        |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| package         | Package and version to bump                                                                                                                                                      | @zero-hub/client@1.0.0                                                                         |
| repositories    | Repositories to bump, separate by comma. use postfix `@` to specify the branch to bump. use `:` to specify the path of the package. eg, `username/repo@branch:./path_to_package` | hotcode-dev/zerohub-share@develop,hotcode-dev/zerohub-share@develop,hotcode-dev/zerohub-meet:. |
| gh_token        | Github token. Required when the destination repository is private.                                                                                                               |                                                                                                |
| package_manager | Package manager to use. only support `npm`, `yarn`, or `pnpm`.                                                                                                                   | npm                                                                                            |

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
          package_manager: "npm"
```

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
      package_manager: "npm"
```
