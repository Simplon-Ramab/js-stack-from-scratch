# 01 - Node, Yarn et `package.json`

Le code de ce chapitre est disponible [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/01-node-yarn-package-json).

Dans cette section, nous allons configurer Node, Yarn, un fichier `package.json` basique, et tester un *package* (paquet :fr:).

## Node :computer:

> :bulb: **[Node.js](https://nodejs.org/)** est un environnement d'exécution JavaScript. On l'utilise principalement pour du développement back-end, mais aussi pour du scripting de façon générale. En développement front-end, Node peut être utilisé pour exécuter une série de tâches comme le linting, les tests ou encore la concaténation de fichiers.

Tout au long de ce tutoriel, nous allons utiliser Node pour pratiquement **tout**, vous allez donc en avoir besoin ! Rendez-vous sur la [page de téléchargement](https://nodejs.org/en/download/current/) pour les installeurs **macOS/OSX** ou **Windows**, ou la [page d'installation via gestionnaire de paquets](https://nodejs.org/en/download/package-manager/) pour les distributions Linux.

Par exemple, sur **Ubuntu / Debian**, vous exécuterez les commandes suivantes pour lancer Node :

```sh
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Vous allez avoir besoin de n'importe quelle version de Node qui soit supérieure à la 6.5.0.

## Node Version Management Tools :wrench:

Si vous avez besoin de flexibilité et de pouvoir utiliser plusieurs versions de Node différentes, jetez un oeil à [NVM](https://github.com/creationix/nvm) ou [tj/n](https://github.com/tj/n).

## NPM :bear:

NPM est le package manager (*gestionnaire de paquets* :fr:) par défaut pour Node. Il est installé automatiquement, en même temps que Node. Les gestionnaires de packages sont utilisés pour installer et gérer des packages (des modules de code que vous ou quelqu'un d'autre a écrit). Nous allons utiliser beaucoup de packages dans ce tutoriel mais, au lieu de Node, nous utiliserons un autre package manager: Yarn.

## Yarn :cat:

> :bulb: **[Yarn](https://yarnpkg.com/)** est un package manager beaucoup plus rapide que NPM qui offre le support hors-ligne et récupère les dépendances [de façon plus prédictible](https://yarnpkg.com/en/docs/yarn-lock).

Depuis qu'il est [sorti](https://code.facebook.com/posts/1840075619545360) en octobre 2016, Yarn a rapidement été adopté. Il pourrait d'ailleurs bientôt devenir le package manager plébiscité par la communauté JavaScript. Si vous voulez rester avec NPM, vous pouvez tout simplement remplacer toutes les commandes `yarn add` et `yarn add --dev` de ce tutoriel par `npm install --save` et `npm install --save-dev`.

Installez Yarn en suivant les [instructions](https://yarnpkg.com/en/docs/install) pour votre système d'exploitation. Nous vous recommandons d'utiliser le **script d'installation** de l'onglet *Alternatives* si vous êtes sous macOS ou Unix, pour [éviter](https://github.com/yarnpkg/yarn/issues/1505) d'avoir à recourir à d'autres gestionnaires de packages :

```
sh
curl -o- -L https://yarnpkg.com/install.sh | bash
```

## `package.json` :package:

> :bulb: **[package.json](https://yarnpkg.com/en/docs/package-json)** est le fichier utilisé pour décrire et configurer votre projet JavaScript. Il contient toutes les informations générales (le nom de votre projet, sa version, les différents contributeurs, sa licence, etc), les configuration d'options pour les outils que vous utilisez, et même une section pour lancer des *tâches*.

- Créez un nouveau dossier dans lequel vous allez travailler et rendez-vous dedans (`cd`).
- Lancez `yarn init` et répondez aux questions (utilisez `yarn init -y` pour passer les questions). Cela générera un fichier `package.json` automatiquement.

Voici le `package.json` basique que nous utiliserons dans ce tutoriel :

```json
{
  "name": "votre-projet",
  "version": "1.0.0",
  "license": "MIT"
}
```

## Hello World :wave:

- Créez un fichier `index.js` qui contient `console.log('Hello world')`

:checkered_flag: Lancez `node .` dans ce dossier (`index.js` est le fichier par défaut que va regarder Node dans ce dossier Node). Votre console devrait afficher "Hello world".

**Note**: Vous avez remarqué l'emoji drapeau de course :checkered_flag: ? Nous l'utiliserons chaque fois que vous attendrez un **checkpoint**. Nous allons parfois effectuer plusieurs gros changement d'un coup, et votre code pourrait ne pas fonctionner jusqu'à ce que vous atteignez le checkpoint suivant.

##  Script `start` :rocket:

Utiliser `node .` pour exécuter notre programme est un peu bas-niveau. À la place, nous allons utiliser un script NPM/Yarn pour déclencher l'exécution de ce code. Cela nous donnera un bon niveau d'abstraction pour pouvoir toujours utiliser `yarn start`, même quand notre programme deviendra plus complexe.

- Dans le fichier `package.json`, ajoutez un objet `scripts` comme ceci :

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT",
  "scripts": {
    "start": "node ."
  }
}
```

`start` est le nom que nous allons donner à la *tâche* qui va lancer notre programme. Nous allons créer de nombreuses tâches différentes dans cet objet `scripts` tout au long de ce tutoriel. `start` est le nom classique qu'on donne à la tâche par défaut d'une application. Parmi les autres noms de tâches standard, on retrouve aussi `stop` et `test`.

`package.json` doit être un fichier JSON valide: il ne doit donc pas avoir de virgule en trop. Faites donc bien attention quand vous éditez votre fichier `package.json` manuellement.

:checkered_flag: Lancez `yarn start`. Votre console devrait afficher `Hello world`.

## Git et `.gitignore` :octocat:

- Initialisez un repo (*repository*, ou dépôt :fr:) Git avec `git init`

- Créez un fichier `.gitignore` et ajoutez-y les lignes suivantes :

```gitignore
.DS_Store
/*.log
```

Les fichiers `.DS_Store` sont auto-générés par macOS. Vous ne devez jamais les avoir dans votre repo.

`npm-debug.log` et `yarn-error.log` sont des fichiers qui sont créés quand votre package manager rencontre une erreur. On ne veut pas non plus versionner cela dans notre repo.

## Installer et utiliser un package :wrench:

Dans cette section, nous allons installer et utiliser un package. Un "package", c'est un morceau de code que quelqu'un d'autre a écrit qu'on peut utiliser dans notre propre code. Ici, nous allons essayer un package qui va nous aider à manipuler les couleurs.

- Installez le package créé par la communauté qui s'appelle `color` en lançant `yarn add color`

Ouvrez votre `package.json`: Yarn a automatiquement ajouté `color` dans `dependencies`!

Un dossier `node_modules` a aussi été créé: c'est ici qu'est placé le package qu'on vient d'installer.

- ajouter le dossier `node_modules/` à votre `.gitignore`

Vous avez remarqué ? Le fichier `yarn.lock` a été généré par Yarn. Vous devez *commit* ce fichier dans votre repo: il permet de s'assurer que tout le monde dans votre équipe va utiliser les mêmes versions de packages. Si vous utilisez NPM au lieu de Yarn, l'équivalent de ce fichier est le *shrinkwrap*.

- Ecrivez les lignes suivantes dans votre fichier `index.js` :

```js
const color = require('color')

const redHexa = color({ r: 255, g: 0, b: 0 }).hex()

console.log(redHexa)
```

:checkered_flag: Lancez `yarn start`. Votre console devrait afficher `#FF0000`.

Félicitations, vous avez installé et utilisé un package ! :tada:

`color` a juste été utilisé dans cette section pour vous apprendre comment utiliser un package simple. Nous n'en avons plus besoin, vous pouvez donc le désinstaller: 

- Lancez `yarn remove color`

## Deux types de dépendances

Il y a deux types de *package dependencies* (dépendances de paquets :fr:), `"dependencies"` et `"devDependencies"`:

**Les Dependencies** (dépendances :fr:) sont des bibliothèques dont vous avez besoin pour que votre application fonctionne (ex: React, Redux, Lodash, jQuery, Vue etc). On les installe avec `yarn add [package]`.

**Les Dev Dependencies** (dépendances de développement :fr:) sont des bibliothèques utilisées pendant le développement de l'application ou pendant son *build* (ex: Webpack, SASS, linters, frameworks de test, etc). On les installe avec `yarn add --dev [package]`.

Section suivante: [02 - Babel, ES6, ESLint, Flow, Jest, Husky](02-babel-es6-eslint-flow-jest-husky.md#readme)

Retour au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).
