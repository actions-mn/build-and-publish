# build-and-publish

Meta action to compile and publish pages in one step

- [Prerequisites](#prerequisites)
- [Scenarios](#scenarios)
  * [Simple, if `metanorma.yml` is in the root of the repository](#simple--if--metanormayml--is-in-the-root-of-the-repository)
  * [For `metanorma` installed with bundler](#for--metanorma--installed-with-bundler)
  * [For documentation in a non-root directory](#for-documentation-in-a-non-root-directory)
  * [Non-standart output directory](#non-standart-output-directory)
- [Complete examples](#complete-examples)
  * [Docker](#docker)
  * [Ubuntu](#ubuntu)

## Prerequisites

Before `uses: actions-mn/build-and-publish@main` ensure that `metanorma-cli` is installed. You can install it in different ways:

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

### Simple, if `metanorma.yml` is in the root of the repository

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    agree-to-terms: true
```

### For `metanorma` installed with bundler

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    agree-to-terms: true
    use-bundler: true
```

### For documentation in a non-root directory

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    source-path: non-root-doc
    agree-to-terms: true
```

### Non-standart output directory

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
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
        uses: actions/checkout@v3

      - name: Cache Metanorma assets
        uses: actions-mn/cache@v1

      - name: Metanorma generate site
        uses: actions-mn/build-and-publish@main
        with:
          agree-to-terms: true
          output-dir: ./_site

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
        uses: actions/deploy-pages@v1
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
        uses: actions/checkout@v3

      - name: Setup Metanorma
        uses: actions-mn/setup@main

      - name: Cache Metanorma assets
        uses: actions-mn/cache@v1

      - name: Metanorma generate site
        uses: actions-mn/build-and-publish@main
        with:
          agree-to-terms: true
          output-dir: ./_site

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
        uses: actions/deploy-pages@v1
```
