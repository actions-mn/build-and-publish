# build-and-publish

Meta action to compile and publish pages in one step

- [Prerequisites](#prerequisites)
- [Usage](#usage)
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

## Usage

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    agree-to-terms: true
```

or

```yml
...
- uses: actions-mn/build-and-publish@main
  with:
    source-path: documents
    config-file: metanorma.yml
    agree-to-terms: true
    output-dir: site
    use-bundler: false
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
