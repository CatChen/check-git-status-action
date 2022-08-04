# check-git-status-action

![test status badge](https://github.com/CatChen/check-git-status-action/actions/workflows/test.yml/badge.svg?event=push)

Do you check in dependency packages or build artefacts? If yes this GitHub Action helps you ensure they are not out-of-sync. Examples:

1. Say we set up to [run Yarn offline](https://classic.yarnpkg.com/blog/2016/11/24/offline-mirror/) and we check in Yarn offline mirror. We want to make sure the offline mirror is in sync with the dependencies declared in the `package.json`. We can set up a GitHub Workflow to run `yarn install` and then use this Action to check if the offline mirror is changed.
2. Say we generate TypeScript type definitions from [JSON Schemas](https://json-schema.org/). The generated TypeScript files are part of the codebase. We want to make sure people remember to regenerate these files when they modify any JSON Schema. We can use a GitHub Workflow to run the code generation and then use this GitHub Action to check if the files are changed. If they are changed this Action can commit the changes and add them to the Pull Request.

## Usage

Set up a GitHub Action like this:

```yaml
name: Verify Build

on:
  pull_request:
    branches: [main] # or [master] if that's name of the main branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Build
        run: |
          # replace the following line with the real build script
          touch some-build-artefact-newly-generated-in-the-build-process

      - uses: CatChen/check-git-status-action@v1.0.0
        with:
          fail-if-not-clean: true # optional
          push-if-not-clean: false # optional
          github-token: ${{ secrets.GITHUB_TOKEN }} # optional
          commit-message: "Changes detected by Check Git Status Action" # optional
```

Save the file to `.github/workflows/build.yml`. It will start working on new Pull Requests.

## Options

### `fail-if-not-clean`

When this option is set to `true` this action will fail if the project directory is no longer clean at the action execution time.

### `push-if-not-clean`

When this option is set to `true` this action will commit the new changes in the project directory and push the commit to the origin.

### `github-token`

The default value is `${{ github.token }}`, which is the GitHub token generated for this workflow. You can [create a different token with a different set of permissions](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) and use it here as well.

### `commit-message`

When `push-if-not-clean` is set to `true` and `git status` is not clean this option will be used as the commit message when committing the changes. Its default value is `"Changes detected by Check Git Status Action"`.

## FAQ

### The commit created by this Action doesn't trigger any Workflow.

> When you use the repository's `GITHUB_TOKEN` to perform tasks, events triggered by the `GITHUB_TOKEN` will not create a new workflow run. This prevents you from accidentally creating recursive workflow runs. -- [Source](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
