# check-git-status-action

[![Test](https://github.com/CatChen/check-git-status-action/actions/workflows/test.yml/badge.svg?branch=main&event=push)](https://github.com/CatChen/check-git-status-action/actions/workflows/test.yml)
[![Release](https://github.com/CatChen/check-git-status-action/actions/workflows/release.yml/badge.svg?branch=main&event=schedule)](https://github.com/CatChen/check-git-status-action/actions/workflows/release.yml)

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
      - uses: actions/checkout@v6

      - name: Build
        run: |
          # replace the following line with the real build script
          touch some-build-artefact-newly-generated-in-the-build-process

      - uses: CatChen/check-git-status-action@v2
        with:
          fail-if-not-clean: true # optional
          push-if-not-clean: false # optional
          request-changes-if-not-clean: true # optional
          push-token: ${{ secrets.GITHUB_TOKEN }} # optional
          request-changes-token: ${{ secrets.GITHUB_TOKEN }} # optional
          commit-message: 'Changes detected by Check Git Status Action' # optional
          request-changes-comment: 'Changes detected by Check Git Status Action' # optional
          targets: '.' #optional
```

Save the file to `.github/workflows/build.yml`. It will start working on new Pull Requests.

## Options

### `fail-if-not-clean`

When this option is set to `true` this action will fail if the project directory is no longer clean at the action execution time.

### `push-if-not-clean`

When this option is set to `true` this action will commit the new changes in the project directory and push the commit to the origin.

### `request-changes-if-not-clean`

When this option is set to `true` and it's run on a Pull Request this action will request change if the project directory isn't clean.

### `push-token`

The default value is `${{ github.token }}`, which is the GitHub token generated for this workflow. This token determines the identity of the user that makes the commit and pushes it to the current branch. By default, it's GitHub Action bot. However, one GitHub Action doesn't trigger another. That means when a GitHub Action bot pushes to a branch it doesn't trigger any other GitHub Actions that are usually triggered by events from this branch or this Pull Request.

You can [create a different token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to work around this. You can also call other Workflows that aren't being triggered by this Workflow. You can add `workflow_run` event to the other Workflows so they are triggered explicitly after a successful run of the current Workflow.

### `request-changes-token`

The default value is `${{ github.token }}`, which is the GitHub token generated for this workflow. This token determines the identity of the user that requests changes. GitHub doesn't allow Pull Request author to request changes. Make sure this token doesn't represent a user that could be the Pull Request author. Usually it's fine to leave it with the default value, unless some Pull Requests are authored by the GitHub Action bot.

### `commit-message`

When `push-if-not-clean` is set to `true` and `git status` is not clean this option will be used as the commit message when committing the changes. Its default value is `"Changes detected by Check Git Status Action"`.

### `request-changes-comment`

When `request-changes-if-not-clean` is set to `true` and `git status` is not clean this option will be used as the comment posted to the Pull Request along side with the request changes. Its default value is `"Changes detected by Check Git Status Action"`.

### `targets`

The default value is `"."`. For example, it could be `"src"` or `"src/**/*.ts"` for a typical TypeScript project with source code files in the `src` directory.

For multiple targets, use one target per line:

```yaml
targets: |
  schema.graphql
  src/__graphql__
  src/**/*.ts
```

Use newline-separated targets when matching multiple files or directories. This action does not perform shell expansion on targets (so brace patterns like `"{src,lib}"` are passed to git as literal pathspecs), but each target can still use git pathspec/glob syntax such as `src/**/*.ts`.

### `github-token` (deprecated)

The default value is `${{ github.token }}`, which is the GitHub token generated for this workflow. You can [create a different token with a different set of permissions](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) and use it here as well.

## FAQ

### The commit created by this Action doesn't trigger any Workflow.

> When you use the repository's `GITHUB_TOKEN` to perform tasks, events triggered by the `GITHUB_TOKEN` will not create a new workflow run. This prevents you from accidentally creating recursive workflow runs. -- [Source](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)

Use the [`workflow_run` event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_run) in your other Workflows so they are triggered when this Action finishes. For example, if the Workflow running this Action is named as `Verify Build` like the example from above use the following code to trigger a follow-up Workflow name `Post-Verification`.

```yaml
name: Post-Verification

on:
  workflow_run:
    branches: [master]
    workflows: ['Verify Build']
    types: [completed]
```
