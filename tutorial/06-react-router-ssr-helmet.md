# 06 - React Router, Server-Side Rendering, et Helmet

Le code pour ce chapitre est disponible [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/06-react-router-ssr-helmet).

Dans ce chapitre, nos allons créer différentes pages pour notre app et rendre la navigation entre celles-ci possible. C'est parti !

## React Router

> :bulb: **[React Router](https://reacttraining.com/react-router/)** est une bibliothèque faite pour naviguer entre les différentes pages de votre app React. Elle peut être utilisée à la fois sur le client et aussi sur le serveur.

React Router a eu de grosses modifications à la sortie de la v4, toujours en beta. Puisqu'on a envie que ce tutoriel soit durable, nous utiliserons la v4 de React Router.

- Lancez `yarn add react-router@next react-router-dom@next`

Du côté du client, on a d'abord besoin *d'emballer* notre app dans un composant `BrowserRouter`.

- Modifier votre fichier `src/client/index.jsx` comme ceci :

```js
// [...]
import { BrowserRouter } from 'react-router-dom'
// [...]
const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <BrowserRouter>
      <AppContainer>
        <AppComponent />
      </AppContainer>
    </BrowserRouter>
  </Provider>
```

## Pages

Notre app aura 4 pages :

- Une page "Home"
- Une page "Hello" qui montre un bouton et un message pour l'action synchrone
- Une page "Hello Async" qui montre un bouton et un message pour l'action asynchrone
- Une page 404 "Not Found".

- Créez un fichier `src/client/component/page/home.jsx` contenant :

```js
// @flow

import React from 'react'

const HomePage = () => <p>Home</p>

export default HomePage
```

- Créez un fichier `src/client/component/page/hello.jsx` contenant :

```js
// @flow

import React from 'react'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const HelloPage = () =>
  <div>
    <Message />
    <HelloButton />
  </div>

export default HelloPage

```

- Créez un fichier `src/client/component/page/hello-async.jsx` contenant :

```js
// @flow

import React from 'react'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const HelloAsyncPage = () =>
  <div>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage
```

- Créez un fichier `src/client/component/page/not-found.jsx` contenant :

```js
// @flow

import React from 'react'

const NotFoundPage = () => <p>Page not found</p>

export default NotFoundPage
```

## Navigation

Ajoutons quelques routes dans le fichier de configuration partagé.

- Éditez votre fichier `src/shared/routes.js` comme ceci :

```js
// @flow

export const HOME_PAGE_ROUTE = '/'
export const HELLO_PAGE_ROUTE = '/hello'
export const HELLO_ASYNC_PAGE_ROUTE = '/hello-async'
export const NOT_FOUND_DEMO_PAGE_ROUTE = '/404'

export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

La route `/404` va juste être utilisée dans un lien de navigation pour bien montrer ce qui se passe quand on clique sur un lien brisé .

- Créez un fichier `src/client/component/nav.jsx` contenant :

```js
// @flow

import React from 'react'
import { NavLink } from 'react-router-dom'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../../shared/routes'

const Nav = () =>
  <nav>
    <ul>
      {[
        { route: HOME_PAGE_ROUTE, label: 'Home' },
        { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
        { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
        { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
      ].map(link => (
        <li key={link.route}>
          <NavLink to={link.route} activeStyle={{ color: 'limegreen' }} exact>{link.label}</NavLink>
        </li>
      ))}
    </ul>
  </nav>

export default Nav
```

Ici on crée simplement quelques `NavLink` qui utilisent les routes que nous avons déclarées précédemment.

- Pour finir, éditez `src/client/app.jsx` comme ceci :

```js
// @flow

import React from 'react'
import { Switch } from 'react-router'
import { Route } from 'react-router-dom'
import { APP_NAME } from '../shared/config'
import Nav from './component/nav'
import HomePage from './component/page/home'
import HelloPage from './component/page/hello'
import HelloAsyncPage from './component/page/hello-async'
import NotFoundPage from './component/page/not-found'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
} from '../shared/routes'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Nav />
    <Switch>
      <Route exact path={HOME_PAGE_ROUTE} render={() => <HomePage />} />
      <Route path={HELLO_PAGE_ROUTE} render={() => <HelloPage />} />
      <Route path={HELLO_ASYNC_PAGE_ROUTE} render={() => <HelloAsyncPage />} />
      <Route component={NotFoundPage} />
    </Switch>
  </div>

export default App
```

:checkered_flag: Lancez `yarn start` et `yarn dev:wds`. Dans votre navigateur, rendez-vous sur `http://localhost:8000`, et cliquez sur les liens pour naviguer entre nos différentes pages. Vous devriez voir que l'URL change dynamiquement. Passez d'une page à l'autre et utilisez le bouton "page précédente" de votre navigateur pour voir que l'historique de votre navigateur fonctionne comme prévu.

Maintenant, disons que vous êtes allés sur `http://localhost:8000/hello` de cette façon. Rafraîchissez la page. Maintenant, vous obtenez une 404, parce que notre serveur Express ne répond qu'à `/`. En fait, quand vous naviguiez entre les pages, vous le faisiez seulement du côté du client. Maintenant, ajoutons le server-side rendering (rendu côté serveu :fr:) à tout ceci pour obtenir le bon comportement.

## Server-Side Rendering

> :question: Le **Server-Side Rendering**, ou rendu côté serveur, signifie afficher votre app au chargement initial de la page au lieu de s'appuyer sur le JavaScript pour l'afficher dans le navigateur du client.

Le SSR est essentiel pour le SEO (search engine optimization - référencement naturel :fr: ) et fournit une meilleure expérience utilisateur en montrant tout de suite notre app à l'utilisateur  

La première chose que nous allons faire ici, c'est migrer la plupart de notre code client dans la partie partagée / isomorphique / universelle de notre code, puisque c'est le serveur qui va maintenant afficher notre app React.

### La grosse migration vers `shared`

- Déplacez tous vos fichiers situés dans le dossier `client` à `shared`, **sauf** `src/client/index.jsx`.

On doit modifier plusieurs de nos imports.

- Dans `src/client/index.jsx`,remplacez les 3 occurences de `'./app'` par `'../shared/app'`, et `'./reducer/hello'` par `'../shared/reducer/hello'`

- Dans `src/shared/app.jsx`, remplacez `'../shared/routes'` par `'./routes'` et `'../shared/config'` par `'./config'`

- Dans `src/shared/component/nav.jsx`, remplacez `'../../shared/routes'` par `'../routes'`

### Changements du serveur

- Créez un fichier `src/server/routing.js` contenant :

```js
// @flow

import {
  homePage,
  helloPage,
  helloAsyncPage,
  helloEndpoint,
} from './controller'

import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  helloEndpointRoute,
} from '../shared/routes'

import renderApp from './render-app'

export default (app: Object) => {
  app.get(HOME_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, homePage()))
  })

  app.get(HELLO_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloPage()))
  })

  app.get(HELLO_ASYNC_PAGE_ROUTE, (req, res) => {
    res.send(renderApp(req.url, helloAsyncPage()))
  })

  app.get(helloEndpointRoute(), (req, res) => {
    res.json(helloEndpoint(req.params.num))
  })

  app.get('/500', () => {
    throw Error('Fake Internal Server Error')
  })

  app.get('*', (req, res) => {
    res.status(404).send(renderApp(req.url))
  })

  // eslint-disable-next-line no-unused-vars
  app.use((err, req, res, next) => {
    // eslint-disable-next-line no-console
    console.error(err.stack)
    res.status(500).send('Something went wrong!')
  })
}
```
C'est dans ce fichier qu'on va gérer les requêtes et les réponses. Les appels à la logique métier sont externalisés dans un module `controller` différent.

**Remarque**: Vous trouverez beaucoup d'exemple de React Router qui utilisent `*` comme route sur le serveur, laissant la gestion de toutes les routes à React Router. Puisque toutes les requêtes passent par la même fonction, ça rend l'implémentation de page style MVC (Model-View-Controller ; Modèle-Vue-Contrôleur :fr:) peu pratique. Au lieu de faire ça, ici on déclare explicitement les routes et leurs réponses afin d'être capable d'aller récupérer les données depuis la base de données et de les passer à une page donnée facilement.

- Créez un fichier `src/server/controller.js` contenant :

```js
// @flow

export const homePage = () => null

export const helloPage = () => ({
  hello: { message: 'Server-side preloaded message' },
})

export const helloAsyncPage = () => ({
  hello: { messageAsync: 'Server-side preloaded message for async page' },
})

export const helloEndpoint = (num: number) => ({
  serverMessage: `Hello from the server! (received ${num})`,
})
```

Voici notre contrôleur. Typiquement, il s'occupe de toute la logique métier et des appels à la base de données mais, dans notre cas, on code juste en dur des résultats. Ces résultats sont repassés au module `routing` afin d'être utilisés pour initialiser notre store Redux SSR. (rappel : SSR = Server-Side Rendering = Rendu Côté Serveur :fr:)

- Créez un fichier `src/server/init-store.js` contenant :

```js
// @flow

import Immutable from 'immutable'
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'

import helloReducer from '../shared/reducer/hello'

const initStore = (plainPartialState: ?Object) => {
  const preloadedState = plainPartialState ? {} : undefined

  if (plainPartialState && plainPartialState.hello) {
    // flow-disable-next-line
    preloadedState.hello = helloReducer(undefined, {})
      .merge(Immutable.fromJS(plainPartialState.hello))
  }

  return createStore(combineReducers({ hello: helloReducer }),
    preloadedState, applyMiddleware(thunkMiddleware))
}

export default initStore
```
La seule chose qu'on fait ici, à part appeler `createStore` et appliquer un middleware, c'est de fusionner le pur objet JS qu'on a reçu du  `controller` en state Redux par défaut contenant des objets immuables.

- Éditez `src/server/index.js` de cette façon

```js
// @flow

import compression from 'compression'
import express from 'express'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'

const app = express()

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

app.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

Rien de spécial ici, on appelle juste `routing(app)` au lieu d'implémenter le routage dans ce fichier.

- Renommez `src/server/render-app.js` en `src/server/render-app.jsx` et éditez-le comme ceci :

```js
// @flow

import React from 'react'
import ReactDOMServer from 'react-dom/server'
import { Provider } from 'react-redux'
import { StaticRouter } from 'react-router'

import initStore from './init-store'
import App from './../shared/app'
import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <App />
      </StaticRouter>
    </Provider>)

  return (
    `<!doctype html>
    <html>
      <head>
        <title>FIX ME</title>
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
      <body>
        <div class="${APP_CONTAINER_CLASS}">${appHtml}</div>
        <script>
          window.__PRELOADED_STATE__ = ${JSON.stringify(store.getState())}
        </script>
        <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
      </body>
    </html>`
  )
}

export default renderApp
```

C'est dans `ReactDOMServer.renderToString` que la magie a lieu. React va évaluer toute notre `App` `shared`, et retourner une pure string d'éléments HTML. `Provider` fonctionne de la même façon que sur le client, mais sur le serveur, on *emballe* notre app dans `StaticRouter` au lieu de `BrowserRouter`. Afin de passer le store Redux du au client, on le passe à `window.__PRELOADED_STATE__` qui est juste un nom de variable arbitraire.

**Remarque**: Les objets immuables implémentent la méthode `toJSON()` ce qui veut dire que vous pouvez utiliser `JSON.stringify` pour les passer en strings JSON pures.

- Éditez `src/client/index.jsx` pour utiliser ce state préchargé :

```js
import Immutable from 'immutable'
// [...]

/* eslint-disable no-underscore-dangle */
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose
const preloadedState = window.__PRELOADED_STATE__
/* eslint-enable no-underscore-dangle */

const store = createStore(combineReducers(
  { hello: helloReducer }),
  { hello: Immutable.fromJS(preloadedState.hello) },
  composeEnhancers(applyMiddleware(thunkMiddleware)))
```

Ici, on alimente notre store côté client avec le `preloadedState` qu'on a reçu du serveur.

:checkered_flag: Maintenant, vous pouvez lancer `yarn start`, `yarn dev:wds` et naviguer entre les pages. Rafraîchissez la page sur `/hello`, `/hello-async`, et `/404` (ou n'importe quelle autre URI). Tout devrait fonctionner correctement. Remarquez comme `message` et `messageAsync` varient selon si vous êtes allés sur cette page depuis le client ou si elle vient du rendu côté serveur.

### React Helmet

> :question: **[React Helmet](https://github.com/nfl/react-helmet)**: Une bibliothèque pour injecter du contenu dans le `head` d'une app React, à la fois sur le client et le serveur.

Nous avons fait exprès de vous faire écrire `FIX ME` dans le titre pour souligner le fait que même si nous faisons du server-side rendering, on ne remplit pas la balise `title` correctement (ni aucune autre balise dans le `head` qui varie selon la page sur laquelle vous vous trouvez).

- Lancez `yarn add react-helmet`

- Éditez `src/server/render-app.jsx` comme ceci :

```js
import Helmet from 'react-helmet'
// [...]
const renderApp = (/* [...] */) => {
  // [...]
  const appHtml = ReactDOMServer.renderToString(/* [...] */)
  const head = Helmet.rewind()

  return (
    `<!doctype html>
    <html>
      <head>
        ${head.title}
        ${head.meta}
        <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
      </head>
    [...]
    `
  )
}
```

React Helmet utilise `rewind` de [react-side-effect](https://github.com/gaearon/react-side-effect) pour extraire des données du rendu de notre app, qui contiendra bientôt quelques composants `<Helmet />` components. Ces composants `<Helmet />` sont là où nous initialisons `title` et d'autres détails de `head` pour chaque page. Noteez que `Helmet.rewind()` *doit* arriver après `ReactDOMServer.renderToString()`.

- Éditez `src/shared/app.jsx` comme ceci :

```js
import Helmet from 'react-helmet'
// [...]
const App = () =>
  <div>
    <Helmet titleTemplate={`%s | ${APP_NAME}`} defaultTitle={APP_NAME} />
    <Nav />
    // [...]
```

- Éditez `src/shared/component/page/home.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <h1>{APP_NAME}</h1>
  </div>

export default HomePage

```

- Éditez `src/shared/component/page/hello.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloButton from '../../container/hello-button'
import Message from '../../container/message'

const title = 'Hello Page'

const HelloPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <Message />
    <HelloButton />
  </div>

export default HelloPage
```

- Éditez `src/shared/component/page/hello-async.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import HelloAsyncButton from '../../container/hello-async-button'
import MessageAsync from '../../container/message-async'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default HelloAsyncPage

```

- Éditez `src/shared/component/page/not-found.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

const title = 'Page Not Found'

const NotFoundPage = () =>
  <div>
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <h1>{title}</h1>
  </div>

export default NotFoundPage
```

En fait, le composant `<Helmet>` ne fait rien apparaître du tout, il injecte juste du contenu dans le `head` de notre document et montre les mêmes données au serveur.

:checkered_flag: Lancez `yarn start`, `yarn dev:wds` et naviguez entre les différentes pages. Le titre de votre onglet devrait changer selon la page sur laquelle vous vous trouvez et devrait rester le même quand vous rafraîchissez la page. Affichez le code source de la page dans votre navigateur pour voir comment React Helmet initialise les balises `title` et `meta` tags même pour du rendu côté serveur.

Prochaine section: [07 - Socket.IO](07-socket-io.md#readme)

Retour à la [section précédente](05-redux-immutable-fetch.md#readme) ou au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).
