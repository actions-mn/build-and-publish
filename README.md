# build-and-publish

[![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/actions-mn/build-and-publish)](https://github.com/actions-mn/build-and-publish/releases)
[![GitHub actions](https://github.com/actions-mn/build-and-publish/actions/workflows/test.yml/badge.svg)](https://github.com/actions-mn/build-and-publish/actions/workflows/test.yml)

Meta action to compile and publish pages in one step

By default, the action will try to compile/generate docs and upload them as a pages artifact for later deployment. If GitHub Pages are not enabled for a repo where this action is used it will just upload an artifact with the name `metanorma-build-artifact`. Check [action.yml](./action.yml) for more details.

- [Description](#description)
- [Prerequisites](#prerequisites)
- [Scenarios](#scenarios)
  * [Destinations](#destinations)
  * [Simple, if `metanorma.yml` is in the root of the repository](#simple--if--metanormayml--is-in-the-root-of-the-repository)
  * [For `metanorma` installed with bundler](#for--metanorma--installed-with-bundler)
  * [For documentation in a non-root directory](#for-documentation-in-a-non-root-directory)
  * [Non-standart output directory](#non-standart-output-directory)
- [Complete examples](#complete-examples)
  * [Docker](#docker)
  * [Ubuntu](#ubuntu)
  * [Private flavors](#private-flavors)
  * [Full examples](#full-examples)
  * [PR previews](#pr-previews)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Prerequisites

1. Make sure that [GitHub Pages enabled in settings](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site#creating-your-site)
1. Before `uses: actions-mn/build-and-publish@main` ensure that `metanorma-cli` is installed. You can install it in different ways:

    1. With GitHub Action
       ```yml
       ...
       - uses: actions-mn/setup@main
       ...
       ```
    1. With Ruby
       ```yml
       ...
       - uses: ruby/setup-ruby@v1
         with:
           ruby-version: '3.1'
       - run: gem install metanorma-cli`
       ...
       ```
       or with `Gemfile` (that contains `gem "metanorma-cli"`)
       ```yml
       ...
       - uses: ruby/setup-ruby@v1
         with:
           ruby-version: '3.1'
       - run: bundle install`
       ...
       ```
       > NOTE: if metanorma-cli is installed with bundle install make sure to path use-bundler: true to the action
    1. With docker container:
       ```yml
       ...
       jobs:
         ...
         metanorma:
           runs-on: ubuntu-latest
           container:
             image: metanorma/metanorma:latest
           steps:
             ...
       ```


## Scenarios

### Destinations

The action supports several destinations:
- `default` - Deploy GitHub Pages if possible (as `gh-pages` do) otherwise upload artifact (as `artifact` do)
  ```yml
  ...
  + uses: actions-mn/build-and-publish@main
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      agree-to-terms: true
  ```
- `gh-pages` - Prepare pages artifact for `actions/deploy-pages` action, and fail if not possible
  ```yml
  ...
  + uses: actions-mn/build-and-publish@main
    with:
      destination: gh-pages
      agree-to-terms: true
  ```
- `artifact` - Upload site artifact as zip archive
  ```yml
  ...
  + uses: actions-mn/build-and-publish@main
    with:
      destination: artifact
      agree-to-terms: true
  ```

> *Important note*. In the case of `default` you must pass `token`, it's required to determine if the repository has GitHub Pages enabled

### Simple, if `metanorma.yml` is in the root of the repository

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    agree-to-terms: true
```

### For `metanorma` installed with bundler

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    agree-to-terms: true
    use-bundler: true
```

### For documentation in a non-root directory

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    source-path: non-root-doc
    agree-to-terms: true
```

### Non-standard output directory

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    agree-to-terms: true
    output-dir: my-output
```

## Complete examples

### Docker

```yml
name: generate

on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: metanorma/metanorma:latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Cache Metanorma assets
        uses: actions-mn/cache@v1

      - name: Metanorma generate site
        uses: actions-mn/build-and-publish@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          agree-to-terms: true

  deploy:
    if: ${{ github.ref == 'refs/heads/main' }}
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Ubuntu

```yml
name: generate

on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest # windows-latest or macos-latest also can be here
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Metanorma
        uses: actions-mn/setup@v1

      - name: Cache Metanorma assets
        uses: actions-mn/cache@v1

      - name: Metanorma generate site
        uses: actions-mn/build-and-publish@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          agree-to-terms: true

  deploy:
    if: ${{ github.ref == 'refs/heads/main' }}
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Private flavors

```yaml
name: generate

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: metanorma/metanorma:latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup BSI
        uses: actions-mn/setup-flavors@v1
        with:
          extra-flavors: bsi
          github-pakages-token: ${{ secrets.GITHUB_PAT_TOKEN }}
          use-bundler: true

      - name: Metanorma generate site
        uses: actions-mn/build-and-publish@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          agree-to-terms: true
```

### Private repository

In the case of a private repository the GitHub Pages are disabled so in this case, it makes sense to set `destination: artifact`. And no `deploy` job required

```yaml
name: generate

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: metanorma/metanorma:latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Metanorma
        uses: actions-mn/setup@v1

      - name: Metanorma generate site
        uses: actions-mn/build-and-publish@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          destination: artifact
          agree-to-terms: true
```

### PR previews 

It's also possible to deploy pages from PR for preview purposes.

For the complete example look at [`actions-mn/deploy-pages` repository](https://github.com/actions-mn/deploy-pages/blob/main/README.md#complete-example)
