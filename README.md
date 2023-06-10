# deno.land/x - Publish Folder

**DO NOT USE THIS - UNDER DEVELOPMENT**

GitHub Action for publishing a built directory to the
[deno.land/x](https://deno.land/x) registry.

## How it works

First build your output to a folder. Then after you can use this action to
perform the following steps:

1. Push the folder to a separate orphan branch in your repository.
2. If specified, tag the commit to publish it to the deno.land/x registry.

## Why?

Generally you don't need a build step and it's not recommended, but _sometimes_
you do. For example:

- You might be building a `.wasm` file on every commit, which checking into
  source control causes issues like merge conflicts and large amounts of data in
  every commit.
- Your module might be very large and have numerous files. To reduce load times,
  you may want to bundle your module (that said, please don't bundle your
  dependencies or minimize your library code).
- Type checking your module takes a long time. You might want to create a
  targeted .d.ts file on every commit.

## Getting started

TODO:

- Need to outline how "read and write" permissions are necessary for a workflow
  or a scoped PAT
  (https://github.com/ad-m/github-push-action/issues/96#issuecomment-889984928)

### Option 1: Release workflow

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

        # Push to "build" branch and tag that commit
      - name: Publish to deno.land/x
        uses: <eventual-name-of-this-action>
        with:
          folder: dist
          token: ${{ secrets.GITHUB_TOKEN }}
          tag: ${{ github.event.inputs.version }}
```

This will create a new workflow in your repository that will:

1. Checkout the main branch.
1. Run your build script at `deno task build`, which outputs to the "dist"
   folder.
1. Push the contents of the "dist" folder to the "build" branch.
1. Tag that commit.

To run the workflow:

1. Check this file into your repository at `.github/workflows/release.yml`
1. Click on the "Actions" tab in your repo.
1. Select the "release" workflow.
1. On the right side, select "Run workflow", ensure the main branch is selected,
   enter the version number, and finally click the "Run workflow" button.

### Option 2: Pushing to the branch on every commit

Add the following to your regular release pipeline:

```yml
steps:
  - uses: actions/checkout@v3
  - uses: denoland/setup-deno@v1

  - name: Build
    run: deno task build

  - uses: <eventual-name-of-this-action>
    if: github.ref == 'refs/heads/main'
    with:
      folder: dist
      token: ${{ secrets.GITHUB_TOKEN }}
```

Then manually tag that branch with a version number to publish OR publish with a
tag using the `tag` input under certain conditions.

#### Note: deno.land/x registry supports tag prefixes

The deno.land/x registry supports
[publishing with tag prefixes](https://github.com/denoland/deno_registry2/blob/main/API.md#request).
For example, you can add `?version_prefix=deno/` to the end of your web hook url
in order to only publish when your repo is tagged with a tag name starting with
`deno/` and it will publish as what comes after.

This means that we can tag our repo with a version say `1.0.0`, then have the
workflow supply a tag of `deno/1.0.0` for the `tag`. When the "build" branch
gets tagged with `deno/1.0.0` then it will be published as `1.0.0` to
deno.land/x

### Inputs

Required inputs:

- `folder` - Folder in your directory to publish (ex. dist)
- `token` - GitHub token with push permissions to your repository.

Optional inputs:

- `branch` - Branch name to publish to. Defaults to "build"
- `tag` - Tag name to tag the published commit with. Does not tag if empty.
- `tagPrefix` - Tags the repo with a specified tag prefix when the workflow run
  is for a tag. Note: This cannot be specified when specifying a `tag` input.
- `git-user-name` - Git user.name to use when committing.
- `git-user-email` - Git user.email to use when committing.
