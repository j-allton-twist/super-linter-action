# Brandwatch Superlinter Action

This repo houses a github action that can be used from to validate PR changes against linting rules contained here.

This allows us to maintain a centralised set of rules for the organisation instead of having them distributed across multiple repositories, avoiding rules becoming out of sync.

This is an extension of the [github super-linter action](https://github.com/github/super-linter) and is intended for use on Brandwatch repositories.

Github Super-linter contains default rules for a wide variety of [linters](https://github.com/github/super-linter#supported-linters). These rules for a particular linter can be overriden by adding an appropriately named file to the [rules](rules/) directory of this repository.

Default rules can be found here: [https://github.com/github/super-linter/tree/master/TEMPLATES](https://github.com/github/super-linter/tree/master/TEMPLATES)

## Why is this repository public?

Github actions do not currently support actions contained within private repositories. As this repository contains no sensitive data, only linting rules it is made public. If Github enable the ability to use private repositories in actions then this will likely be made private.

## How do I add this to my repository

All you need to do is add a `linter.yml` file to the `.github/workflows/` directory of your repository with the following configuration.

```yaml
---
name: Lint Code Base

on:
  pull_request:
    branches: [ master, main, develop ]

jobs:
  build:
    name: Lint Code Base
    runs-on: ubuntu-latest

    steps:
      # checkout the repo to be linted
      - name: Checkout Code
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      # run the linter
      - name: Lint Code Base
        uses: BrandwatchLtd/super-linter-action@HEAD #@HEAD ensures you always use the latest rules
        env:
          VALIDATE_ALL_CODEBASE: false #prevent validating files that are not part of the PR
          DEFAULT_BRANCH: ${{ github.base_ref }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

See [Github Actions Documentation](https://docs.github.com/en/actions/reference) for more detail or help with different configuration.

## A note on disabling linters for specific lines of code

Some linters support suppressing rules directly in the file OOTB, some do not. E.g. hadolint allows the user to add a comment like `# hadolint ignore=DL4006` above the line they want the linting check suppressed for.
For checkstyle this is enabled using the `SuppressWithPlainTextCommentFilter` module which has been configured to allow the use of comments to temporarily disable/enable modules.

```java
//CheckStyleOff: AbbreviationAsWordInName
String myHTTPURL = "http://myurl.com";
//CheckStyleOn: AbbreviationAsWordInName
```

Multiple module names can be specified as pipe separated strings

```java
//CheckStyleOff: LineLength|AbbreviationAsWordInName
String myHTTPURL = "http://some-long-stuff-some-long-stuff-some-long-stuff-some-long-stuff-some-long-stuff-some-long-stuff-some-long-stuff-some-long-stuff-some-long-stuff.com";
//CheckStyleOn: LineLength|AbbreviationAsWordInName
```

## Rule X is stupid!

If you are a Brandwatch employee and feel rules are too strict or incorrect, please feel free to raise a PR to change them. There maybe some debate whether your change is in line with the department's coding standards. If you are not a brandwatch employee, your opinion on our internal coding standards is considered moot and your PR will be ignored/rejected.

## I want to test some rule changes against a PR.

Simply create a branch with your rule changes on this repo or on your fork of this repository and change the repository/branch that the `Lint Code Base` step uses in your PR. E.g.

```yaml
- name: Lint Code Base
        uses: BrandwatchLtd/super-linter-action@my-branch
```

or if your changes to the linter are on a fork

```yaml
- name: Lint Code Base
        uses: my-github-user/super-linter-action@my-branch
```

Your PR checks should now use the modified rules from your branch.

## Running Brandwatch Superlinter Action Locally

Add this repo to your path, e.g. for bash users:

```bash
# Run this from the directory this README.md is in.
echo "export PATH=\$PATH:$(pwd)" >> ~/.bashrc
source ~/.bashrc
```

Then run `superlint` from anywhere within a git repository to lint the changes from the remotes HEAD branch!
By default, this will assume the remotes name is `origin`.
If you wish to target a remote repository with a different name, simply pass in the remote name using the `--remote`
named parameter (or `-r` for short), e.g. for a remote named `upstream`:

```bash
superlint --remote upstream
```

To override the branch being used for determining the files that have changed you can specify the `--branch` named
parameter (`-b` for short). e.g. to diff against the remote's `awesome-feature` branch do

```bash
superlint --branch awesome-feature
```

By default, only changed files will be linted, however this can be overridden to lint all files by passing in
the `--lint-all` flag (`-a` for short). e.g.

```bash
superlint --lint-all
```

## Using the Checkstyle rules in Intellij
While checkstyle is only one of the many linters that are run by this action it, as much of the code we work
with is Java it can be useful to run checkstyle directly in the IDE to get realtime feedback on rule violations.

You can add the checkstyle plugin and point it to the checkstyle rules file in this repo. In Intellij do the following:

* Intellij > Preferences > Plugins > Search for "CheckStyle - IDEA". Install the plugin.
* Intellij > Preferences > Toools > Checkstyle
* Add a configuration file with this url: `https://raw.githubusercontent.com/BrandwatchLtd/super-linter-action/main/rules/sun_checks.xml`
* Ensure the Checkstyle version is set to at least 8.42 and the scope is set to include tests.
