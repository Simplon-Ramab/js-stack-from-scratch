# 04 - Webpack, React et Hot Module Replacement

Le code pour ce chapitre est disponible [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr).

## Webpack

> :bulb: **[Webpack](https://webpack.js.org/)** est un *module bundler* (*empaqueteur de module* :fr:). Il prend nos différents fichiers sources, les traitent et les fusionne (habituellement) en un seul fichier JavScript appelé "bundle", qui est le seul fichier qui sera exécuté par le client.

Créons un *hello world* basique et *empaquetons-le* avec Webpack.

- Dans le fichier `src/shared/config.js`, ajoutons les constantes suivantes :

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- Créez un fichier `src/client/index.js` contenant :

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

Si vous voulez utiliser les fonctionnalités ES les plus récentes dans votre code client comme les `Promise`, vous allez avoir besoin d'inclure le [Polyfill Babel](https://babeljs.io/docs/usage/polyfill/) avant tout autre chose dans votre bundle.

- Lancez `yarn add babel-polyfill`

Si vous lancez ESLint sur ce fichier, il se plaindra que `document` est *undefined* (non-défini).

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

Ce fichier est utilisé pour décrire comment notre bundle devrait être assemblé. `entry` est le point d'entrée de notre app, `output.filename` est le nom du bundle à générer, `output.path` et `output.publicPath` décrivent le dossier de destination et l'URL.

On place ensuite le bundle dans un fichier `dist`, qui va contenir les choses qui seront générées automatiquement (pas comme le CSS déclaratif qu'on a créé plus tôt et qui, lui, est dans `public`). `module.rules` est l'endroit où vous dites à Webpack d'appliquer des traitements à certains types de fichiers. Ici, on lui dit que nous voulons que tous nos fichiers `.js` et `.jsx` (pour React), exceptés ceux dans les `node_modules` doivent passer dans quelque chose qui s'appelle `babel-loader`. On veut aussi que ces deux extensions soient utilisées pour `resolve` les modules quand on les `import`. Pour finir, on déclare un port pour le Webpack Dev Server.

**Remarque**: L'extension `.babel.js` est une fonctionnalité Webpack pour appliquer nos transformations Babel à ce fichier de configuration.

`babel-loader` est un plugin pour Webpack qui transpile votre code tout comme nous l'avons fait depuis le début de ce tutoriel. La seule différence est que cette fois, le code finira dans le navigateur et non plus dans votre serveur.

- Lancez `yarn add --dev webpack webpack-dev-server babel-core babel-loader`

`babel-core` est une dépendance paire de `babel-loader`, donc nous l'installerons aussi.

- Ajoutez `/dist/` à votre `.gitignore`

### Modifications de tâches

En mode développement, nous allons utiliser `webpack-dev-server` pour prendre avantage du Hot Module Reloading (qu'on verra plus tard dans ce chapitre), et en production on utilisera simplement `webpack` pour générer nos bundles. Dans les deux cas, le flag `--progress` est utile pour afficher des informations supplémentaires quand Webpack compile vos fichiers. En production, on passera aussi le flag `-p` à Webpack pour qu'il minifie notre code, et qu'il passe la variable d'environnement `NODE_ENV` à `production`.

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

Selon l'environnement dans lequel nous sommes, nous inclurons soit le bundle Webpack Dev Server, soit celui de production. Notez que le chemin pour le bundle Webpack Dev Server est *virtuel*, `dist/js/bundle.js` n'est pas vraiment lu de votre disque dur en mode développement. Il est également nécessaire de donner à Webpack Dev Server un port différent de votre port web principal.

- Pour finir, dans `src/server/index.js`, modifiez le message de votre `console.log` comme ceci :

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

Cela donnera aux autres développeurs une indication sur ce qu'il faut faire s'ils essaient juste de lancer `yarn start` sans Webpack Dev Server.

Ok, ça fait beaucoup de changement, voyons si tout marche comme prévu :

:checkered_flag: Lancez `yarn start` dans votre terminal. Ouvrez en un autre dans une nouvelle fenêtre ou un nouvel onglet, et lancez-y `yarn dev:wds`. Une fois que Webpack Dev Server a fini de générez le bundle et son sourcemaps (les deux devraient être des fichiers d'environ ~600ko) et que les 2 processus sont tranquilles dans votre terminal, ouvrez `http://localhost:8000/` dans votre navigateur. Vous devriez voir "Hello Webpack!". Ouvrez la console de votre navigateur et sous l'onglet Source, vérifiez quels fichiers sont inclus. Vous devriez seulement voir `static/css/style.css` sous `localhost:8000/`, et tous vos fichiers sources ES6 sous `webpack://./src`. Cela signifie que les sourcemaps fonctionnent. Dans votre éditeur de texte, dans `src/client/index.js`, changez `Hello Webpack!` en n'importe quelle autre chaîne de caractère. Dès que vous aurez sauvegardé votre fichier, dans votre terminal, Webpack Dev Server devrait générer un nouveau bundle, et l'onglet de votre navigateur, se rafraîchir automatiquement.

- Tuez le processus précédent dans votre terminal avec Ctrl+C, puis lancez `yarn prod:build`, et ensuite `yarn prod:start`. Ouvrez `http://localhost:8000/` dans votre navigateur. Vous devriez toujours voir "Hello Webpack!". Dans l'onglet Source de votre navigateur, vous devriez voir `static/js/bundle.js` sous `localhost:8000/`, mais pas de sources `webpack://`. Cliquez sur `bundle.js` pour être sûr que tout est bien minifié. Lancez `yarn prod:stop`.

Bien joué, c'était assez intense. Vous avez bien mérité une pause :clap: ! La prochaine section est plus simple.

**Note**: Nous vous recommandons d'avoir au moins 3 terminaux ouverts: un pour le serveur Express, un autre pour Webpack Dev Server, et enfin un pour Git, les tests, et les commandes générales comme l'installation de package avec `yarn`. Idéalement, vous devriez scindez la fenêtre de vos terminaux en plusieurs panels afin de tous les voir d'un coup.

## React

> :bulb: **[React](https://facebook.github.io/react/)** est une bibliothèque créée par Facebook qui permet de mettre en place des interfaces utilisateur. Il utilise la syntaxe **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** pour représenter des éléments HTML et des composants tout en exploitant la puissance de JavaScript.

Dans cette section nous allons "rendre" du texte en utilisant React et JSX.

Tout d'abord, installons React et ReactDOM :

- Lancez `yarn add react react-dom`

Renommez votre fichier `src/client/index.js` en `src/client/index.jsx` et écrivez-y un peu de React :

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- Créez un fichier `src/client/app.jsx` contenant :

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

Puisqu'on utilise la syntaxe JSX ici, nous devons expliquer à Babel qu'il a besoin de transformer ce JSX avec la préconfiguration `babel-preset-react`. Et tant qu'on y est, nous allons aussi ajouter le plugin Babel `flow-react-proptypes` qui va automatiquement générer les PropTypes depuis nos annotations Flow pour nos composants React.

- Lancez `yarn add --dev babel-preset-react babel-plugin-flow-react-proptypes` et éditez votre fichier `.babelrc` de cette façon :

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

:checkered_flag: Lancez `yarn start` et `yarn dev:wds`. Dans votre navigateur, rendez vous sur `http://localhost:8000`. Vous devriez voir "Hello React!".

Maintenant essayez de changer le texte du fichier `src/client/app.jsx` en quelque chose d'autre. Webpack Dev Server devrait recharger la page automatiquement, ce qui est plutôt cool, mais on va rendre ça encore mieux. :ok_hand:

## Hot Module Replacement

> :bulb: **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) est une fonctionnalité Webpack puissante qui remplace automatiquement un module sans avoir à recharger toute la page.

Pour que HMR fonctionne avec React, nous allons devoir ajuster quelques petites choses :

- Lancez `yarn add react-hot-loader@next`

- Éditez votre fichier `webpack.config.babel.js` de la façon suivante :

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

La partie `headers` autorise le *Cross-Origin Resource Sharing* qui est nécessaire au bon fonctionnement des Hot Module Replacement.

- Éditez votre fichier `src/client/index.jsx` :

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

Nous avons besoin de créer à notre `App` un enfant de l'`AppContainer` de `react-hot-loader`, et nous allons avoir besoin de `require` la prochaine version de notre `App` lors du *hot reloading*. Pour faire en sorte que ce processus soit propre et respecte bien les principes DRY (don't repeat yourself - ne te répète pas :fr:), nous créons une petite fonction `wrapApp` que nous utilisons aux endroits où il a besoin d'afficher `App`. Vous êtes libre de bouger `eslint-disable global-require` en haut du fichier pour que ce soit plus lisible.

:checkered_flag: S'il était toujours en train de tourner, redémarrez votre processus `yarn dev:wds`. Ouvrez `localhost:8000` dans votre navigateur. Dans l'onglet console de l'inspecteur du navigateur, vous devriez voir des logs qui concerne HMR. Allez-y, changez quelque chose dans `src/client/app.jsx` et sauvegardez. Vos changements ont lieu dans le navigateur en quelques secondes sans que la page ait à se recharger entièrement ! :+1:

Prochaine section: [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

Retourner à la [section précédente](03-express-nodemon-pm2.md#readme) ou au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).
