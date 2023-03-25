# deno.land/x - Publish Folder

**DO NOT USE THIS - UNDER DEVELOPMENT**

GitHub Action for publishing a built directory to the
[deno.land/x](https://deno.land/x) registry.

## How it works

First you build your output to a folder. Then after you can use this action
which will:

1. Push the folder to a separate orphan branch in your repository.
2. Optionally tag that commit to publish to the deno.land/x registry.

## Why?

Generally you don't need a build step and overall it's not recommended, but
_sometimes_ you do. For example:

- You might be building a `.wasm` file on every commit and checking that into
  source control causes issues like merge conflicts and every commit has a
  massive amount of data in it.
- Your module might be very large and have a lot of files. To reduce load times,
  you may want to bundle your module (without bundling dependencies or
  minimizing the code please).
- Type checking your module takes a long time. Maybe you want to create a
  targeted .d.ts file on every commit.

## Getting started

TODO:

- Need to outline how "read and write" permissions are necessary for a workflow
  or a scoped PAT

### Release workflow

You can create a release workflow that looks similar to the following:

```yml
# .github/workflows/release.yml
name: release

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version number to tag'
        required: true
        type: string

jobs:
  release:
    name: release
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - name: Ensure main branch
        if: github.ref != 'refs/heads/main'
        run: |
          echo "Run this workflow on your main branch."
          exit 1

        # Setup
      - uses: actions/checkout@v3
      - uses: denoland/setup-deno@v1

        # Build
      - name: Build
        run: deno task build

        # Push to "builds" branch and tag that commit
      - use: <eventual-name-of-this-action>
        with:
          folder: dist
          token: ${{ secrets.GITHUB_TOKEN }}
          tag: ${{ github.event.inputs.version }}
```

This will create a new workflow in your repository that will:

1. Checkout the main branch
1. Run your build script at `deno task build`, which outputs to the "dist"
   folder.
1. Push the contents of the "dist" folder to the "builds" branch.
1. Tag that commit.

To run the workflow:

1. Check this file into your repository at `.github/workflows/release.yml`
1. Click on the "Actions" tab in your repo.
1. Select the "release" workflow.
1. On the right side, select "Run workflow", ensure the main branch is selected,
   enter the version number, and finally click the "Run workflow" button.

### Pushing to the branch on every commit

```yml
steps:
  - uses: actions/checkout@v3
  - uses: denoland/setup-deno@v1
  - name: Build
    run: deno task build
  - use: <eventual-name-of-this-action>
    if: github.ref == 'refs/heads/main'
    with:
      folder: dist
      token: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

Required inputs:

- `folder` - Folder in your directory to publish (ex. dist)
- `token` - GitHub token with push permissions to your repository.

Optional inputs:

- `branch` - Branch name to publish to. Defaults to "builds"
- `tag` - Tag name to tag the published commit with. Does not tag if empty.
- `git-user-name` - Git user.name to use when committing.
- `git-user-email` - Git user.email to use when committing.
