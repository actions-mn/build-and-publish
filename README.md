# build-and-publish

Meta action to compile and publish pages in one step

- [Prerequisites](#prerequisites)
- [Exaples](#exaples)

## Prerequisites

Before `uses: actions-mn/build-and-publish@main` ensure that `metanorma-cli` installed. You can install it by defferent ways:

1. With GitHub Action
   ```yml
   ...
   - uses: actions-mn/setup@main
   ...
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
   > NOTE: if metanorma-cli installed with bundle install make sure to path use-bundler: true to the action
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

## Exaples

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
