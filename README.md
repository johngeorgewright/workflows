# Workflows

> Reusable github action workflows

## Dependency Cache

Speed up your Node.js builds by caching the dependencies.

```yaml
jobs:
  cache:
    uses: johngeorgewright/workflows/.github/workflows/yarn-cache.yml@master
    # Or if you're using NPM...
    # uses: johngeorgewright/workflows/.github/workflows/npm-cache.yml@master

  test:
    needs: [cache]
    name: Test
    runs-on: dmg-arc-ubuntu

    steps:
      - name: Checkout project
        uses: actions/checkout@v4

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc

      # !!IMPORTANT!!
      # Make sure to restore the cache, like so:
      - name: Restore cache
        uses: actions/cache@v4
        with:
          key: ${{ needs.cache.outputs.key }}
          path: ${{ needs.cache.outpus.path }}

      # ...
```

