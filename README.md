# hmpps-assess-risks-and-needs-github-actions

Reusable Github workflows and actions across the ARNS space.

### How to run Cypress tests in parallel, split by duration

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

