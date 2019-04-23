# ⌘ Re-Cli generator

The core motivation is to reduce file creation routine from dev process for one side. From other is to increase the code quality by automating new code injection. It allows you to have strict code agreements cross over their project, and decrease the onboarding process.

## API

Full API for generators are here:

```js
const { cliOf, useImport, usePath, useCustom } = require("@re/cli");

cliOf('Create reducer', module)
  .ask({
    name: 'reducerName',
    message: 'Reducer name',
    type: 'input'
  })
  .setKey('key')
  .ask({
    name: 'model',
    message: 'Reducer data model',
    type: 'input'
  })
  .setAnswers((answers) => {
    // extend answers object with new data
    return {
      reducerName: answers.reducerName,
      model: answers.model,
      upperCaseReducerName: answers.reducerName.toUpperCase(),
      otherVariable: 'My name is John Cena',
    }
  });
  .move('../../fake/destination', ['./reducer.template.js'])
  .rename('../../fake/destination/reducer.template.js', (answers) => {
    return `${answers.reducerName}.js`;
  })
  .useHooks('../../fake/destination/store.js', (answers) => [
    useImport(`./${answers.file2}`, answers.file2),
    usePath(`./${answers.file2}`),
    useCustom({regex, content}),
  ])
  .ask({
    message: 'Witch styles do you use?',
    type: 'list',
    name: 'style',
    choices: [
      {name: 'style.template.scss', value: 'Scss'},
      {name: 'style.template.less', value: 'Less'},
    ],
  })
  .call((answers) => {
    // do any wiered stuff you need,
    // use setAnswers other vice to store result
    return axios.get(`./ping-hook/?${answers.style}`)
  })
  .check((answers, goTo) => {
    if (answers.continue) {
      goTo('begining')
    }
  })
  .move('../../fake/destination', (answers) => [
    {from: './' + answers.style, to: 'style/' + answers.style}
  ])
```

**Notes** all callback functions can be async or return promise, to apply pause on the task.

## ⚛️ Hooks

The core feature all around is code injectors to existed files. We call it hooks, to be on hype.

It works like so. Let's imagine you have a file called `router.js`. After new `route` generation you want to append new route here.

So, let's add some hook to the `router.js`,

```js
import React from "react";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

import { HomeRoute } from './home-route';
/* re-cli:use-import */

function AppRouter() {
  return (
    <Router>
        <Route path="/about/" component={HomeRoute} />
        {/* re-cli:use-route */}
      </div>
    </Router>
  );
}

export default AppRouter;
```

So step after generation we expect to have like this

```js
import React from "react";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

import { HomeRoute } from './home-route';
import { NewRoute } from './new-route';
/* re-cli:use-import */

function AppRouter() {
  return (
    <Router>
        <Route path="/about/" component={HomeRoute} />
        <Route path="/new-route/" component={NewRoute} />
        {/* re-cli:use-route */}
      </div>
    </Router>
  );
}

export default AppRouter;
```

To make it done we have hooks:

- useImport -> useImport(`{ NewRoute }`, `./new-route`) -> import { NewRoute } from './new-route';
- usePath -> usePath(`./new-route`) -> `'./new-route',`
- useModuleName -> useModuleName(`NodeModule`) -> `NodeModule,`
- useCustom -> useCustom({ regex, content }) -> content

they are applied to file by

```js
  cliOf('My awesome task')
    ...
    .useHooks('path', (answers) => [
      useImport(`{${answers.camelCaseName}}`, `./${answers.name}.js`),
      usePath(`./${answers.name}.js`),
      useCustom({
        regex: /(\s*)(\/\*.*re-cli:use-module-name.*\*\/)/,
        content: `$1${moduleName}$1$2,`,
      }),
    ])
```

## 🚀 Setup

```js
npm install @re/cli
// or
yarn add @re/cli
```

it's possible to use it by global setup

```js
npm install @re/cli -g
// or
yarn add @re/cli -g
```

## Placing generators right way

The default agreements you should have `generators` folder at project root with your own generators are inside folders.

Other words, your generators should match the path: `generators/**/index*`

To override default behavior is simple (following same markup: https://storybook.js.org/docs/guides/guide-react/)

To do that, create a file at `.re-cli/config.js` with the following content:

```js
const { configure } = require("@re/cli");

function loadStories() {
  require("../generators/**/index*");
  // You can require as many stories as you need.
}

configure(loadStories, module);
```

But the full file path will resolved to `node.js` module and will execute it.

Let's say you wan't to have generators like stand alone module, to share it cross over the projects you have. Let's say it have name: `@re/xxx-generators`

The code markup can looks like:

- create a file at `index.js` with the following content:

  ```js
  const { configure } = require("re-cli");

  function loadStories() {
    require("./generators/**/*.gen.js");
  }

  configure(loadStories, module);
  ```

- make sure you have `package.json` main section like that:

  ```json
  "main": "index.js",
  ```

- the @re/cli will try to find the generators in that node module by using the `path` you prodive at `index.js`

## Tech notes

You can't use modern syntax different to Node.js you have installed. Cause we doesn't use `babel`, `webpack` inside.

`@re/cli` is written by using TS. So you will receive the extra IDE help by using TS for generators. But, you have to compile them. It should be simple for stand alone set of generators.

---

[MIT](./LICENSE)
