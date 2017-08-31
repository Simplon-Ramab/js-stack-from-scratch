# 05 - Redux, Immutable et Fetch

Le code pour ce chapitre est disponible [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/05-redux-immutable-fetch).

Dans ce chapitre, nous allons coupler React et Redux pour faire une app très simple. Cette app contiendra un message et un bouton. Le message changera lorsque l'utilisateur clique sur le bouton.

Avant de commencer, voici une rapide introduction à ImmutableJS, qui n'a absolument rien à voir avec React et Redux mais que nous utiliserons dans ce chapitre.

## ImmutableJS

> :bulb: **[ImmutableJS](https://facebook.github.io/immutable-js/)** (ou Immutable) est une bibliothèque Facebook qui sert à manipuler des collections immuables (immutable :uk:), comme les listes ou les maps. N'importe quel changement effectué sur une collection immuable retourne un nouvel objet sans transformer l'objet original.

Par exemple, au lieu de faire :

```js
const obj = { a: 1 }
obj.a = 2 // Mute `obj`
```

Vous feriez

```js
const obj = Immutable.Map({ a: 1 })
obj.set('a', 2) // Retourne un nouvel object sans muter `obj`
```

Cette approche suit le paradigme de la **programmation fonctionnelle** qui fonctionne très bien avec Redux. :ok_hand:

Quand on crée des collections immuables, la méthode `Immutable.fromJS()` s'avère être très pratique : elle prend n'importe quel objet/tableau JS et retourne une version immuable de celui-ci :

```js
const immutablePerson = Immutable.fromJS({
  name: 'Stan',
  friends: ['Kyle', 'Cartman', 'Kenny'],
})

console.log(immutablePerson)

/*
 *  Map {
 *    "name": "Stan",
 *    "friends": List [ "Kyle", "Cartman", "Kenny" ]
 *  }
 */
```

- Lancez `yarn add immutable@4.0.0-rc.2`

## Redux

> :bulb: **[Redux](http://redux.js.org/)** est une bibliothèque qui va prendre en charge les cycles de vie de notre application. Redux crée un  *store*, qui est la seule source de vérité à propos du *state* (l'état :fr:) de votre app à n'importe quel moment.

Commençons par la partie simple, déclarons nos actions Redux :

- Lancez `yarn add redux redux-actions`

- Créez un fichier `src/client/action/hello.js` qui contient :

```js
// @flow

import { createAction } from 'redux-actions'

export const SAY_HELLO = 'SAY_HELLO'

export const sayHello = createAction(SAY_HELLO)
```

Ce fichier expose une *action*, `SAY_HELLO`, et son *action creator* (*créateur d'action* :fr:), `sayHello`, qui est une fonction. Nous utilisons [`redux-actions`](https://github.com/acdlite/redux-actions) pour réduire le *boilerplate* associés aux actions Redux. `redux-actions` implémente le modèle de [standard d'action Flux](https://github.com/acdlite/flux-standard-action) , qui fait que les *action creators* retournent des objets ayant les attributs `type` et `payload`.

- Créez une fichier `src/client/reducer/hello.js` qui contient :

```js
// @flow

import Immutable from 'immutable'
import type { fromJS as Immut } from 'immutable'

import { SAY_HELLO } from '../action/hello'

const initialState = Immutable.fromJS({
  message: 'Initial reducer message',
})

const helloReducer = (state: Immut = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    default:
      return state
  }
}

export default helloReducer
```

Dans ce fichier, on initialise le *state* (l'état :fr:) de notre *reducer* (réducteur :fr:) avec un map immuable contenant une propriété, `message`, initialisé à `Initial reducer message`. Le `helloReducer` gère l'action `SAY_HELLO` en initialisant le nouveau `message` avec le *payload* (la charge :fr:) de l'action. L'annotation Flow pour `action` la déstructure en un `type` et un `payload`. Le `payload` peut être de n'importe quel type. Ca peut avoir l'air assez funky si c'est la première fois que vous voyez ça, mais ça reste assez compréhensible. Pour le type de  `state`, on utilise l'instruction Flow `import type`pour obtenir le type retourné de `fromJS`. On le renomme `Immut` pour plus de clareté, car `state: fromJS` serait assez confus. La ligne `import type` sera retirée comme n'importe quelle autre annotation Flow. Remarquez l'utilisation de `Immutable.fromJS()` et `set()` comme nous avons pu les voir précédemment.

## React-Redux

> :bulb: **[react-redux](https://github.com/reactjs/react-redux)** *connecte* un store Redux avec des composants React. Avec `react-redux`, quand le store Redux change, les composants React sont modifiés automatiquement. Ils déclenchent aussi les actions Redux.

- Lancez `yarn add react-redux`

Dans ces sections, nous allons créer des *Components* (composants :fr:) des *Containers* (conteneurs :fr:).

Les **Components** sont des composants React *stupides*, dans le sens où ils ne savent rien du state Redux (l'état de Redux :fr:). Les **Containers** sont des composants *intelligents* qui connaissent le *state* et que nous allons *connecter* à nos composants stupides.

- Créez un fichier `src/client/component/button.jsx` contenant:

```js
// @flow

import React from 'react'

type Props = {
  label: string,
  handleClick: Function,
}

const Button = ({ label, handleClick }: Props) =>
  <button onClick={handleClick}>{label}</button>

export default Button
```

**Remarque**: Vous pouvez voir un cas de *type alias* (alias de type :fr:) Flow ici. Nous définissons le type de `Props` avant d'annoter les `props` déstructurées de notre composant avec.

- Créez un fichier `src/client/component/message.jsx` contenant :

```js
// @flow

import React from 'react'

type Props = {
  message: string,
}

const Message = ({ message }: Props) =>
  <p>{message}</p>

export default Message
```

Voici des exemples de composants *stupides*. Ils n'ont pas de logique et affichent juste ce qu'on leur dit d'afficher via les React **props**. La principale différence entre `button.jsx` et `message.jsx` est que `Button` contient une référence à un *action dispatcher* (expéditeur d'action :fr:) dans ses props, où `Message` contient juste quelques données à afficher.

Encore une fois, les *components* ne savent rien des **actions** Redux ou sur le **state** de notre app. C'est pourquoi nous allons créer des **containers** intelligents qui alimenterons les bons *action dispatchers* et les bonnes données à ces composants stupides.

- Créez un fichier `src/client/container/hello-button.js` contenant :

```js
// @flow

import { connect } from 'react-redux'

import { sayHello } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHello('Hello!')) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

Ce container relie le composant `Button` avec l'action `sayHello` et la méthode `dispatch` de Redux.

- Créez un fichier `src/client/container/message.js` contenant :

```js
// @flow

import { connect } from 'react-redux'

import Message from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('message'),
})

export default connect(mapStateToProps)(Message)
```
 Ce container relie le state de l'app Redux avec le composant `Message`. Quand le state change, `Message` ré-affichera automatique le bon prop  (propriété :fr:) `message` . Ces connexions sont faites via la fonction `connect` de `react-redux`.

- Modifier votre fichier `src/client/app.jsx` de la manière suivante :

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import Message from './container/message'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
  </div>

export default App
```

Nous n'avons toujours pas initialisé le store Redux et n'avons pas encore mis ces deux containers dans notre app :

- Éditez le fichier `src/client/index.jsx` comme ceci :

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers } from 'redux'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

const store = createStore(combineReducers({ hello: helloReducer }),
  // eslint-disable-next-line no-underscore-dangle
  isProd ? undefined : window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__())

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```
Prenons un instant pour revoir tout ça. D'abord, nous créons un *store* avec  `createStore`. Les stores sont créés en leur passant des reducers. Ici, nous n'avons qu'un seul reducer, mais pour le bien de notre évolutivité future, nous utilisons `combineReducers` pour regrouper tous nos reducers ensemble. Le dernier paramètre bizarre de `createStore`  est un truc pour relier Redux aux [outils de développement](https://github.com/zalmoxisus/redux-devtools-extension) (Redux Devtools :uk:) du navigateur, qui sont incroyablement pratiques pour débugger. Puisque ESLint va se plaindre des underscores dans `__REDUX_DEVTOOLS_EXTENSION__`, on désactive cette règle. Ensuite, on *emballe* toute notre app dans le composant `Provider` de `react-redux`'  grâce à notre fonction `wrapApp`, et lui passons notre store.


:checkered_flag: Maintenant, vous pouvez lancer `yarn start` et `yarn dev:wds` et vous rendre sur `http://localhost:8000`. Vous devriez voir s'afficher "Initial reducer message" et un bouton. Quand vous cliquez sur le bouton, le message devrait changer pour "Hello!". Si vous avez installé les outils de développement Redux dans votre navigateur, vous devriez voir le state de votre app changer au fur et à mesure que vous cliquez sur le bouton.

Félicitations, nous avons enfin créé une app qui fait quelque chose :tada: :clap: ! Bon d'accord, ce n'est pas super impressionnant de l'extérieur, mais on sait tous que c'est propulsé par une stack hyper badass sous le capot :wink: .

## Étendre notre app avec un appel asynchrone

Nous allons maintenant ajouter un second bouton à notre app. Il déclenchera un appel AJAX pour récupérer un message depuis le serveur. Pour le bien de la démo, cet appel enverra aussi quelques données, le nombre codé en dur `1234`.

### L'extrémité du serveur (endpoint :uk:)

- Créez un fichier `src/shared/routes.js` contenant :

```js
// @flow

// eslint-disable-next-line import/prefer-default-export
export const helloEndpointRoute = (num: ?number) => `/ajax/hello/${num || ':num'}`
```

Cette fonction est une petite aide pour nous aider à produire les lignes suivantes :

```js
helloEndpointRoute()     // -> '/ajax/hello/:num' (for Express)
helloEndpointRoute(1234) // -> '/ajax/hello/1234' (for the actual call)
```

Maintenant, créons rapidement un fichier de test pour nous assurer que tout fonctionne correctement :

- Créez un fichier `src/shared/routes.test.js` contenant :

```js
import { helloEndpointRoute } from './routes'

test('helloEndpointRoute', () => {
  expect(helloEndpointRoute()).toBe('/ajax/hello/:num')
  expect(helloEndpointRoute(123)).toBe('/ajax/hello/123')
})
```

- Lancez `yarn test` et tous les tests devraient se dérouler avec succès

- Dans `src/server/index.js`, ajoutez les lignes suivantes :

```js
import { helloEndpointRoute } from '../shared/routes'

// [Au-dessous de app.get('/')...]

app.get(helloEndpointRoute(), (req, res) => {
  res.json({ serverMessage: `Hello from the server! (received ${req.params.num})` })
})
```

### Nouveaux containers

- Créez un fichier `src/client/container/hello-async-button.js` contenant :

```js
// @flow

import { connect } from 'react-redux'

import { sayHelloAsync } from '../action/hello'
import Button from '../component/button'

const mapStateToProps = () => ({
  label: 'Say hello asynchronously and send 1234',
})

const mapDispatchToProps = dispatch => ({
  handleClick: () => { dispatch(sayHelloAsync(1234)) },
})

export default connect(mapStateToProps, mapDispatchToProps)(Button)
```

Afin de démontrer comment vous passeriez un paramètre à votre appel asynchrone et pour garder les choses simples, nous allons coder en dur la valeur `1234` ici. Typiquement, cette valeur pourrait venir d'un champ de formulaire rempli par l'utilisateur.

- Créez un fichier `src/client/container/message-async.js` contenant :

```js
// @flow

import { connect } from 'react-redux'

import MessageAsync from '../component/message'

const mapStateToProps = state => ({
  message: state.hello.get('messageAsync'),
})

export default connect(mapStateToProps)(MessageAsync)
```

Vous pouvez voir que dans ce container, nous faisons référence à une propriété `messageAsync`, que nous allons bientôt ajouter à notre reducer.

Ce dont nous avons besoin maintenant, c'est de créer l'action `sayHelloAsync`.

### Fetch

> :bulb: **[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)** est une fonction JavaScript standardisée pour faire des appels asynchrones inspirée par les méthodes AJAX de jQuery.

Nous allons utiliser `fetch` pour faire des appels au serveur depuis le client. `fetch` n'est pas encore supporté par tous les navigateurs, donc nous allons avoir besoin d'un polyfill (c'est-à-dire un ensemble de fonctions permettant de simuler une ou des fonctionnalités qui ne sont pas nativement disponibles dans le navigateur).

 `isomorphic-fetch` est un polyfill qui fait fonctionner `fetch`  de façon cross-browsers et dans Node aussi ! :ok_hand:

- Lancez `yarn add isomorphic-fetch`

Puisque nous utilisons `eslint-plugin-compat`, nous allons avoir besoin d'indiquer que nous utilisons un polyfill pour `fetch`, histoire de ne pas recevoir des warning à ce sujet.

- Ajoutez les lignes suivantes à votre fichier `.eslintrc.json` :

```json
"settings": {
  "polyfills": ["fetch"]
},
```

### 3 actions asynchrones

`sayHelloAsync` ne va pas être une action normale. Les actions asynchrones sont le plus souvent séparées en 3 actions qui déclenchent 3 states différents: une action *requête* ou *chargement* (request ou loading :uk:), une action *succès* (success :uk:) et une action échec (failure :uk:)

- Éditez le fichier `src/client/action/hello.js` comme ceci :

```js
// @flow

import 'isomorphic-fetch'

import { createAction } from 'redux-actions'
import { helloEndpointRoute } from '../../shared/routes'

export const SAY_HELLO = 'SAY_HELLO'
export const SAY_HELLO_ASYNC_REQUEST = 'SAY_HELLO_ASYNC_REQUEST'
export const SAY_HELLO_ASYNC_SUCCESS = 'SAY_HELLO_ASYNC_SUCCESS'
export const SAY_HELLO_ASYNC_FAILURE = 'SAY_HELLO_ASYNC_FAILURE'

export const sayHello = createAction(SAY_HELLO)
export const sayHelloAsyncRequest = createAction(SAY_HELLO_ASYNC_REQUEST)
export const sayHelloAsyncSuccess = createAction(SAY_HELLO_ASYNC_SUCCESS)
export const sayHelloAsyncFailure = createAction(SAY_HELLO_ASYNC_FAILURE)

export const sayHelloAsync = (num: number) => (dispatch: Function) => {
  dispatch(sayHelloAsyncRequest())
  return fetch(helloEndpointRoute(num), { method: 'GET' })
    .then((res) => {
      if (!res.ok) throw Error(res.statusText)
      return res.json()
    })
    .then((data) => {
      if (!data.serverMessage) throw Error('No message received')
      dispatch(sayHelloAsyncSuccess(data.serverMessage))
    })
    .catch(() => {
      dispatch(sayHelloAsyncFailure())
    })
}
```

Au lieu de retourner une action, `sayHelloAsync` retourne une fonction qui lance l'appel `fetch`. `fetch` retourne une `Promise`, que nous utilisons pour *expédier* différentes actions selon l'état actuel de notre appel asynchrone.

### 3 gestionnaires d'actions asynchrones

Gérons ces différentes actions dans `src/client/reducer/hello.js`:

```js
// @flow

import Immutable from 'immutable'
import type { fromJS as Immut } from 'immutable'

import {
  SAY_HELLO,
  SAY_HELLO_ASYNC_REQUEST,
  SAY_HELLO_ASYNC_SUCCESS,
  SAY_HELLO_ASYNC_FAILURE,
} from '../action/hello'

const initialState = Immutable.fromJS({
  message: 'Initial reducer message',
  messageAsync: 'Initial reducer message for async call',
})

const helloReducer = (state: Immut = initialState, action: { type: string, payload: any }) => {
  switch (action.type) {
    case SAY_HELLO:
      return state.set('message', action.payload)
    case SAY_HELLO_ASYNC_REQUEST:
      return state.set('messageAsync', 'Loading...')
    case SAY_HELLO_ASYNC_SUCCESS:
      return state.set('messageAsync', action.payload)
    case SAY_HELLO_ASYNC_FAILURE:
      return state.set('messageAsync', 'No message received, please check your connection')
    default:
      return state
  }
}

export default helloReducer
```

Nous avons ajouté un nouveau champ dans notre store, et nous le modifions avec différents messages selon l'action que nous recevons. Pendant `SAY_HELLO_ASYNC_REQUEST`, nous montrons `Loading...`. `SAY_HELLO_ASYNC_SUCCESS` modifie `messageAsync` d'une façon similaire à comment `SAY_HELLO` modifie `message`. `SAY_HELLO_ASYNC_FAILURE` donne un message d'erreur.


### Redux-thunk

Dans `src/client/action/hello.js`, nous avons créé `sayHelloAsync`, un créateur d'actions qui retourne une fonction. En fait, ce n'est pas une fonctionnalité qui est nativement supportée par Redux. Afin de pouvoir effectuer ces actions asynchrones nous avons besoin d'étendre la fonctionnalité de Redux avec le *middleware* `redux-thunk`.

- Lancez `yarn add redux-thunk`

- Modifier votre fichier `src/client/index.jsx` comme ceci :

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'
import { Provider } from 'react-redux'
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunkMiddleware from 'redux-thunk'

import App from './app'
import helloReducer from './reducer/hello'
import { APP_CONTAINER_SELECTOR } from '../shared/config'
import { isProd } from '../shared/util'

// eslint-disable-next-line no-underscore-dangle
const composeEnhancers = (isProd ? null : window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__) || compose

const store = createStore(combineReducers({ hello: helloReducer }),
  composeEnhancers(applyMiddleware(thunkMiddleware)))

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = (AppComponent, reduxStore) =>
  <Provider store={reduxStore}>
    <AppContainer>
      <AppComponent />
    </AppContainer>
  </Provider>

ReactDOM.render(wrapApp(App, store), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp, store), rootEl)
  })
}
```

Ici, nous passons`redux-thunk` à la fonction `applyMiddleware` de Redux. Afin que les outils de développement Redux continuent de fonctionner, nous avons aussi besoin d'utiliser la fonction `compose` de Redux. Ne vous en faites pas trop à propos de cette partie, retenez juste qu'on améliore Redux avec `redux-thunk`.

- Modifiez le fichier `src/client/app.jsx` comme ceci :

```js
// @flow

