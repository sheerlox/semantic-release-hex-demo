# SemanticReleaseHexDemo

This is a demo project to showcase version management of an Elixir project with [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) using [`semantic-release`](https://github.com/semantic-release/semantic-release/) and [`semantic-release-hex`](https://github.com/talent-ideal/semantic-release-hex).

Feel free to explore its content, commit history, or even fork it to test things out, as it's what it's meant for.

If you have any question or feedback, feel free to reply on the Elixir Forum thread or create a new [GitHub Discussion](https://github.com/sheerlox/semantic-release-hex-demo/discussions), I'll be happy to read and reply to you :heart:

## Learn more

- [`semantic-release` documentation](https://semantic-release.gitbook.io/semantic-release/)
- [`semantic-release-hex` documentation](https://github.com/talent-ideal/semantic-release-hex#readme)

## Setting up an existing Elixir project

### Before we begin

`semantic-release` uses git tags to determine the current version of a project, so you must make sure your latest version is tagged (and pointing to the right commit).

If you are using a different tag format than `v${version}`, see the [`tagFormat` option](https://semantic-release.gitbook.io/semantic-release/usage/configuration#tagformat).

### Install Node.js

You'll need Node.js `>=18.17`.

The recommended way to install it is with the [`asdf`](https://asdf-vm.com/) version manager (which also supports Elixir) by adding it to your [`.tool-versions`](./.tool-versions) and running `asdf install`.

Otherwise, you can [download it from nodejs.org](https://nodejs.org/en/download/).

### Install the tools

Install the required libraries:

```shell
npm i --save-dev semantic-release @semantic-release/changelog @semantic-release/git semantic-release-hex
```

Add `node_modules` to your `.gitignore`:

```shell
echo -e "\n# Node.js dependencies directory\nnode_modules" >> .gitignore
```

### Configuring `semantic-release`

`semantic-release` is [modular](https://semantic-release.gitbook.io/semantic-release/extending/plugins-list), [extendable](https://semantic-release.gitbook.io/semantic-release/developer-guide/plugin) and [configurable to your preferences](https://semantic-release.gitbook.io/semantic-release/usage/configuration). Here we'll setup a workflow for the `main` branch with the following steps:

1. Analyze the commits to determine the version bump type (if any)
2. Generate release notes from the commits
3. Write the release notes to the `CHANGELOG.md` file
4. Bump the version in `mix.exs`
5. Create and push a release commit (with the updated `CHANGELOG.md` and `mix.exs` files)
6. Create a release tag
7. Create a GitHub release

There's [a few ways](https://semantic-release.gitbook.io/semantic-release/usage/configuration#configuration-file) to configure `semantic-release`, but we'll chose to do it in `package.json` to avoid creating an extra file. Add a `release` key as follow:

```json
{
  "devDependencies": {...},

  "release": {
    "branches": ["main"],
    "plugins": [
      "@semantic-release/commit-analyzer",
      "@semantic-release/release-notes-generator",
      "@semantic-release/changelog",
      "semantic-release-hex",
      [
        "@semantic-release/git",
        {
          "assets": ["CHANGELOG.md", "mix.exs"],
          "message": "chore(release): v${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
        }
      ],
      "@semantic-release/github"
    ]
  }
}
```

_Note: if you want your CHANGELOG to include all commit types (and add a bit of color with category emojis) like the one in this demo, checkout [`@insurgent/conventional-changelog-preset`](https://github.com/insurgent-lab/conventional-changelog-preset#readme)_.

### CI configuration (GitHub Actions)

See [`release.yml`](./.github/workflows/release.yml).

For the `CI_GITHUB_TOKEN` secret needed to bypass branch protection when pushing the release commit, see [this StackOverflow answer](https://stackoverflow.com/questions/74744498/github-pushing-to-protected-branches-with-fine-grained-token/76550826#76550826) on how to create a fine-grained token with the right scopes (I know, I really need to [update the docs](https://github.com/semantic-release/semantic-release/issues/2883)).

## Running from the CLI

_Note: If you're concerned about security, you can run `gh auth refresh` after using `gh auth token` to regenerate [`gh`](https://cli.github.com/)'s token and invalidate the previous one (or you can create a fine-grained token by visiting the StackOverflow link in [CI configuration](#ci-configuration-github-actions))._

### To test everything is working

```shell
GH_TOKEN=$(gh auth token) npx semantic-release --dry-run
```

### To make an actual release

```shell
GH_TOKEN=$(gh auth token) npx semantic-release --no-ci
```

_Note: if your git is configured to automatically sign tags, this won't work [due to a known issue](https://github.com/semantic-release/semantic-release/issues/3065). Run `git config tag.gpgSign false` in your repo first._
