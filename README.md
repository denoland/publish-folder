# deno.land/x - Publish Folder

**DO NOT USE THIS - UNDER DEVELOPMENT**

GitHub Action for publishing a folder to the [deno.land/x](https://deno.land/x) registry.

## How it works

First you build your output to a folder. Then after you can use this action which will:

1. Push the folder to a separate orphan branch in your repository.
2. Optionally tag that commit to publish to the deno.land/x registry.

## Why?

Generally you don't need a build step and overall it's not recommended, but *sometimes* you do. For example:

* You might be building a `.wasm` file on every commit and checking that into source control causes issues like merge conflicts and every commit has a massive amount of data in it.
* Your module might be very large and have a lot of files. To reduce load times, you may want to bundle your module (without bundling dependencies please)
* Type checking your module takes a long time. Maybe you want to create a targeted .d.ts file on every commit.

## Getting started