import React from 'react'
import HelloButton from './container/hello-button'
import HelloAsyncButton from './container/hello-async-button'
import Message from './container/message'
import MessageAsync from './container/message-async'
import { APP_NAME } from '../shared/config'

const App = () =>
  <div>
    <h1>{APP_NAME}</h1>
    <Message />
    <HelloButton />
    <MessageAsync />
    <HelloAsyncButton />
  </div>

export default App
```

:checkered_flag: Lancez `yarn start` et `yarn dev:wds` et vous devriez maintenant être capable de cliquer sur le bouton "Say hello asynchronously and send 1234" et de récupérer le message correspondant depuis le serveur ! Puisque vous travaillez en local, l'appel est instantané, mais si vous ouvrez les outils de développement Redux, vous remarquerez que chaque clic déclenche à la fois `SAY_HELLO_ASYNC_REQUEST` et `SAY_HELLO_ASYNC_SUCCESS`, ce qui fait que le message passe par le state intermédiaire `Loading...` comme on l'attendait.

Vous pouvez vous félicitez, c'était une section intense :tada: :clap: ! Mais ajoutons quelques tests :wink:

## Tests

Dans cette section, nous allons tester nos actions et notre reducer. Commençons avec les actions.

Afin d'isoler la logique spécifique au fichier `action/hello.js`, on va se moquer des choses qui ne le concerne pas, ainsi que cette requête AJAX `fetch` qui ne devrait pas déclencher un véritable appel AJAX dans nos tests.

- Lancez `yarn add --dev redux-mock-store fetch-mock`

- Créez un fichier `src/client/action/hello.test.js` contenant :

```js
import fetchMock from 'fetch-mock'
import configureMockStore from 'redux-mock-store'
import thunkMiddleware from 'redux-thunk'

