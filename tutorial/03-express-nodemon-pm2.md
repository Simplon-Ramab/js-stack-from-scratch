# 03 - Express, Nodemon, and PM2

Le code pour ce chapitre est disponible ici [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/03-express-nodemon-pm2).

Dans cette section, nous allons créer un serveur qui affichera notre web app. Nous configurerons également un mode développment et un mode production pour ce serveur.

## Express

> 💡 **[Express](http://expressjs.com/)** est de loin le framework le plus populaire pour Node. Il fournit une API minimaliste très simple, et ses fonctionnalités peuvent être étendues avec *middleware*.

Mettons en place un serveur Express minimal qui servira à afficher notre page HTML avec un peu de CSS.

- Supprimez tous les fichiers dans`src`

Créez les fichiers et dossiers suivants :

- un fichier `public/css/style.css` qui contient :

```css
body {
  width: 960px;
  margin: auto;
  font-family: sans-serif;
}

h1 {
  color: limegreen;
}
```

- un dossier `src/client/`

- un dossier `src/shared/`

C'est dans ce dossier que nous allons mettre le code JavaScript *universel/isomorphique* (des fichiers qui vont être utilisés par le client et par le serveur). Les routes sont un bon cas d'usage de code partagé, comme vous le constaterez plus tard dans ce tutoriel quand nous ferons des appel asynchrones. Ici nous avons simplement quelques constantes de configuration comme exemple.

- Créez un fichier `src/shared/config.js` qui contient :

```js
// @flow

export const WEB_PORT = process.env.PORT || 8000
export const STATIC_PATH = '/static'
export const APP_NAME = 'Hello App'
```

Si le processus Node utilisé pour faire fonctionner votre app a une variable d'environnement `process.env.PORT` de configurée (c'est le cas quand on va déployer sur Heroku par exemple), il l'utilisera comme port. S'il n'y en a pas, le port par défaut sera `8000`.

- Créez un fichier `src/shared/util.js` qui contient :

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const isProd = process.env.NODE_ENV === 'production'
```

C'est un simple utilitaire pour tester si nous sommes en production ou non. Le commentaire `// eslint-disable-next-line import/prefer-default-export` est présent car nous n'avons qu'un seul export nommé ici. Si vous avez plusieurs exports dans ce fichier, vous pouvez le retirer.

- Lancez `yarn add express compression`

`compression` est un middleware qui va activer la compression Gzip sur le serveur.

- Créez un fichier `src/server/index.js` qui contient :

```js
// @flow

import compression from 'compression'
import express from 'express'

import { APP_NAME, STATIC_PATH, WEB_PORT } from '../shared/config'
import { isProd } from '../shared/util'
import renderApp from './render-app'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

app.get('/', (req, res) => {
  res.send(renderApp(APP_NAME))
})

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' : '(development)'}.`)
})
```

Rien de bien méchant ici, c'est presque le 'Hello world' du tutoriel de Express avec quelques imports en plus. Nous utilisons 2 dossiers statiques différents ici : `dist` pour les fichiers générés, `public` pour ceux déclarés.

- Créez un fichier `src/server/render-app.js` qui contient :

```js
// @flow

import { STATIC_PATH } from '../shared/config'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <h1>${title}</h1>
  </body>
</html>
`

export default renderApp
```

Vous savez, votre habitude d'avoir un langage de template pour le back-end ? Et bien c'est assez obsolète, maintenant que Javascript supporte les template strings. Ici, nous créons une fonction qui prend un paramètre `title` et l'injecte dans les balises `title` et `h1` de la page, et qui retourne une chaîne de caractère complète en HTML. Nous utilisons aussi une constante `STATIC_PATH` comme chemin de base pour tous nos assets statiques

### Coloration syntaxique pour les template strings HTML sur Atom (optionnel)


