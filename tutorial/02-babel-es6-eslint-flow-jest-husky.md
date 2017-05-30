# 02 - Babel, ES6, ESLint, Flow, Jest, and Husky

Le code de ce chapitre est disponible [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/02-babel-es6-eslint-flow-jest-husky).

Maintenant nous allons utiliser un peu de syntaxe ES6, qui est une grande amélioration pour la "vieille" syntaxe ES5.  Tous les navigateurs et environnements JS comprennent l'ES5, **mais pas l'ES6**. C'est là qu'un certain outil appelé Babel vient à notre secours !

## Babel

> 💡 **[Babel](https://babeljs.io/)** est un compilateur qui transforme le code ES6 (et d'autres trucs comme la syntaxe JSX de React par exemple en code ES5. Babel est très modulaire et peut être utilisé dans une tonne d'[environnements différents](https://babeljs.io/docs/setup/). C'est de loin le compilateur ES5 préféré de la communauté React.

- Déplacez votre fichier `index.js` dans un nouveau dossier appelé `src` (src pour *source*). C'est ici que nous allons écrire tout notre code ES6. Retirez tout le code précédent relatif à `color` et remplacez le par un simple :

```js
const str = 'ES6'
console.log(`Hello ${str}`)
```

Ici, nous utilisons une *template string* (chaîne de template :fr:), une fonctionnalité ES6 qui nous laisse injecter des variables directement dans une chaîne de caractères sans concaténation en utilisant `${}`. Notez que les template strings sont créées en utilisant des **backquotes** (`...`).

- Lancez `yarn add --dev babel-cli` pour installer la CLI (*Command Line Interface*, interface en ligne de commande :fr:) de Babel.

La CLI Babel est "livrée" avec [deux exécutables](https://babeljs.io/docs/usage/cli/): `babel`, qui compile nos fichiers ES6 en nouveaux, et `babel-node`, que vous pouvez utiliser pour remplacer la commande `node` et exécuter vos fichiers ES6 directement. `babel-node` est super pour le développement mais reste très lourd et ne doit pas être utilisé en production. Dans ce chapitre, nous allons utiliser `babel-node` pour mettre en place notre environnement de développement, et dans le prochain, nous utiliserons `babel` pour *build* nos fichiers ES5 pour la production.

- Dans votre `package.json`, dans le script `start`, remplacez `node .` par `babel-node src` (`index.js` est le fichier par défaut que va chercher Node, c'est pourquoi on peut omettre de préciser `index.js`).

Maintenant, si vous essayez de lancer `yarn start`, la console devrait afficher ce qu'il faut mais en fait, Babel ne fait rien du tout. C'est parce qu'on a pas indiqué quelles informations on souhaite appliquer. La seule raison pour laquelle les bonnes informations sont affichées... c'est parce que Node comprend nativement ES6, sans l'aide de Babel ! Pour autant, certains navigateurs ou des versions plus anciennes de Node ne réussiraient pas à en faire de même.

- Lancez `yarn add --dev babel-preset-env` pour installer un package préconfiguré qui s'appelle `env`. Il contient des configurations pour les fonctionnalités ECMAScript les plus récentes supportées par Babel.

- Créez un fichier `.babelrc` à la racine de votre projet. Il s'agit d'un fichier JSON pour votre configuration Babel. Ecrivez les lignes suivantes pour que Babel utilise la préconfiguration `env` :

```json
{
  "presets": [
    "env"
  ]
}
```

:checkered_flag: `yarn start` devrait toujours marcher, mais maintenant il fait quelque chose. Mais vrai dire on ne peut pas vraiment savoir, étant donné qu'on utilise `babel-node` pour interpréter de l'ES6 directement. Bientôt, vous aurez la preuve que notre code ES6 est bien transformé quand vous atteindrez la section [syntaxe des modules ES6](#the-es6-modules-syntax) de ce chapitre.

## ES6

> 💡 **[ES6](http://es6-features.org/)**: L'amélioration la plus significative du langage JavaScript. Il y a trop de fonctionnalités ES6 pour qu'on les liste toutes ici, mais du code ES6 typique utilise des classes avec `class`, `const` et `let`, les template strings, et les *arrow functions* (fonction flèche :fr:) (`(text) => { console.log(text) }`).

### Créer une classe en ES6

- Créez un nouveau fichier, `src/dog.js`, contenant la classe ES6 suivante :

```js
class Dog {
  constructor(name) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

module.exports = Dog
```

Si vous avez déjà fait de la programmation orientée objet avant, cela ne devrait pas vous choquer. Pourtant, c'est quelque chose d'assez récent en JavaScript. La classe est offerte au monde extérieur via l'affectation `module.exports`.

Dans `src/index.js`, on écrit les lignes suivantes :

```js
const Dog = require('./dog')

const toby = new Dog('Toby')

console.log(toby.bark())
```

Comme vous pouvez le voir, à la différence du package `color` utilisé précédemment, lorsqu'on *require* (appelle :fr:) l'un de nos fichiers, on utilise `./` dans le `require()`.

:checkered_flag: Lancez `yarn start`. Vous devriez lire "Wah wah, I am Toby" dans votre console. :dog:

### La syntaxe des modules ES6

Ici, on remplace tout simplement `const Dog = require('./dog')` par `import Dog from './dog'`, qui est la nouvelle syntaxe de modules ES6 (opposé à la syntaxe des modules "CommonJS"). Actuellement, elle n'est pas supporté nativement par NodeJS : ça sera donc une preuve que notre Babel transforme nos fichiers ES6 correctement.
Dans `dog.js`, on remplace aussi `module.exports = Dog` par `export default Dog`

:checkered_flag: `yarn start` devrait toujours afficher "Wah wah, I am Toby". :dog:

## ESLint

> **Un linter** est un outil visant à améliorer la qualité et la consistance du code.

> 💡 **[ESLint](http://eslint.org)** est un linter de choix pour du code ES6. Un linter va nous donner des informations à propos du format du code, qui va renforcer sa consistance et sa cohérence. C'est aussi un bon moyen d'apprendre des choses sur JavaScript en faisant des erreurs que ESLint va détecter.

ESLint travaille avec des *règles*, et il y en a [beaucoup](http://eslint.org/docs/rules/) (:uk:). Au lieu de configurer les règles que nous voulons suivre pour notre code par nous-même, Nous allons utiliser une configuration faite par Airbnb. Cette configuration utilise quelques plugins que nous allons aussi avoir besoin d'installer.

Jetez un oeil aux [instructions](https://www.npmjs.com/package/eslint-config-airbnb) les plus récentes d'Airbnb pour installer le package de configuration et toutes ses dépendances. Le 2017-02-03, il est recommandé d'utiliser les commandes suivantes dans votre terminal :

```sh
npm info eslint-config-airbnb@latest peerDependencies --json | command sed 's/[\{\},]//g ; s/: /@/g' | xargs yarn add --dev eslint-config-airbnb@latest
```

Ça devrait installer tout ce dont vous avez besoin et ajouter automatiquement `eslint-config-airbnb`, `eslint-plugin-import`, `eslint-plugin-jsx-a11y`, et `eslint-plugin-react` à votre fichier `package.json`.

**Remarque**: Nous avons remplacé `npm install` par `yarn add` dans cette. **De plus, elle ne fonctionnera pas sous Windows**, alors regardez le fichier `package.json` de ce repo installez juste les dépendances liées à ESLint manuellement avec `yarn add --dev packagename@^#.#.#`; `#.#.#` étant la version donnée par `package.json` pour chaque package.

- Créer un fichier `.eslintrc.json` à la racine de votre projet, comme nous l'avons fait pour Babel. Ecrivez-y les lignes suivantes :

```json
{
  "extends": "airbnb"
}
```

Nous allons créer un nouveau script NPM/Yarn pour lancer ESLint. Maintenant, installons le package `eslint` pour pouvoir utiliser la `eslint` :

- Lancez `yarn add --dev eslint`

Modifiez l'objet `scripts` de votre `package.json` pour inclure la nouvelle tâche `test` :

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src"
},
```

Ici, on indique à ESLint qu'on veut qu'il *lint* tous nos fichiers JavaScript  situés sous le dossier `src`.

On utilisera cette tâche `test` standard pour lancer une série de commandes qui validerons notre code, que ce soit du *linting*, du *type checking*, ou des tests unitaires.

- Lancez `yarn test`. Vous devriez voir une série d'erreurs pour des point-virgule manquants et un warning pour avoir utilisé `console.log()` dans `index.js`. Ajoutez `/* eslint-disable no-console */` en haut de votre fichier `index.js`: cela va nous permettre d'utiliser `console` dans ce fichier.

**Note**: Si vous êtes sous, soyez sûr de configurer votre éditeur de texte et Git pour qu'il utilise les fins de ligne LF Unix LF et pas Windows CRLF. Si votre projet est seulement utilisé dans des environnements Windows, vous pouvez ajouter `"linebreak-style": [2, "windows"]` dans le tableau de règles ESLint' (voir l'exemple ci-dessous) pour imposer CRLF à la place.

### Point-virgule

Ok, c'est probablement le débat le plus chaud dans la communauté JavaScript mais parlons-en un peu. JavaScript a ce truc qui s'appelle l'insertion automatique de point-virgule, qui nous permet d'écrire notre code avec ou sans point-virgule. Sur ce sujet, rien de bon ou mauvais: il s'agit juste d'une préférence personnelle. Si vous aimez la syntaxe de Python, Ruby ou Scala, alors il est très probable que vous préfériez ne pas mettre de point virgule. Si vous préférez la syntaxe de Java, C#, ou PHP, c'est l'inverse.

La plupart des gens écrivent du JavaScript avec des point-virgules par habitude. C'était mon cas jusqu'à ce que j'y aille en mode no point-virgule après avoir lu quelques exemples de code de la documentation Redux. Dans un premier temps c'était un peu bizarre, mais juste parce que je n'y était pas habitué. Mais après une journée à écrire mon code de cette manière, je ne me voyais pas recommencer à utiliser des point-virgules dans mon code. Ils semblaient lourds, non-nécessaires et pas à leur place. Un code sans point-virgule est plus facile à lire à mon avis, et aussi plus facile à taper.

Je vous recommande de lire [la documentation ESLint sur les point-virgules](http://eslint.org/docs/rules/semi) (:uk:). Comme mentionné dans cette page, si vous décidez de ne pas utiliser de point-virgule, il y a quelques cas, assez rares, où ceux-ci sont toutefois nécessaire. ESLint peut vous protéger de tels cas avec la règle `no-unexpected-multiline`. Configurons ESLint pour pouvoir ne plus mettre de point-virgule en toute sécurité dans `.eslintrc.json`:

```json
{
  "extends": "airbnb",
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2
  }
}
```

:checkered_flag: Lancez `yarn test`, et tout devrait se passer tranquillement. Essayez d'ajouter des point-virgules n'importe où dans le code pour vérifier le linter fonctionne.

Nous sommes bien conscient que certains d'entre vous vont vouloir garder les point-virgules, ce qui rend le code de ce tutoriel gênant. Si vous utilisez ce tutoriel juste pour apprendre, nous sommes sûr qu'il restera supportable pour apprendre sans point-virgules, jusqu'à pouvoir les utiliser à nouveau dans vos projets. Si vous voulez utilise le code fourni dans ce tutoriel comme *boilerplate*, cela demandera un peu de réécriture, ce qui devrait être rapide avec ESLint configuré pour imposer les point-virgules, celui-ci vous guidant durant tout le processus. Nous nous excusons si tel est votre cas !

### Compat

[Compat](https://github.com/amilajack/eslint-plugin-compat) est un plugin ESLint qui vous prévient si vous utilisez des API Javascript ne sont pas disponibles dans les navigateurs que vous supportez. Il utilise [Browserslist](https://github.com/ai/browserslist), qui s'appuie sur [Can I Use](http://caniuse.com/).

- Lancez `yarn add --dev eslint-plugin-compat`

- Ajoutez les lignes suivantes à votre fichier `package.json`. Elles indiquent que nous souhaitons supporter les navigateurs représentant plus de 1% du trafic.

```json
"browserslist": ["> 1%"],
```

- Edit your `.eslintrc.json` file like so:

```json
{
  "extends": "airbnb",
  "plugins": [
    "compat"
  ],
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2,
    "compat/compat": 2
  }
}
```

Vous pouvez essayer le plugin en utilisant `navigator.serviceWorker` ou `fetch` dans votre code par exemple, ce qui devrait lancer un warning ESLint.

### ESLint dans votre éditeur

Ce chapitre vous a permi d'installer ESLint dans votre terminal, ce qui est plutôt cool pour détecter les erreurs pendant le *build*/avant de *push*, mais vous voulez aussi probablement l'intégrer dans votre IDE pour un retour immédiat. N'UTILISEZ PAS le linter ES6 natif de votre IDE. Configurez le pour que l'exécutable qu'il utilise pour le *linting* soit celui dans votre dossier `node_modules`. De cette façon, il peut utiliser toute la configuration de votre projet, la préconfiguration Airbnb etc. Sinon, vous aurez juste un linting ES6 générique.

## Flow

> 💡 **[Flow](https://flowtype.org/)**: Un type checker statique par Facebook. Il détecte les types incohérents dans le code. Par exemple, il vous donnera une erreur si vous utilisez une chaîne de caractère là où il faudrait utiliser un nombre.

Maintenant, notre code JavaScript est de l'ES6 valide. Flow peut analyser du JavaScript pur pour nous donner quelques idées, mais pour utiliser toute sa puissance, on va avoir besoin d'ajouter des annotations de type dans notre code, ce qui le rendra non-standard. On va avoir besoin d'apprendre à Babel et à ESLint ce que sont ces annotations de types pour qu'ils ne paniquent pas quand ils parcourront nos fichiers.

- Lancez `yarn add --dev flow-bin babel-preset-flow babel-eslint eslint-plugin-flowtype`

`flow-bin` est l'exécutable pour lancer Flow dans notre tâche `scripts` , `babel-preset-flow` est une préconfiguration pour que Babel puisse comprendre les annotations de type, `babel-eslint` est un package qui permet à ESLint de *s'appuier sur le parseur de Babel* au lieu du sien, et `eslint-plugin-flowtype` est un plugin ESLint pour *linter* les annotations Flow. Pfiou.

- Modifiez votre fichier `.babelrc` comme ceci :

```json
{
  "presets": [
    "env",
    "flow"
  ]
}
```

- Modifiez aussi votre fichier `.eslintrc.json` :

```json
{
  "extends": [
    "airbnb",
    "plugin:flowtype/recommended"
  ],
  "plugins": [
    "flowtype",
    "compat"
  ],
  "rules": {
    "semi": [2, "never"],
    "no-unexpected-multiline": 2,
    "compat/compat": 2
  }
}
```

**Remarque**: `plugin:flowtype/recommended` contient les instructions pour dire à ESLint d'utiliser le parseur de Babel. Si vous voulez être plus explicite, libre à vous d'ajouter `"parser": "babel-eslint"` dans `.eslintrc.json`.

Ça fait beaucoup d'un coup, essayez de prendre un instant pour y penser. C'est assez impressionnant de voir qu'il est possible pour ESLint d'utiliser le parseur de Babel pour comprendre les annotations Flow. Ces 2 outils sont formidables pour être aussi modulaires !

- Chaînez `flow` à votre tâche `test` :

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow"
},
```

- Créez un fichier `.flowconfig` à la racine de votre projet contenant les lignes suivantes :

```flowconfig
[options]
suppress_comment= \\(.\\|\n\\)*\\flow-disable-next-line
```
Il s'agit d'un petit utilitaire que nous configurons pour que Flow ignore les warning détectés sur la ligne suivante. Vous pourrez l'utiliser de cette façon (qui rappelle `eslint-disable`) :

```js
// flow-disable-next-line
something.flow(doesnt.like).for.instance()
```

Bien, on devrait en avoir fini pour ce qui est de la configuration.

- Ajoutez une annotation Flow au fichier `src/dog.js` comme ceci :

```js
// @flow

class Dog {
  name: string

  constructor(name: string) {
    this.name = name
  }

  bark() {
    return `Wah wah, I am ${this.name}`
  }
}

export default Dog
```

Le commentaire `// @flow` dit à Flow que nous voulons que ce fichier soit soumis aux vérifications de types. Pour le reste, les annotations Flow sont typiquement deux-point après le paramètre d'une fonction ou le nom d'une fonction. Voici la [documentation](https://flowtype.org/docs/quick-reference.html) pour plus de détail.

- Ajoutez  également`// @flow` en haut de votre fichier `index.js`

`yarn test` devrait maintenant *linter* et vérifier les types de votre code comme il faut.

Il y a deux choses que l'on veut essayer :

- Dans `dog.js`, remplacez `constructor(name: string)` par `constructor(name: number)`, et lancez `yarn test`. Vous devriez obtenir une erreur **Flow** qui dit que ces 2 types sont incompatibles. **Ce qui signifie que Flow est correctement configuré :+1: !**

- Maintenant, remplacez `constructor(name: string)` par `constructor(name:string)`, et lancez `yarn test`. Vous devriez obtenir une erreur **ESLint** qui dit que vos annotations Flow devrait avoir un espace après les deux points. should have a space after the colon.**Le pluglin Flow pour ESLint est correctement configuré :+1: !**

:checkered_flag: Si vous avez obtenu ces 2 erreurs, vous avez réussi à configurer Flow et ESLint ! Prenez garde à bien remettre en place les espaces dans vos annotations Flow :wink:

### Flow dans votre éditeur de texte

Comme avec ESLint, vous devriez consacrer un peu de votre temps à configurer votre IDE/éditeur de texte pour qu'il vous donne un feedback immédiat quand Flow détecte un problème dans votre code.

## Jest

> 💡 **[Jest](https://facebook.github.io/jest/)**: Une bibliothèque de test JavaScript créée par Facebook. Elle est très simple à configurer et fournit tout ce qu'il faut pour faire nos tests unitaires. Jest permet aussi de tester les composants React.

- Lancez `yarn add --dev jest babel-jest` pour installer Jest et le package pour l'utiliser avec Babel.

- Dans `.eslintrc.json` ajoutez les lignes suivantes à la racine de l'objet pour permettre l'utilisation des fonctions Jest sans avoir à les importer dans chaque fichier de test :

```json
"env": {
  "jest": true
}
```

- Créez un fichier `src/dog.test.js` qui contient les lignes suivantes :

```js
import Dog from './dog'

test('Dog.bark', () => {
  const testDog = new Dog('Test')
  expect(testDog.bark()).toBe('Wah wah, I am Test')
})
```

- Ajoutez `jest` à votre script `test` :

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage"
},
```

Le drapeau `--coverage` dit à Jest de générer automatiquement des données de couverture pour nos tests. C'est utile pour voir quelles parties de votre code manque de test. Il écrit ces données dans un dossier nommé `coverage`.

- Ajoutez `/coverage/` à votre fichier `.gitignore`

:checkered_flag: Lancez `yarn test`. Après avoir *linté* et vérifié les types, nos tests Jest devraient se lancer et afficher un tableau de données. Tout devrait être vert !

## Git Hooks avec Husky

> :question: **[Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)**: Des scripts qui sont lancés quand certaines actions ont lieu, comme un commit ou un push

Ok, maintenant on a cette superbe tâche `test` qui nous dit si notre code est bon ou pas. Maintenant, on va configurer des Git Hooks pour lancer automatiquement cette tâche avant chaque `git commit` et `git push`, ce qui nous évitera de pusher du mauvais code dans notre repo si la tâche `test` ne réussi pas.

[Husky](https://github.com/typicode/husky) est un package qui permet de créer facilement des Git Hooks.

- Lancez `yarn add --dev husky`

Tout ce que nous avons à faire est de créer deux tâche dans `scripts`, `precommit` et `prepush` :

```json
"scripts": {
  "start": "babel-node src",
  "test": "eslint src && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test"
},
```

:checkered_flag: Maintenant, si vous essayez de faire un `git commit` ou un `git push`, la tâche `test` devrait se lancer automatiquement.

Si ça ne marche pas, il est possible que `yarn add --dev husky` n'ait pas correctement installé les Git Hooks. Ce problème peut arriver à certaines personnes. Si c'est votre cas, lancez `yarn add --dev husky --force`,et laissez peut-être un mot décrivant votre situation sur [cette issue](https://github.com/typicode/husky/issues/84).

**Remarque**: Si vous pushez directement après votre commit, vous pouvez utiliser `git push --no-verify` pour éviter d'avoir à lancer à nouveau les tests.

Prochaine section: [03 - Express, Nodemon, PM2](03-express-nodemon-pm2.md#readme)

Retour à la [section précédente](01-node-yarn-package-json.md#readme) ou au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).
