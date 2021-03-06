<p align="center">
  <img alt="Lerna" src="https://cloud.githubusercontent.com/assets/952783/15271604/6da94f96-1a06-11e6-8b04-dc3171f79a90.png" width="480">
</p>

<p align="center">
  A tool for managing JavaScript projects with multiple packages.
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/lerna"><img alt="NPM Status" src="https://img.shields.io/npm/v/lerna.svg?style=flat"></a>
  <a href="https://travis-ci.org/lerna/lerna"><img alt="Travis Status" src="https://img.shields.io/travis/lerna/lerna/master.svg?style=flat&label=travis"></a>
  <a href="https://slack.lernajs.io/"><img alt="Slack Status" src="https://slack.lernajs.io/badge.svg"></a>
</p>

## About

Splitting up large codebases into separate independently versioned packages
is extremely useful for code sharing. However, making changes across many
repositories is *messy* and difficult to track, and testing across repositories
gets complicated really fast.

To solve these (and many other) problems, some projects will organize their
codebases into multi-package repostories (sometimes called [monorepos](https://github.com/babel/babel/blob/master/doc/design/monorepo.md)). Projects like [Babel](https://github.com/babel/babel/tree/master/packages), [React](https://github.com/facebook/react/tree/master/packages), [Angular](https://github.com/angular/angular/tree/master/modules),
[Ember](https://github.com/emberjs/ember.js/tree/master/packages), [Meteor](https://github.com/meteor/meteor/tree/devel/packages), [Jest](https://github.com/facebook/jest/tree/master/packages), and many others develop all of their packages within a
single repository.

**Lerna is a tool that optimizes the workflow around managing multi-package
repositories with git and npm.**

### What does a Lerna repo look like?

There's actually very little to it. You have a file system that looks like this:

```
my-lerna-repo/
  package.json
  packages/
    package-1/
      package.json
    package-2/
      package.json
```

### What can Lerna do?

The two primary commands in Lerna are `lerna bootstrap` and `lerna publish`.

`bootstrap` will link dependencies in the repo together.
`publish` will help publish any updated packages.

## Getting Started

> The instructions below are for Lerna 2.x which is currently in beta (actively being worked on).
> We recommend using it instead of 1.x for a new lerna project. Check the [wiki](https://github.com/lerna/lerna/wiki/1.x-Docs) if you need to see the 1.x README.

Let's start by installing Lerna globally with [npm](https://www.npmjs.com/).

```sh
$ npm install --global lerna
# install the latest 2.x version
$ npm install --global lerna@2.0.0-beta.13
```

Next we'll create a new [git](https://git-scm.com/) repository:

```sh
$ git init lerna-repo
$ cd lerna-repo
```

And now let's turn it into a Lerna repo:

```sh
$ lerna init
```

Your repository should now look like this:

```
lerna-repo/
  packages/
  package.json
  lerna.json
```

This will create a `lerna.json` configuration file as well as a `packages` folder.

## How it works

There are 2 modes when working with Lerna.

### Fixed/Locked mode (default)

Fixed mode Lerna projects operate on a single version line. The version is kept in the `lerna.json` file at the root of your project under the `version` key. When you run `lerna publish`, if a module has been updated since the last time a release was made, it will be updated to the new version you're releasing. This means that you only publish a new version of a package when you need to.

This is the mode that [Babel](https://github.com/babel/babel) is currently using. Use this if you want to automatically tie all package versions together. One issue with this approach is that a major change in any package will result in all packages having a new major version.

### Independent mode (`--independent`)

Independent mode Lerna projects allows maintainers to increment package versions independently of each other. Each time you publish, you will get a prompt for each package that has changed to specify if it's a patch, minor, major or custom change. 

Independent mode allows you to more specifically update versions for each package and makes sense for a group of components. Combining this mode with something like [semantic-release](https://github.com/semantic-release/semantic-release) would make it less painful. (There is work on this already at [atlassian/lerna-semantic-release](https://github.com/atlassian/lerna-semantic-release).

> The `version` key in `lerna.json` is ignored in independent mode.

## Commands

### Init

```sh
$ lerna init
```

Create a new lerna repo or upgrade an existing repo to the current version of Lerna.

> Lerna assumes the repo is already been initialized with `git init`.

1. Add lerna as a [`devDependency`](https://docs.npmjs.com/files/package.json#devdependencies) in `package.json` if it isn't already there.
2. Create a `lerna.json` config file to store the `version` number.
3. Create a `packages` folder if it's not created already.

Example output on a new git repo:

```sh
> lerna init
$ Lerna v2.0.0-beta.15
$ Creating packages folder.
$ Updating package.json.
$ Creating lerna.json.
$ Successfully created Lerna files
```

#### --independent, -i

```sh
$ lerna publish --independent
```

Use independent versioning mode.

### Bootstrap

```sh
$ lerna bootstrap
```
Bootstrap (setup) the packages in the current Lerna repo.
Installs all their dependencies and links any cross-dependencies.

1. Link together all lerna `packages` that have dependencies on each other.
  2. This doesn't currently use [npm link](https://docs.npmjs.com/cli/link), and rather a proxy to the actual package in the monorepo.
2. `npm install` all **outside** dependencies of each package.

Currently, what lerna does to link internal dependencies is replace the
`node_modules/package-x` with a link to the actual file in the repo.

#### How `bootstrap` works

Lets use `babel` as an example.

- `babel-core` has a dependency on `babel-generator` and `source-map` (among others).
-  In `babel-core`'s [`package.json`](https://github.com/babel/babel/blob/13c961d29d76ccd38b1fc61333a874072e9a8d6a/packages/babel-core/package.json#L28-L47), it has both of them as keys in `dependencies` like shown below.

```js
// babel-core package.json
{
  "name": "babel-core",
  ...
  "dependencies": {
    ...
    "babel-generator": "^6.9.0",
    ...
    "source-map": "^0.5.0"
  }
}
```

- Lerna checks if each dependency is also part of the lerna repo.
  - `babel-generator` is while `sourcemap` is not.
  - `sourcemap` is `npm install`ed like normal.
- `babel-core/node_modules/babel-generator` is replaced with 2 files
  - A `package.json` with keys `name` and `version`
  - A `index.js` with the contents `module.exports = require("relative-path-to-babel-generator-in-the-lerna-repo")`
- This links the `babel-generator` in `node_modules` with the actual `babel-generator` files.

### Publishing

```sh
$ lerna publish
```

Create a new release of the packages that have been updated.
Prompt for a new version.
Create a new git commit/tag in the process of publishing to npm.

1. Publish each module in `packages` that has been updated since the last version to npm with the [dist-tag](https://docs.npmjs.com/cli/dist-tag) `lerna-temp`.
  1. Run the equivalent of `lerna updated` to determine which packages need to be published.
  2. If necessary, increment the `version` key in `lerna.json`.
  3. Update the `package.json` of all updated packages to their new versions.
  4. Update all dependencies of the updated packages with the new versions.
  5. Create a new git commit and tag for the new version.
  6. Publish updated packages to npm.
2. Once all packages have been published, remove the `lerna-temp` tags and add the tags to `latest`.

> A temporary dist-tag is used at the start to prevent the case where only some of the packages are published; this can cause issues for users installing a package that only has some updated packages.

#### --npm-tag [tagname]

```sh
$ lerna publish --npm-tag=next
```

Publish to npm with the given npm [dist-tag](https://docs.npmjs.com/cli/dist-tag) (defaults to `latest`).

This option could be used to publish a [`prerelease`](http://carrot.is/coding/npm_prerelease) or `beta` version.

> Note: the `latest` tag is the one that is used when a user runs `npm install my-package`.
> To install a different tag, a user can run `npm install my-package@prerelease`.

#### --canary, -c

```sh
$ lerna publish --canary
```

The `canary` flag will publish packages in a more granular way (per commit).

It will take the current `version` and append the sha as the new `version`. ex: `1.0.0-alpha.81e3b443`.

More specifically, `canary` flag will append the current git sha to the current `version` and publish it.

> The use case for this is a per commit level release or a nightly release.

#### --skip-git

```sh
$ lerna publish --skip-git
```

Don't run any of the git commands.

> Only publish to npm; skip commiting, tagging, and pushing git changes (this only affects publish).

#### --force-publish [packages]

```sh
$ lerna publish --force-publish=package-2,package-4
# force publish all packages
$ lerna publish --force-publish=*
```

Force publish for the specified packages (comma-separated) or all packages using `*`.

> This basically skips the `lerna updated` check for changed packages and forces a package that didn't have a `git diff` change to be updated.

#### --yes

```sh
$ lerna publish --canary --yes
# skips `Are you sure you want to publish the above changes?`
```

Skip all confirmation prompts.
Useful in [Continuous integration (CI)](https://en.wikipedia.org/wiki/Continuous_integration) to automatically answer the publish confirmation prompt.

#### --repo-version

```sh
$ lerna publish --repo-version 1.0.1
# Applies version and skips `Select a new version for...` prompt
```

Specify the repo version check when publishing.
Useful for bypassing the user input prompt if you already know which version to publish.

### Updated

```sh
$ lerna updated
```

Check which `packages` have changed since the last release (the last git tag).

Lerna determines the last git tag created and runs `git diff --name-only v6.8.1` to get all files changed since that tag. It essentially returns an array of packages that have an updated file.

### Diff

```sh
$ lerna diff [package?]

$ lerna diff
# diff a specific package
$ lerna diff package-name
```

Diff all packages or a single package since the last release.

> Similar to `lerna updated`. This command runs `git diff`.

### Ls

```sh
$ lerna ls
```

List all of the public packages in the current Lerna repo.

### Run


```sh
$ lerna run [script] // runs npm run my-script in all packages that have it
$ lerna run test
$ lerna run build
```

Run an [npm script](https://docs.npmjs.com/misc/scripts) in each package that contains that script.

## Misc

Lerna will log to a `lerna-debug.log` file (same as `npm-debug.log`) when lerna encounters an error running a command.

Lerna also has support for [scoped packages](https://docs.npmjs.com/misc/scope).

Running `lerna` without arguments will show all commands/options.

### lerna.json

```js
{
  "lerna": "2.0.0-beta.9",
  "version": "1.1.3",
  "publishConfig": {
    "ignore": [
      "ignored-file",
      "*.md"
    ]
  }
}
```

- `lerna`: the current version of `lerna` being used.
- `version`: the current version of the repository.
- `publishConfig.ignore`: an array of globs that won't be included in `lerna updated/publish`. Use this to prevent publishing a new version just because of `README.md` typo.