Il est possible de faire fonctionner la coloration syntaxique pour du code HTML dans des template strings selon votre éditeur. Dans Atom, si vous préfixez votre template string avec le tag `html` (ou n'importe quel tag qui *fini* par `html`, code `jaimehtml`), il colorera automatiquement le contenu de cette string.

```js
import { html } from `common-tags`

const template = html`
<div>Wow, colors!</div>
`
```

Nous n'avons pas inclu ce petit tour dans le *boilerplate* de ce tutoriel puisqu'il semble fonctionner seulement pour Atom, ce qui est loin d'être idéal. Certains d'entre vous utilisent Atom pourraient trouver cela utile.

Enfin bref, revenous à nos moutons !

- Dans le fichier `package.json` changez votre script `start` comme ceci : `"start": "babel-node src/server",`

:checkered_flag: Lancez `yarn start`, et dans votre navigateur, rendez-vous sur `localhost:8000`. Si tout fonctionne comme prévu, vou devriez avoir une page blanche avec "Hello App" d'écrit dans l'onglet et dans le titre vert de la page.

**Remarque**: Quelques processus - typiquement, ceux qui attendent que des choses se passent, comme un serveur par exemple - vous éviterons d'entrer des commandes dans votre terminal jusqu'à ce qu'il ait fini. Pour interrompre ce genre de processus et avoir à nouveau accès à votre prompt, il faut faire **Ctrl+C**. Comme alternative, vous pouvez ouvrir un nouvel onglet dans votre terminal pour être capable d'entre d'autres commandes en même temps que votre processus tourne. Vous pouvez ausi faire tourner ces processus en arrière-plan mais c'est ça sort du cadre de notre tutoriel.

## Nodemon

> 💡 **[Nodemon](https://nodemon.io/)** est un utilitaire qui va automatiquement relancer votre serveur Node dès qu'un fichier est modifié dans le dossier. Nous allons utiliser Nodemon dès que nous sommes en mode **développement**.

- Lancez `yarn add --dev nodemon`

- Changez votre `scripts` de la manière suivante :

```json
"start": "yarn dev:start",
"dev:start": "nodemon --ignore lib --exec babel-node src/server",
```

`start` est maintenant un pointeur vers une autre tâche, `dev:start`. Cela nous donne une couche d'abstraction pour modifier ce qu'est la tâche par défaut.

Dans `dev:start`, le drapeau `--ignore lib` est pour *ne pas* redémarrer le serveur quand des changements arrivent dans le dossier `lib`. Vous n'avez pas encore ce dossier mais nous allons le générer dans la prochaine section de ce chapitre, donc tout va bientôt faire sens. Typiquement, Nodemon lance l'exécutable `node` Dans notre cas, puisqu'on utilise Babel, on peut dire à Nodemon d'utiliser l'exécutable `babel-node` à la place. De cet façon, il comprendra notre code ES6/Flow.



:checkered_flag: Lancez `yarn start` et ouvrez `localhost:8000` dans votre navigateur. Changez la constante `APP_NAME` dans `src/shared/config.js`, qui devrait déclencher le redémarrage de votre serveur dans le terminal. Rafraîchissez la page pour voir le titre modifié.  Notez que le redémarrage automatique du serveur est différent du *Hot Module Replacement* (quand les composants d'une page sont mis à jour en temps réel). Ici nous avons toujours besoin de rafraîchir la page manuellement, mais au moins nous n'avons pas à kill le processus et à le redémarrer manuellement pour voir les changements. Le Hot Module Replacement sera introduit dans le prochain chapitre.

## PM2

> 💡 **[PM2](http://pm2.keymetrics.io/)** est un process manager pour Node. Il garde tous nos processus vivants en production et offre des tonnes de fonctionnalités pour les gérer et suivre leurs performances.

Nous allons utiliser PM2 dès que nous sommes en **production**.

- Lancez `yarn add --dev pm2`

En production, vous voulez que votre serveur soit aussi performant que possible. `babel-node` déclenche tout le processus de transpilage de Babel pour vos fichiers à chaque exécution: vous ne voulez pas ça en production. Nous avons besoin que Babel fasse tout ce travail en amont, et que notre serveur rende nos bons vieux fichiers ES5 précompilés.

Une des principales fonctionnalités de Babel est de prendre un dossier de code ES6 (habituellement nommé `src`) et le transpiler en un dossier de code ES5 (habituellement nommé `lib`).

Ce dossier `lib` étant auto-généré, le nettoyer avant un nouveau *build* est une bonne pratique, puisqu'il peut contenir d'anciens fichiers dont on ne veut plus. `rimraf` est un package simple et efficace pour supprimer les fichiers sur différentes plateformes.

- Lancez `yarn add --dev rimraf`

Ajoutons la tâche `prod:build` à `scripts`:

```json
"prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
```

- Lancez `yarn prod:build`: il devrait générer un dossier `lib` qui contient le code transpilé, sauf pour les fichiers se terminant par `.test.js` (notez que les fichiers `.test.jsx` sont aussi ignorés par ce paramètre).

- Ajoutez `/lib/` à votre `.gitignore`

Une dernière chose: nous allons passer une variable d'environnement `NODE_ENV` à notre exécutable PM2. Avec Unix, vous pouvez faire ça en lançant `NODE_ENV=production pm2`, mais Windows utilise une syntaxe différente. Nous allons utiliser un package qui s'appelle `cross-env` pour que cette syntaxe soit également utilisable sous Windows.

- Lancez `yarn add --dev cross-env`

Modifions notre fichier `package.json` :

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon --ignore lib --exec babel-node src/server",
  "prod:build": "rimraf lib && babel src -d lib --ignore .test.js",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete server",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

:checkered_flag: Lancez `yarn prod:build`, puis `yarn prod:start`. PM2  devrait montrer un processus actif. Rendez vous sur `http://localhost:8000/`: vous devriez voir votre app. Votre terminal devrait afficher les logs, qui devraient être "Server running on port 8000 (production).". Notez qu'avec PM2, vos processus sont lancés en arrière-plan. Si vous faites Ctrl+C, cela tuera la commande `pm2 logs`, qui était la denière commande de notre chaîne `prod:start`, mais le serveur devrait toujours afficher la page. Si vous voulez arrêter le serveur, lancez `yarn prod:stop`.

Maintenant que nous avons une tâche `prod:build`, ça serait parfait si on pouvait s'assurer que tout fonctionne correctement avec de *push* du code sur le repo. Puisqu'il n'est pas vraiment nécessaire de le lancer à chaque commit, nous vous suggérons de l'ajouter à la tâche `prepush` :

```json
"prepush": "yarn test && yarn prod:build"
```

:checkered_flag: Lancez `yarn prepush` ou *pushez* vos fichiers pour déclencher le processus.

**Remarque**: Nous n'avons aucun test ici, donc Jest se plaindra un peu. Ignorez le pour l'instant.

Prochaine section: [04 - Webpack, React, HMR](04-webpack-react-hmr.md#readme)

Retour à la [section précédent](02-babel-es6-eslint-flow-jest-husky.md#readme) ou au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).
