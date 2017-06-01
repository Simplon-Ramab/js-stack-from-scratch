# JavaScript Stack from Scratch :ok_hand:

[![Build Status](https://travis-ci.org/verekia/js-stack-from-scratch.svg?branch=master)](https://travis-ci.org/verekia/js-stack-from-scratch)
[![Release](https://img.shields.io/github/release/verekia/js-stack-from-scratch.svg?style=flat-square)](https://github.com/verekia/js-stack-from-scratch/releases)
[![Dependencies](https://img.shields.io/david/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate)
[![Dev Dependencies](https://img.shields.io/david/dev/verekia/js-stack-boilerplate.svg?style=flat-square)](https://david-dm.org/verekia/js-stack-boilerplate?type=dev)
[![Gitter](https://img.shields.io/gitter/room/js-stack-from-scratch/Lobpar.svg?style=flat-square)](https://gitter.im/js-stack-from-scratch/)

[![React](/img/react-padded-90.png)](https://facebook.github.io/react/)
[![Redux](/img/redux-padded-90.png)](http://redux.js.org/)
[![React Router](/img/react-router-padded-90.png)](https://github.com/ReactTraining/react-router)
[![Flow](/img/flow-padded-90.png)](https://flowtype.org/)
[![ESLint](/img/eslint-padded-90.png)](http://eslint.org/)
[![Jest](/img/jest-padded-90.png)](https://facebook.github.io/jest/)
[![Yarn](/img/yarn-padded-90.png)](https://yarnpkg.com/)
[![Webpack](/img/webpack-padded-90.png)](https://webpack.github.io/)
[![Bootstrap](/img/bootstrap-padded-90.png)](http://getbootstrap.com/)

Bienvenue sur ce tutoriel de prise en main d'une *stack* (ensemble d'outils :fr:) Javascript moderne : **JavaScript Stack from Scratch**.

> :exclamation: **Il s'agit de la V2 du tutoriel, des changements majeurs sont apparus depuis la première version en 2016. Jetez un oeil au [Change Log](/CHANGELOG.md) !**

Il s'agit d'un guide allant droit au but sur comment mettre en place toute une *stack* JavaScript. Pour le suivre, il faut avoir un minimum de connaissances en programmation ainsi que des bases de JavaScript. **Ce guide se concentre sur comment relier les outils ensemble** et vous donne les **exemples les plus simples possible** pour chaque outil. Vous pouvez voir ce tutoriel comme *un moyen d'écrire votre boilerplate (*modèle* :fr: *) depuis rien*. Le but de ce tutoriel étant d'assembler divers outils, on ne va pas explorer en détails comment ces outils fonctionnent individuellement. Pour cela, référez-vous à leur documentation ou essayez de trouver d'autres tutoriels si vous voulez en acquérir une plus grande connaissance.

Vous n'avez pas besoin d'utiliser la *stack* entière si vous construisez une simple page web avec quelques interactions en JS (une combinaison Browserify/Webpack + Babel + jQuery est suffisant pour écrire en ES6 dans différents fichiers), mais si vous voulez construire une web app évolutive et avez besoin d'aide pour mettre tous les différents outils en place, alors ce tutoriel est parfait pour vous ! :+1:

Une bonne partie de la *stack* décrite dans ce tutoriel utilise React. Si vous débutez et avez juste envie d'apprendre React, [create-react-app](https://github.com/facebookincubator/create-react-app) (:uk:) vous permettra de mettre en place un environnement React très rapidement avec une config déjà préparée. On vous recommande cette approche si par exemple vous arrivez dans une équipe qui utilise React et que vous avez besoin d'une petite mise à niveau avec un environnement d'apprentissage. Dans ce tutoriel, nous n'utiliserons pas de configuration toute prête car nous souhaitons que vous compreniez tout ce qui se passe sous le capot.

Des exemples de code sont disponibles pour chaque chapitre. Vous pouvez les lancer avec `yarn && yarn start`. Cependant, nous vous recommandons d'écrire tout par vous-même en suivant les instructions étape par étape. :wink:


Le code final est disponible dans le [repo JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate), et dans les [releases](https://github.com/verekia/js-stack-from-scratch/releases). Il y a aussi une [démo live](https://js-stack.herokuapp.com/).

Ce tutoriel fonctionne sur les plateformes suivantes: Linux, OSX, Windows.

## Sommaire

[01 - Node, Yarn, `package.json`](/tutorial/01-node-yarn-package-json.md#readme)

[02 - Babel, ES6, ESLint, Flow, Jest, Husky](/tutorial/02-babel-es6-eslint-flow-jest-husky.md#readme)

[03 - Express, Nodemon, PM2](/tutorial/03-express-nodemon-pm2.md#readme)

[04 - Webpack, React, HMR](/tutorial/04-webpack-react-hmr.md#readme)

[05 - Redux, Immutable, Fetch](/tutorial/05-redux-immutable-fetch.md#readme)

[06 - React Router, Server-Side Rendering, Helmet](/tutorial/06-react-router-ssr-helmet.md#readme)

[07 - Socket.IO](/tutorial/07-socket-io.md#readme)

[08 - Bootstrap, JSS](/tutorial/08-bootstrap-jss.md#readme)

[09 - Travis, Coveralls, Heroku](/tutorial/09-travis-coveralls-heroku.md#readme)

## A venir :fire:

Configurer votre éditeur (Atom dans un premier temps), MongoDB, Progressive Web App, test E2E.

## Traductions :uk: :fr: :de: :cn: :jp: :ru:

Si vous souhaitez ajouter votre traduction, merci de lire les [recommandations de traduction](/how-to-translate.md) pour vous lancer !

### V2

- [Bulgare](https://github.com/mihailgaberov/js-stack-from-scratch) par [mihailgaberov](http://github.com/mihailgaberov)
- [Italien](https://github.com/fbertone/guida-javascript-moderno) par [Fabrizio Bertone](https://github.com/fbertone) - [fbertone.it](http://fbertone.it)
- [Chinois simplifié](https://github.com/yepbug/js-stack-from-scratch/) par [@yepbug](https://github.com/yepbug)
- [Français](https://github.com/naomihauret/js-stack-from-scratch/) par [Naomi Hauret](https://twitter.com/naomihauret)

Vous pouvez jeter un oeil aux autres [traductions en cours](https://github.com/verekia/js-stack-from-scratch/issues/147).

### V1 :baby:

- [中文](https://github.com/pd4d10/js-stack-from-scratch) par [@pd4d10](http://github.com/pd4d10)
- [Italien](https://github.com/fbertone/js-stack-from-scratch) par [Fabrizio Bertone](https://github.com/fbertone)
- [日本語](https://github.com/takahashim/js-stack-from-scratch) par [@takahashim](https://github.com/takahashim)
- [Русский](https://github.com/UsulPro/js-stack-from-scratch) par [React Theming](https://github.com/sm-react/react-theming)
- [ไทย](https://github.com/MicroBenz/js-stack-from-scratch) par [MicroBenz](https://github.com/MicroBenz)

## Crédits

Créé par [@verekia](https://twitter.com/verekia) – [verekia.com](http://verekia.com/).
Traduit par [@naomihauret](https://twitter.com/naomihauret)

Licence: MIT