import {
  sayHelloAsync,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from './hello'

import { helloEndpointRoute } from '../../shared/routes'

const mockStore = configureMockStore([thunkMiddleware])

afterEach(() => {
  fetchMock.restore()
})

test('sayHelloAsync success', () => {
  fetchMock.get(helloEndpointRoute(666), { serverMessage: 'Async hello success' })
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncSuccess('Async hello success'),
      ])
    })
})

test('sayHelloAsync 404', () => {
  fetchMock.get(helloEndpointRoute(666), 404)
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})

test('sayHelloAsync data error', () => {
  fetchMock.get(helloEndpointRoute(666), {})
  const store = mockStore()
  return store.dispatch(sayHelloAsync(666))
    .then(() => {
      expect(store.getActions()).toEqual([
        sayHelloAsyncRequest(),
        sayHelloAsyncFailure(),
      ])
    })
})
```
Bien, jetons un oeil à ce qui se passe ici. D'abord, on *mock* (on ignore) le store Redux en utilisant `const mockStore = configureMockStore([thunkMiddleware])`. En faisant cela, on peut expédier des actions sans qu'ils déclenchent la logique du reducer. Pour chaque test, on *mock* `fetch` en utilisant `fetchMock.get()` et le faisons retourner ce que nous voulons. En fait, ce qu'on teste avec `expect()`, c'est quelles séries d'actions ont été expédiées par le store, grâce à la fonction `store.getActions()` de `redux-mock-store`. Après chaque test, on restaure le comportement normal de `fetch` avec `fetchMock.restore()`.

Et maintenant, testons notre reducer (ce qui est beaucoup plus simple) :

- Créez un fichier `src/client/reducer/hello.test.js` contenant :

```js
import {
  sayHello,
  sayHelloAsyncRequest,
  sayHelloAsyncSuccess,
  sayHelloAsyncFailure,
} from '../action/hello'

