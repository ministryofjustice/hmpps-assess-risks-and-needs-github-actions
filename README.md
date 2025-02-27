# hmpps-assess-risks-and-needs-github-actions

Reusable Github workflows and actions across the ARNS space.

## Run Cypress tests in parallel, split by duration

Install [cypress-split](http://example.com) plugin in your test suite/project:

```bash
npm i -D cypress-split
```

Call the plugin inside `setupNodeEvents` in your `cypress.config.js`:

```js
// cypress.config.js
const { defineConfig } = require('cypress')
// https://github.com/bahmutov/cypress-split
const cypressSplit = require('cypress-split')

module.exports = defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      cypressSplit(on, config)
      // IMPORTANT: return the config object
      return config
    },
  },
})
```

Configure your GH Actions workflow.

Here's a complete workflow example running 4 parallel containers:

```yaml
name: Run Cypress tests

on:
  pull_request:

permissions:
  actions: write

jobs:
  get-timings:
    uses: ministryofjustice/hmpps-assess-risks-and-needs-github-actions/.github/workflows/cypress_get_timings.yml@v1
    with:
      spec-file-patterns: 'e2e/**/*.cy.js other-e2e/**/*.cy.js'

  e2e-test:
    runs-on: ubuntu-latest
    needs: get-timings
    strategy:
      fail-fast: false
      matrix:
        container: [ 1, 2, 3, 4 ]
    outputs:
      timings-1: ${{ steps.timings.outputs.t1 }}
      timings-2: ${{ steps.timings.outputs.t2 }}
      timings-3: ${{ steps.timings.outputs.t3 }}
      timings-4: ${{ steps.timings.outputs.t4 }}
    steps:
      - uses: ministryofjustice/hmpps-assess-risks-and-needs-github-actions/.github/actions/cypress/create_timings_file@v1
        with:
          timings: ${{ needs.get-timings.outputs.timings }}

      - name: Run the tests
        uses: cypress-io/github-action@v6
        with:
          publish-summary: false
        env:
          SPLIT: ${{ strategy.job-total }}
          SPLIT_INDEX: ${{ strategy.job-index }}
          SPLIT_FILE: 'timings.json'

      - name: Output updated timings
        if: success() || failure()
        id: timings
        run: echo "t${{ matrix.container }}=$(jq -c . < timings.json)" >> $GITHUB_OUTPUT

  save-timings:
    if: success() || failure()
    uses: ministryofjustice/hmpps-assess-risks-and-needs-github-actions/.github/workflows/cypress_save_timings.yml@v1
    needs:
      - get-timings
      - e2e-test
    with:
      initial-timings: ${{ needs.get-timings.outputs.timings }}
      updated-timings: ${{ toJSON(needs.e2e-test.outputs) }}
```
