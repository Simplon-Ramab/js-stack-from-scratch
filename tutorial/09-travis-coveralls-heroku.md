# 09 - Travis, Coveralls et Heroku

Le code pour ce chapitre est disponible dans la branche `master` du [repo JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate).

Dans ce chapitre, nous allons intégrer notre app avec des services tiers. Ces services ont des offres payantes et gratuites. C'est un peu controversé d'utiliser de tels services dans un tutoriel qui s'appuie exclusivement sur des outils gratuits, open-source et lancés par la communauté. C'est la raison pour laquelle nous maintenons 2 branches différentes du [repo JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate), `master` et `master-no-services`.

## Travis

> :bulb: **[Travis CI](https://travis-ci.org/)** est une plateforme d'intégration continue populaire qui est gratuite pour les projets open-source.

Si votre projet est hebergé publiquement sur Github, intégrer Travis est très simple. D'abord, identifiez-vous sur Travis grâce à votre compte Github et ajoutez votre repo.

- Ensuite, créez un fichier `.travis.yml` contenant :

```yaml
language: node_js
node_js: node
script: yarn test && yarn prod:build
```

Travis détectera automatiquement que vous utilisez Yarn car vous avez un fichier `yarn.lock`. Chaque fois que vous pushez du code à votre repo Github, il lancera `yarn test && yarn prod:build`. Si tout se passe bien votre *build* devrait être vert.

## Coveralls

> :bulb: **[Coveralls](https://coveralls.io)** est un service qui vous donne un historique et des statistiques concernant la couverture de votre code par vos tests.

Si votre projet est open-source sur Github et compatible avec les services d'intégration continue supportés par Coveralls, la seule chose que vous avez besoin de faire est d'acheminer le fichier généré par Jest à l'exécutable `coveralls`.

- Lancez `yarn add --dev coveralls`

- Éditez l'instruction `script` de votre fichier `.travis.yml` comme ceci :

```yaml
script: yarn test && yarn prod:build && cat ./coverage/lcov.info | ./node_modules/coveralls/bin/coveralls.js
```

Maintenant, à chaque fois que Travis lancera un *build*, il enverra automatiquement les données de vos tests Jest à Coveralls.

## Badges

Tout est vert sur Travis et Coveralls? Super, montrez-le au monde entier avec de jolis badges :sparkles: !

Vous pouvez utiliser directement le code fourni par Travis ou Coveralls, ou utiliser [shields.io](http://shields.io/) pour homogénéiser ou customiser vos badges. Utilisons shields.io !

- Créez ou éditez votre fichier `README.md` de la façon suivante :

```md
[![Build Status](https://img.shields.io/travis/GITHUB-USERNAME/GITHUB-REPO.svg?style=flat-square)](https://travis-ci.org/GITHUB-USERNAME/GITHUB-REPO)
[![Coverage Status](https://img.shields.io/coveralls/GITHUB-USERNAME/GITHUB-REPO.svg?style=flat-square)](https://coveralls.io/github/GITHUB-USERNAME/GITHUB-REPO?branch=master)
```

Évidemment, remplacez `GITHUB-USERNAME/GITHUB-REPO` par votre nom d'utilisateur sur Github et le nom de votre repo.

## Heroku

> :bulb: **[Heroku](https://www.heroku.com/)** est un [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) sur lequel on peut déployer. Il s'occupe des détails d'infrastructure, vous laissant vous concentrer sur le développement de votre app sans vous soucier de ce qui se passe derrière.

Ce tutoriel n'est en aucun cas sponsorisé par Heroku, mais puisqu'Heroku est une très, très grosse plateforme, nous allons vous montrer comme faire pour y déployer votre app. Ouais, c'est le genre de truc cool que vous obtenez quand vous faites un produit cool !

**Remarque**: On pourrait ajouter une section AWS plus tard, mais une chose à la fois.

### Web setup

- Si ce n'est pas encore fait, installez la [CLI Heroku](https://devcenter.heroku.com/articles/getting-started-with-nodejs) et connectez-vous.

- Allez sur votre [Dashboard Heroku](https://dashboard.heroku.com/) et créez deux apps, une appelée `your-project` et une autre `your-project-staging` par exemple.

On va laisser Heroku s'occuper de transpiler notre code ES6/Flow avec Babel, et générer les bundles client avec Webpack. Mais puisque ce sont des `devDependencies`, Yarn ne les installera pas dans un environnement de production comme Heroku. Modifions ce comportement avec la variable d'environnement `NPM_CONFIG_PRODUCTION`.

- Dans les deux apps, sous Settings > Config Variables, ajoutez `NPM_CONFIG_PRODUCTION` et initialisez-la à `false`.

- Créez un Pipeline, et donnez accès à Heroku à votre Github.

- Ajoutez les 2 apps au Pipeline, faites que celle en attente s'auto-déploie dès qu'il y a des changements sur la branche `master`, et activez Review Apps.

Bien, préparons notre projet pour le déploiement sur Heroku.

### Lancer le mode "production" en local

- Créez un fichier `.env` contenant :

```.env
NODE_ENV='production'
PORT='8000'
```
C'est dans ce fichier que vous devez mettre toutes vos variables locales et vos variables secrètes. Ne commitez pas ce fichier dans un repo public si votre projet est privé.

- Ajoutez `/.env` à votre fichier `.gitignore`

- Créez un fichier `Procfile` contenant :

```Procfile
web: node lib/server
```
C'est ici qu'on spécifie le point d'entrée de notre serveur.

Nous n'allons plus utiliser PM2 mais `heroku local` pour passer en mode production localement.

- Lancez `yarn remove pm2`

- Éditez votre script `prod:start` dans votre fichier `package.json`:

```json
"prod:start": "heroku local",
```

- Retirez `prod:stop` de votre fichier `package.json`. Nous n'en avons plus besoin puisque `heroku local` est un processus bloquant que l'on peut *kill*  avec un Ctrl+C, pas comme `pm2 start`.

:checkered_flag: Lancez `yarn prod:build` et `yarn prod:start`. Cela devrait démarrer votre serveur et vous afficher les logs.

### Déployer en production

- Ajoutez les lignes suivantes à `scripts` dans votre fichier `package.json`:

```json
"heroku-postbuild": "yarn prod:build",
```

`heroku-postbuild` est une tâche qui sera lancée chaque fois que vous déployez votre app sur Heroku.

Vous allez aussi probablement vouloir spécifier quelle version de Node ou de Yarn Heroku doit utiliser.

- Ajoutez ceci à votre fichier `package.json`:

```json
"engines": {
  "node": "7.x",
  "yarn": "0.20.3"
},
```

- Créez un fichier `app.json` contenant :

```json
{
  "env": {
    "NPM_CONFIG_PRODUCTION": "false"
  }
}
```

C'est ce que votre Review Apps va utiliser.

Maintenant, vous êtes prêt à utiliser les pipelines de déploiement d'Heroku ! :tada:

:checkered_flag: Créez une nouvelle branche git, et ouvrez une Pull Request sur Github pour instantier une Review App. Vérifiez vos changement sur l'URL Review App URL, et si tout à l'air correct, mergez votre Pull Request avec `master` sur Github. Quelques minutes plus tard, votre app en attente devrait avoir été déployée automatiquement. Vérifiez vos changement sur l'URL de l'app en attente, et si tout a toujours l'air bon, passez en production.

Vous avez terminé ! Félicitations si vous avez terminé tout ce tutoriel en partant de rien :tada: :clap: !

Vous avez bien mérité cet emoji trophée : :trophy:

Retourner à la [section précédente](08-bootstrap-jss.md#readme) ou au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).

> Merci infiniment à [@verekia](https://twitter.com/verekia) pour ce tutoriel très complet ! You rock ! :metal:
