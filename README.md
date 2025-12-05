# Workflows

> Reusable github action workflows

## Dependency Cache

Speed up your Node.js builds by caching the dependencies.

```yaml
jobs:
  test:
    needs: [cache]
    name: Test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout project
        uses: actions/checkout@v4

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc

      - name: Dependencies
        uses: johngeorgewright/workflows/actions/yarn-cache@main

      # Or NPM
      # - name: Dependencies
      #   uses: johngeorgewright/workflows/actions/npm-cache@main
```