import helloReducer from './hello'

let helloState

beforeEach(() => {
  helloState = helloReducer(undefined, {})
})

test('handle default', () => {
  expect(helloState.get('message')).toBe('Initial reducer message')
  expect(helloState.get('messageAsync')).toBe('Initial reducer message for async call')
})

test('handle SAY_HELLO', () => {
  helloState = helloReducer(helloState, sayHello('Test'))
  expect(helloState.get('message')).toBe('Test')
})

test('handle SAY_HELLO_ASYNC_REQUEST', () => {
  helloState = helloReducer(helloState, sayHelloAsyncRequest())
  expect(helloState.get('messageAsync')).toBe('Loading...')
})

test('handle SAY_HELLO_ASYNC_SUCCESS', () => {
  helloState = helloReducer(helloState, sayHelloAsyncSuccess('Test async'))
  expect(helloState.get('messageAsync')).toBe('Test async')
})

test('handle SAY_HELLO_ASYNC_FAILURE', () => {
  helloState = helloReducer(helloState, sayHelloAsyncFailure())
  expect(helloState.get('messageAsync')).toBe('No message received, please check your connection')
})
```

Avant chaque test, on initialise `helloState` avec le résultat par défaut de notre reducer (le cas `default` de notre `switch` dans le reducer, qui retourne `initialState`). Les tests deviennent alors très explicites, on s'assure juste que le reducer modifie `message` et `messageAsync` correctement selon l'action qu'il reçoit.

:checkered_flag: Lancez `yarn test`. Tout devrait être vert !

Prochaine: [06 - React Router, Server-Side Rendering, Helmet](06-react-router-ssr-helmet.md#readme)

Retourner à la [section précédente](04-webpack-react-hmr.md#readme) ou au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).
