# 04 - Webpack, React, and Hot Module Replacement

Le code pour ce chapitre est disponible [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr).

## Webpack

> 💡 **[Webpack](https://webpack.js.org/)** est un *module bundler* (*empaqueteur de module* :fr:). Il prend nos différents fichiers sources, les traite et les fusionne (habituellement) en un seul fichier JavScript appelé "bundle", qui est le seul fichier qui sera exécuté par le client.

Créons un *hello world* basique et *empaquetons-le* avec Webpack.

- Dans le fichier `src/shared/config.js`, ajoutons les constantes suivantes :

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- Create an `src/client/index.js` file containing:

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

Si vous voulez utiliser les fonctionnalités ES les plus récentes dans votre code client comme les `Promise`, vous allez avoir besoin d'inclure le [Polyfill Babel](https://babeljs.io/docs/usage/polyfill/)  avant tout autre chose dans votre bundle.

- Lencez `yarn add babel-polyfill`

Si vous lancez ESLint sur ce fichier, il se plaindra que `document` *undefined* (non-défini).

- Ajoutez les lignes suivantes à `env` dans votre fichier `.eslintrc.json`. Cela vous permettra d'utiliser `window` et `document`:

```json
"env": {
  "browser": true,
  "jest": true
}
```

Bien, maintenant, nous avons besoin d'empaqueter cette app client ES6 dans un bundle ES5.

- Créez un fichier`webpack.config.babel.js` contenant :

```js
// @flow

import path from 'path'

import { WDS_PORT } from './src/shared/config'
import { isProd } from './src/shared/util'

export default {
  entry: [
    './src/client',
  ],
  output: {
    filename: 'js/bundle.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: isProd ? '/static/' : `http://localhost:${WDS_PORT}/dist/`,
  },
  module: {
    rules: [
      { test: /\.(js|jsx)$/, use: 'babel-loader', exclude: /node_modules/ },
    ],
  },
  devtool: isProd ? false : 'source-map',
  resolve: {
    extensions: ['.js', '.jsx'],
  },
  devServer: {
    port: WDS_PORT,
  },
}
```

Ce fichier est utilisé pour décrire comment notre bundle devrait être assemblé. `entry` est le point d'entrée de notre app,  `output.filename` est le nom du bundle à générer, `output.path` et `output.publicPath` décrivent le dossier de destination et l'URL.

On place ensuite le bundle dans un fichier `dist`, qui va contenir les choses qui seront générés automatiquement (pas comme le CSS déclaratif qu'on a créé plus tôt et qui, lui, est dans `public`). `module.rules` est l'endroit où vous dites à Webpack d'appliquer des traitements à certains types de fichiers. Ici, on lui dit que nous voulons que tous nos fichiers `.js` et `.jsx` (pour React), exceptés ceux dans les `node_modules` doivent passer dans quelque chose qui s'appelle `babel-loader`. On veut aussi que ces deux extensions soient utilisées pour `resolve` les modules quand on les `import`. Pour finir, on déclare un port pour le Webpack Dev Server.

**Remarque**: L'extension `.babel.js` est une fonctionnalité Webpack pour appliquer nos transformations Babel à ce fichier de configuration.

`babel-loader` est un plugin pour Webpack qui transpile votre code tout comme nous l'avons fait depuis le début de ce tutoriel. La seule différence est que cette fois, le code finira dans le navigateur et non plus dans votre serveur.

- Lancez `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

`babel-core` est une dépendance paire de `babel-loader`, donc nous l'installerons aussi.

- Ajoutez `/dist/` à votre `.gitignore`

### Modifications de tâches

En mode développement, nous allons utiliser  `webpack-dev-server` pour prendre avante du Hot Module Reloading (qu'on verra plus tard dans ce chapitre), et en production on utilisera simplement `webpack` pour générer nos bundles. Dans les deux cas, le flag `--progress` est utile pour afficher des informations supplémentaires quand Webpack compile vos fichiers. En production, on passera aussi le flag `-p` à Webpack pour minifir notre code, et la variable d'environnement `NODE_ENV` passée à `production`.

Modifions notre `scripts` pour implémenter tout ça et profitons-en pour modifier nos autres tâches aussi :

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon -e js,jsx --ignore lib --ignore dist --exec babel-node src/server",
  "dev:wds": "webpack-dev-server --progress",
  "prod:build": "rimraf lib dist && babel src -d lib --ignore .test.js && cross-env NODE_ENV=production webpack -p --progress",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete server",
  "lint": "eslint src webpack.config.babel.js --ext .js,.jsx",
  "test": "yarn lint && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test && yarn prod:build"
},
```

Dans `dev:start` on déclare explicitement les extensions de fichiers à suivre : `.js` et `.jsx`. Ajoutez `dist` dans les dossiers ignorés.

Nous avons créé une tâche `lint` séparée et avons ajouté `webpack.config.babel.js` aux fichiers à *linter*.

- Ensuite, créons le conteneur pour notre app dans `src/server/render-app.js`, et incluons le bundle qui sera généré :

```js
// @flow

import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <div class="${APP_CONTAINER_CLASS}"></div>
    <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
  </body>
</html>
`

export default renderApp
```

