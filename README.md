# build-and-publish

Meta action to compile and publish pages in one step

Exaples:

```yml
- uses: actions-mn/build-and-publish@main
  with:
    agree-to-terms: true
```

Or

```yml
- uses: actions-mn/build-and-publish@main
  with:
    source-path: documents
    config-file: metanorma.yml
    agree-to-terms: true
    output-dir: site
    use-bundler: false
```