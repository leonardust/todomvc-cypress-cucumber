# Cypress, VsCode extensions and Cucumber pre-processor installation and configuration

# Cypress installation and configuration

Check node version

```sh
node -v

```

If node missing install node v16.16.0 (LTS) from [nodejs.org](https://nodejs.org/en/blog/release/v16.16.0)

To install cypress follow _[Cypress - installing cypress](https://docs.cypress.io/guides/getting-started/installing-cypress)_ or perform command below

```sh
npm install cypress --save-dev


```

Install npx package to executes  either from a local node_modules/.bin

```sh
npm install -g npx


```

Create and configure **tsconfig.json** file inside your cypress folder

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["es5", "dom"],
    "types": ["cypress", "node"]
  },
  "include": ["**/*.ts"]
}

```

# VsCode extensions

_[Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)_ - Material Design Icons for Visual Studio Code

_[Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)_ - Code formatter

To add prettier as default formatter add this code to the **settings.json**

```json
"editor.defaultFormatter": "esbenp.prettier-vscode"
```

_[Cucumber (Gherkin) Full Support](https://marketplace.visualstudio.com/items?itemName=alexkrechik.cucumberautocomplete)_

# Cucumber setup

### Install dependencies

1. Install cucumber dependencies follow _[@badeball/cypress-cucumber-preprocesor](https://www.npmjs.com/package/@badeball/cypress-cucumber-preprocessor)_ or execute command below

```sh
npm install --save-dev @badeball/cypress-cucumber-preprocessor@16.0.3


```

According to the _[quick-start.md](https://github.com/badeball/cypress-cucumber-preprocessor/blob/master/docs/quick-start.md#example-setup)_ Replace content of the __cypress.config.ts__ file by code below

```ts
import { defineConfig } from "cypress";
import createBundler from "@bahmutov/cypress-esbuild-preprocessor";
import { addCucumberPreprocessorPlugin } from "@badeball/cypress-cucumber-preprocessor";
import createEsbuildPlugin from "@badeball/cypress-cucumber-preprocessor/esbuild";

export default defineConfig({
  e2e: {
    specPattern: "**/*.feature",
    async setupNodeEvents(
      on: Cypress.PluginEvents,
      config: Cypress.PluginConfigOptions
    ): Promise<Cypress.PluginConfigOptions> {
      // This is required for the preprocessor to be able to generate JSON reports after each run, and more,
      await addCucumberPreprocessorPlugin(on, config);

      on(
        "file:preprocessor",
        createBundler({
          plugins: [createEsbuildPlugin(config)],
        })
      );

      // Make sure to return the config object as it might have been modified by the plugin.
      return config;
    },
  },
});

```

In case of errors in **cypress.config.ts** shown below:

![Missing dependencies](../pictures/missing_dependencies.png)

Install missing dependencies

```sh
npm install --save-dev @bahmutov/cypress-esbuild-preprocessor


```

and fix import error for `@badeball/cypress-cucumber-preprocessor/esbuild` by adding to **tsconfig.json**

```json
{
  "compilerOptions": {
    "paths": {
      "@badeball/cypress-cucumber-preprocessor/*": ["./node_modules/@badeball/cypress-cucumber-preprocessor/dist/subpath-entrypoints/*"]
    }
  }
}

```

### Configure step definition

Add configuration to the **settings.json**

```json
"cucumberautocomplete.customParameters": [],
"cucumberautocomplete.strictGherkinCompletion": true,
"cucumberautocomplete.steps": ["cypress/support/step_definitions/*.js"]
```

Add configuration to the **package.json**

```json
"cypress-cucumber-preprocessor": {
    "stepDefinitions": "cypress/support/step_definitions/**/*.js"   
}
```

Create stepdefinition folder in __cypress/support/step_definitions/__ folder

##### HTML Reports

To create html reports add to the **package.json** in **"cypress-cucumber-preprocessor"** section

```json
"html": {
      "enabled": true,
      "output": "cypress/reports/cucumber-html/cucumber-report.html"
},
"messages": {
      "enabled": true,
      "output": "cypress/reports/cucumber-ndjson/cucumber-report.ndjson"
},
```

##### JSON Reports

To create json reports add to the **package.json** in **"cypress-cucumber-preprocessor"** section

```json
"json": {
      "enabled": true,
      "output": "cypress/reports/cucumber-json/cucumber-report.json"
}
```

#### Multiple Cucumber HTML Report

Install [multiple-cucumber-html-reporter](https://github.com/WasiqB/multiple-cucumber-html-reporter) dependencies

```sh
npm install multiple-cucumber-html-reporter --save-dev


```

Create **multiple-cucumber-report-config.js** file in cypress folder and provide **jsonDir(folder)** where cucumber-report.json is
and specify path to the report folder **reportPath**

```js
const report = require("multiple-cucumber-html-reporter");

report.generate({
  jsonDir: "./cypress/reports/cucumber-json/",
  reportPath: "./cypress/reports/cucumber-report/",
});
```

Add in script section of **package.json** to generate multiple-cucumber-html-report

```json
"cy:report": "node ./cypress/multiple-cucumber-report-config.js"
```

#### Example report

![Multiple Cucumber HTML Report](../pictures/multiple-cucumber-html-reporter.png)

### Customize report 

config file `multiple-cucumber-report-config.js`

### Metadata

#### Browser information

Import keyword `After` and create step to write browser information to file `browserDetails.json` in root folder.

```typescript
import { After } from "@badeball/cypress-cucumber-preprocessor";

After(() => {
  cy.writeFile("browserDetails.json", Cypress.browser);
});
```

Import dependency to work with files

```js
const fs = require("fs");
```

Create function to read `browserDetails.json` and return json parsed content.

```js
function getBrowserDetails() {
  const fileContent = fs.readFileSync("./browserDetails.json", "utf-8");
  return JSON.parse(fileContent);
}
```

Create variable to store browser details and use them to pass information in metadata

```typescript
const browserDetails = getBrowserDetails();
```

```js
metadata: {
    browser: {
      name: browserDetails.displayName || "unknown-browser",
      version: browserDetails.majorVersion || "unknown-version",
    }
  }
```  

#### Device, Os and platform information

Import dependency to read os information

```js
const os = require("node:os");
```

Add information about device, platform name and version

```js
metadata: {
    device: os.hostname() || "unknown-device",
    platform: {
      name: os.platform() || "unknown-platform",
      version: os.version() || "unknown-version",
    },
  }

### Custom data

To add custom data use example code snippet inside `report.generate()` function

```js
customData: {
    title: "Run info",
    data: [
      { label: "Project", value: "todomvc-cucumber" },
      { label: "Release", value: "1.0.0" },
      { label: "Node Version", value: nodeVersion },
      { label: "Cypress Version", value: cypressVersion },
    ],
  },
```

To retrieve `node` and `cypress` version import dependencies

```js
const cypressVersion = require("cypress/package.json").version;
const nodeVersion = process.version;
```