Selon l'environnement dans lequel nous sommes, nous inclurons soit le bundle Webpack Dev Server, soit celui de production. Notez que le chemin pour le bundle Webpack Dev Server est *virtuel*, `dist/js/bundle.js` n'est pas vraiment lu de votre disque dur en mode développement. Il est également nécessaire de donner à Webpack Dev Server un port différent que votre port web principal.

- Pour finir, dans `src/server/index.js`, modifiez le message de votre `console.log` comme ceci :

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

Cela donnera aux autres développeurs une indications sur ce qu'il faut faire s'ils essaient juste de lancer `yarn start` sans Webpack Dev Server.

Ok, ça fait beaucoup de changement, voyons si tout marche comme prévu :

:checkered_flag: Lancez `yarn start` dans votre terminal. Ouvrez en un autre dans une nouvelle fenêtre ou un nouvel onglet, et lancez-y `yarn dev:wds`. Une fois que Webpack Dev Server a fini de générez le bundle et son sourcemaps (les deux devraient être des fichiers d'environ ~600ko) et que les 2 processus sont tranquilles dans votre terminal, ouvrez `http://localhost:8000/` dans votre navigateur. Vous devriez voir "Hello Webpack!". Ouvrez la console de votre navigateur  et sous l'onglet Source tab, vérifiez quels fichiers sont inclus. Vous devriez seulement voir sous

You should only see `static/css/style.css` under `localhost:8000/`, and have all your ES6 source files under `webpack://./src`. That means sourcemaps are working. In your editor, in `src/client/index.js`, try changing `Hello Webpack!` into any other string. As you save the file, Webpack Dev Server in your terminal should generate a new bundle and the Chrome tab should reload automatically.

- Kill the previous processes in your terminals with Ctrl+C, then run `yarn prod:build`, and then `yarn prod:start`. Open `http://localhost:8000/` and you should still see "Hello Webpack!". In the Source tab of the Chrome console, you should this time find `static/js/bundle.js` under `localhost:8000/`, but no `webpack://` sources. Click on `bundle.js` to make sure it is minified. Run `yarn prod:stop`.

Good job, I know this was quite dense. You deserve a break! The next section is easier.

**Note**: I would recommend to have at least 3 terminals open, one for your Express server, one for the Webpack Dev Server, and one for Git, tests, and general commands like installing packages with `yarn`. Ideally, you should split your terminal screen in multiple panes to see them all.

## React

> 💡 **[React](https://facebook.github.io/react/)** is a library for building user interfaces by Facebook. It uses the **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** syntax to represent HTML elements and components while leveraging the power of JavaScript.

In this section we are going to render some text using React and JSX.

First, let's install React and ReactDOM:

- Run `yarn add react react-dom`

Rename your `src/client/index.js` file into `src/client/index.jsx` and write some React code in it:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- Create a `src/client/app.jsx` file containing:

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

Since we use the JSX syntax here, we have to tell Babel that it needs to transform it with the `babel-preset-react` preset. And while we're at it, we're also going to add a Babel plugin called `flow-react-proptypes` which automatically generates PropTypes from Flow annotations for your React components.

- Run `yarn add --dev babel-preset-react babel-plugin-flow-react-proptypes` and edit your `.babelrc` file like so:

```json
{
  "presets": [
    "env",
    "flow",
    "react"
  ],
  "plugins": [
    "flow-react-proptypes"
  ]
}
```

:checkered_flag: Run `yarn start` and `yarn dev:wds` and hit `http://localhost:8000`. You should see "Hello React!".

Now try changing the text in `src/client/app.jsx` to something else. Webpack Dev Server should reload the page automatically, which is pretty neat, but we are going to make it even better.

## Hot Module Replacement

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) is a powerful Webpack feature to replace a module on the fly without reloading the entire page.

To make HMR work with React, we are going to need to tweak a few things.

- Run `yarn add react-hot-loader@next`

- Edit your `webpack.config.babel.js` like so:

```js
import webpack from 'webpack'
// [...]
entry: [
  'react-hot-loader/patch',
  './src/client',
],
// [...]
devServer: {
  port: WDS_PORT,
  hot: true,
  headers: {
    'Access-Control-Allow-Origin': '*',
  },
},
plugins: [
  new webpack.optimize.OccurrenceOrderPlugin(),
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NamedModulesPlugin(),
  new webpack.NoEmitOnErrorsPlugin(),
],
```

The `headers` bit is to allow Cross-Origin Resource Sharing which is necessary for HMR.

- Edit your `src/client/index.jsx` file:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = AppComponent =>
  <AppContainer>
    <AppComponent />
  </AppContainer>

ReactDOM.render(wrapApp(App), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp), rootEl)
  })
}
```

We need to make our `App` a child of `react-hot-loader`'s `AppContainer`, and we need to `require` the next version of our `App` when hot-reloading. To make this  process clean and DRY, we create a little `wrapApp` function that we use in both places it needs to render `App`. Feel free to move the `eslint-disable global-require` to the top of the file to make this more readable.

:checkered_flag: Restart your `yarn dev:wds` process if it was still running. Open `localhost:8000`. In the Console tab, you should see some logs about HMR. Go ahead and change something in `src/client/app.jsx` and your changes should be reflected in your browser after a few seconds, without any full-page reload!

Next section: [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

Back to the [previous section](03-express-nodemon-pm2.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
