# 08 - Bootstrap and JSS

Le code de ce chapitre est disponible dans la branche [`master-no-services`](https://github.com/verekia/js-stack-boilerplate/tree/master-no-services) du [repo JS-Stack-Boilerplate](https://github.com/verekia/js-stack-boilerplate).

Bien ! Il est temps de faire une beauté à notre app toute moche :lipstick:. Nous allons utiliser Twitter Bootstrap pour lui appliquer des styles basiques. On ajoutera ensuite une bibliothèque CSS-dans-le-JS pour ajouter du style personnalisé.

## Twitter Bootstrap

> :bulb: **[Twitter Bootstrap](http://getbootstrap.com/)** est une bibliothèque de composants d'interface utilisateur.

Il y a 2 options pour intégrer Bootstrap dans une app React. Les 2 ont leurs avantages et leurs inconvénients :

- la première consiste à utiliser la distribution officielle, **qui utilise jQuery et Tether** pour le comportement de ses composants
- la deuxième repose sur une bibliothèque qui ré-implémente tous les composants Boostrap en React, comme [React-Bootstrap](https://react-bootstrap.github.io/) or [Reactstrap](https://reactstrap.github.io/).

Les bibliothèques tierces fournissent des composants React très pratique qui réduisent drastiquement le code trop long des composants HTML officiels, et s'intègre vraiment bien avec votre base React. Cela étant dit, nous devons admettre que nous sommes assez réticent à utiliser ces bibliothèques, car elles sont toujours *derrière* par rapport aux distributions officiels (et parfois, potentiellement très loin derrière). Aussi, elles ne marchent pas avec les thèmes Bootstrap qui implémentent leur propre JS. C'est un gros pas en arrière quand on sait que l'une des forces principales de Bootstrap est son énorme communauté de designers capable de créer des thèmes magnifiques.

Pour cette raison, nous allons plutôt intégrer la distribution officielle, avec jQuery et Tether. Un des problèmes de cette approche est bien sûr, la taille de notre bundle. Pour votre information, un bundle fait environ 200Ko (zippé), jQuery, Tether, et le JS de Bootstrap' inclus. Nous pensons que c'est assez raisonnable, mais si c'est trop pour vous, vous devriez considérez le fait d'utiliser une autre option pour Bootstrap, voire de ne pas utiliser Bootstrap du tout.

### Le CSS de Bootstrap

- Supprimez le fichier `public/css/style.css`

- Lancez `yarn add bootstrap@4.0.0-alpha.6`

- Copiez le fichier `bootstrap.min.css` et `bootstrap.min.css.map` de `node_modules/bootstrap/dist/css` dans votre dossier `public/css`.

- Éditez `src/server/render-app.jsx` comme ceci :

```html
<link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
```

### Le JS de Bootstrap avec jQuery et Tether

Maintenant que le style de Bootstrap est chargé sur notre page, nous avons besoin du comportement Javascript pour les composants.

- Lancez `yarn add jquery tether`

- Éditez votre fichier `src/client/index.jsx` comme ceci :

```js
import $ from 'jquery'
import Tether from 'tether'

// [right after all your imports]

window.jQuery = $
window.Tether = Tether
require('bootstrap')
```
Ça chargera le code Javascript de Bootstrap.

### Les composants Bootstrap

Bien, c'est l'heure copier/coller un paquet de fichiers ! :smile:

- Éditez votre fichier `src/shared/component/page/hello-async.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import MessageAsync from '../../container/message-async'
import HelloAsyncButton from '../../container/hello-async-button'

const title = 'Async Hello Page'

const HelloAsyncPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello asynchronously' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <MessageAsync />
        <HelloAsyncButton />
      </div>
    </div>
  </div>

export default HelloAsyncPage
```

- Éditez votre fichier `src/shared/component/page/hello.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import Message from '../../container/message'
import HelloButton from '../../container/hello-button'

const title = 'Hello Page'

const HelloPage = () =>
  <div className="container mt-4">
    <Helmet
      title={title}
      meta={[
        { name: 'description', content: 'A page to say hello' },
        { property: 'og:title', content: title },
      ]}
    />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <Message />
        <HelloButton />
      </div>
    </div>
  </div>

export default HelloPage
```

- Éditez votre fichier `src/shared/component/page/home.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'

import ModalExample from '../modal-example'
import { APP_NAME } from '../../config'

const HomePage = () =>
  <div>
    <Helmet
      meta={[
        { name: 'description', content: 'Hello App is an app to say hello' },
        { property: 'og:title', content: APP_NAME },
      ]}
    />
    <div className="jumbotron">
      <div className="container">
        <h1 className="display-3 mb-4">{APP_NAME}</h1>
      </div>
    </div>
    <div className="container">
      <div className="row">
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Bootstrap</h3>
          <p>
            <button type="button" role="button" data-toggle="modal" data-target=".js-modal-example" className="btn btn-primary">Open Modal</button>
          </p>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">JSS (soon)</h3>
        </div>
        <div className="col-md-4 mb-4">
          <h3 className="mb-3">Websockets</h3>
          <p>Open your browser console.</p>
        </div>
      </div>
    </div>
    <ModalExample />
  </div>

export default HomePage
```

- Éditez votre fichier `src/shared/component/page/not-found.jsx` comme ceci :

```js
// @flow

import React from 'react'
import Helmet from 'react-helmet'
import { Link } from 'react-router-dom'
import { HOME_PAGE_ROUTE } from '../../routes'

const title = 'Page Not Found!'

const NotFoundPage = () =>
  <div className="container mt-4">
    <Helmet title={title} />
    <div className="row">
      <div className="col-12">
        <h1>{title}</h1>
        <div><Link to={HOME_PAGE_ROUTE}>Go to the homepage</Link>.</div>
      </div>
    </div>
  </div>

export default NotFoundPage
```

- Éditez votre fichier `src/shared/component/button.jsx` comme ceci :

```js
// [...]
<button
  onClick={handleClick}
  className="btn btn-primary"
  type="button"
  role="button"
>{label}</button>
// [...]
```

- Créez un fichier `src/shared/component/footer.jsx` contenant :

```js
// @flow

import React from 'react'
import { APP_NAME } from '../config'

const Footer = () =>
  <div className="container mt-5">
    <hr />
    <footer>
      <p>© {APP_NAME} 2017</p>
    </footer>
  </div>

export default Footer
```

- Créez un fichier `src/shared/component/modal-example.jsx` contenant :

```js
// @flow

import React from 'react'

const ModalExample = () =>
  <div className="js-modal-example modal fade">
    <div className="modal-dialog">
      <div className="modal-content">
        <div className="modal-header">
          <h5 className="modal-title">Modal title</h5>
          <button type="button" className="close" data-dismiss="modal">×</button>
        </div>
        <div className="modal-body">
          This is a Bootstrap modal. It uses jQuery.
        </div>
        <div className="modal-footer">
          <button type="button" role="button" className="btn btn-primary" data-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>

export default ModalExample
```

- Éditez votre fichier `src/shared/app.jsx` comme ceci :

```js
const App = () =>
  <div style={{ paddingTop: 54 }}>
```

Voici un exemple de *style inline React* (style en ligne React :fr:).

Cela se traduira en `<div style="padding-top:54px;">` dans votre DOM. Nous avons besoin de ce code pour pousser le contenu sous la barre de navigation, mais ce n'est pas ce qui est important ici. [Les styles inline React](https://speakerdeck.com/vjeux/react-css-in-js) sont un super moyen d'isoler le style de votre composant du CSS global, mais tout cela a un coût: vous ne pouvez pas utiliser les fonctionnalités CSS natives comme `:hover`, les media queries, les animations, ou `font-face`. C'est [l'une des raisons](https://github.com/cssinjs/jss/blob/master/docs/benefits.md#compared-to-inline-styles) pour laquelle nous allons intégrer une bibliothèque CSS-dans-le-JS, JSS, qu'on verra plus tard dans ce chapitre.

- Éditez votre fichier `src/shared/component/nav.jsx` comme ceci :

```js
// @flow

import $ from 'jquery'
import React from 'react'
import { Link, NavLink } from 'react-router-dom'
import { APP_NAME } from '../config'
import {
  HOME_PAGE_ROUTE,
  HELLO_PAGE_ROUTE,
  HELLO_ASYNC_PAGE_ROUTE,
  NOT_FOUND_DEMO_PAGE_ROUTE,
} from '../routes'

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

const Nav = () =>
  <nav className="navbar navbar-toggleable-md navbar-inverse fixed-top bg-inverse">
    <button className="navbar-toggler navbar-toggler-right" type="button" role="button" data-toggle="collapse" data-target=".js-navbar-collapse">
      <span className="navbar-toggler-icon" />
    </button>
    <Link to={HOME_PAGE_ROUTE} className="navbar-brand">{APP_NAME}</Link>
    <div className="js-navbar-collapse collapse navbar-collapse">
      <ul className="navbar-nav mr-auto">
        {[
          { route: HOME_PAGE_ROUTE, label: 'Home' },
          { route: HELLO_PAGE_ROUTE, label: 'Say Hello' },
          { route: HELLO_ASYNC_PAGE_ROUTE, label: 'Say Hello Asynchronously' },
          { route: NOT_FOUND_DEMO_PAGE_ROUTE, label: '404 Demo' },
        ].map(link => (
          <li className="nav-item" key={link.route}>
            <NavLink to={link.route} className="nav-link" activeStyle={{ color: 'white' }} exact onClick={handleNavLinkClick}>{link.label}</NavLink>
          </li>
        ))}
      </ul>
    </div>
  </nav>

export default Nav
```
Il y a un truc nouveau ici, `handleNavLinkClick`. Un problème qu'on a recontré en utilisant la `navbar` de Bootstrap dans une SPA (SPA = Single Page Application = Application à Page Unique :fr:), c'est que cliquer sur un lien en mobile n'affiche pas le menu, et ne scroll pas en haut de page. Voilà une super opportunité pour vous montrer un exemple de comment intégrer un peu de jQuery / du code spécifique à Bootstrap dans votre app :

```js
import $ from 'jquery'
// [...]

const handleNavLinkClick = () => {
  $('body').scrollTop(0)
  $('.js-navbar-collapse').collapse('hide')
}

<NavLink /* [...] */ onClick={handleNavLinkClick}>
```

**Remarque**: Ici on a retiré tous les attributs liés à l'accessibilité (comme les `aria`) afin de rendre le code plus lisible *dans le contexte de ce tutoriel*. **Vous devez absolument les remettre**. Référez vous à la documentation de et aux exemples de code pour voir comment les utiliser.

:checkered_flag: Votre devrait maintenant être totalement stylisée avec Bootstrap :sparkles: .

## L'état actuel de CSS

En 2016, la stack moderne typique de JavaScript s'est intallée. Les différentes bibliothèques et outils que ce tutoriel vous a fait installer sont pratiquement les *tous derniers standards de l'industrie* (*tousse - même si tout va être totalement obsolète d'ici un an - tousse* ). Oui, c'est une stack assez complexe à mettre en place, mais au moins, la plus part des développeurs front-end sont d'accord pour dire que c'est vers React-Redux-Webpack que l'on doit se diriger. Maintenant, en ce qui concerne le CSS, on a d'assez mauvaises nouvelles. Rien n'est défini, il n'y a pas de façon standard de faire les choses.

SASS, BEM, SMACSS, SUIT, Bass CSS, React Inline Styles, LESS, Styled Components, CSSX, JSS, Radium, Web Components, CSS Modules, OOCSS, Tachyons, Stylus, Atomic CSS, PostCSS, Aphrodite, React Native for Web, [MaintainableCSS](https://github.com/naomiHauret/maintainablecss.com), [Trowel](https://github.com/Trowel/Trowel) et beaucoup d'autres qu'on ne cite pas, sont toutes des approches différentes ou des outils sur comment faire le travail. Et ils le font tous très bien ! Et c'est ça le problème : il n'y a pas de gagnant qui ressorte du lot, c'est un bazar énorme.

Les cool kids React ont tendance à favoriser les approches CSS-in-JS, React inline styles et les CSS modules, étant donné qu'ils s'intégrent très facilement avec React en résolvent plusieurs [problèmes](https://speakerdeck.com/vjeux/react-css-in-js) avec lesquelles les approches CSS normales ont du mal.

Les CSS Modules fonctionnent bien, mais ils ne tirent pas avantages de la puissance de JavaScript et de toutes ses fonctionnalités sur le CSS. Ils fournissent juste l'encapsulation, ce qui est correct, mais à notre avis, les React inline styles et CSS-in-JS emmènenent CSS à un tout autre niveau. Notre suggestion est d'utiliser les React inline styles pour les styles communs (c'est aussi ce que vous pouvez utiliser dans React Native), et utiliser une bibliothèque CSS-in-JS  pour les trucs comme `:hover` et les media queries.

Il y a des [tonnes de bibliothèques CSS-in-JS](https://github.com/MicheleBertoli/css-in-js). JSS en est une très complète et [performante](https://github.com/cssinjs/jss/blob/master/docs/performance.md).

## JSS

> :bulb: **[JSS](http://cssinjs.org/)** bibliothèque CSS-in-JS qui permet d'écrire du style en Javascript et de l'injecter dans votre app.

Maintenant qu'on a un template de base avec Bootstrap, écrivons un peu de CSS personnalisé. On a mentionné plus tôt que les React inline styles ne pouvait pas gérer les `:hover` et les media queries. Nous allons donc vous en montrer un exemple simple sur la homepage en utilisant JSS. JSS peut être utilisé via `react-jss`, une bibliothèque facile à utiliser avec les composants React.

- Lancez `yarn add react-jss`

Ajoutez les lignes suivantes à votre fichier `.flowconfig`, étant donné que Flow a un [problème](https://github.com/cssinjs/jss/issues/411) avec JSS:

```flowconfig
[ignore]
.*/node_modules/jss/.*
```

### Côté serveur

JSS peut rendre du style sur le serveur pour le rendu initial de la page.

- Ajoutez les constantes suivantes à votre fichier `src/shared/config.js` :

```js
export const JSS_SSR_CLASS = 'jss-ssr'
export const JSS_SSR_SELECTOR = `.${JSS_SSR_CLASS}`
```

- Éditez votre fichier `src/server/render-app.jsx` comme ceci :

```js
import { SheetsRegistry, SheetsRegistryProvider } from 'react-jss'
// [...]
import { APP_CONTAINER_CLASS, JSS_SSR_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
// [...]
const renderApp = (location: string, plainPartialState: ?Object, routerContext: ?Object = {}) => {
  const store = initStore(plainPartialState)
  const sheets = new SheetsRegistry()
  const appHtml = ReactDOMServer.renderToString(
    <Provider store={store}>
      <StaticRouter location={location} context={routerContext}>
        <SheetsRegistryProvider registry={sheets}>
          <App />
        </SheetsRegistryProvider>
      </StaticRouter>
    </Provider>)
  // [...]
      <link rel="stylesheet" href="${STATIC_PATH}/css/bootstrap.min.css">
      <style class="${JSS_SSR_CLASS}">${sheets.toString()}</style>
  // [...]
```

## Côté client

La première chose que le client doit faire après avoir affiché l'app côté client, c'est de se débarassé de tout le style généré côté serveur par JSS.

- Ajoutez les lignes suivantes au fichier `src/client/index.jsx` après les appels `ReactDOM.render` (par exemple, avant `setUpSocket(store)`) :

```js
import { APP_CONTAINER_SELECTOR, JSS_SSR_SELECTOR } from '../shared/config'
// [...]

const jssServerSide = document.querySelector(JSS_SSR_SELECTOR)
// flow-disable-next-line
jssServerSide.parentNode.removeChild(jssServerSide)

setUpSocket(store)
```

Éditez le fichier `src/shared/component/page/home.jsx` comme ceci :

```js
import injectSheet from 'react-jss'
// [...]
const styles = {
  hoverMe: {
    '&:hover': {
      color: 'red',
    },
  },
  '@media (max-width: 800px)': {
    resizeMe: {
      color: 'red',
    },
  },
  specialButton: {
    composes: ['btn', 'btn-primary'],
    backgroundColor: 'limegreen',
  },
}

const HomePage = ({ classes }: { classes: Object }) =>
  // [...]
  <div className="col-md-4 mb-4">
    <h3 className="mb-3">JSS</h3>
    <p className={classes.hoverMe}>Hover me.</p>
    <p className={classes.resizeMe}>Resize the window.</p>
    <button className={classes.specialButton}>Composition</button>
  </div>
  // [...]

export default injectSheet(styles)(HomePage)
```
A la différence des React inline styles, JSS utilise des classes. Vous passez vos styles à `injectSheet` et les classes CSS atterrissent dans les props de votre composant.

:checkered_flag: Lancez `yarn start` et `yarn dev:wds`. Ouvrez la homepage. Affichez le code source de la page (pas dans l'inspecteur) pour voir que les styles JSS sont présents dans le DOM au rendu initial dans l'élément `<style class="jss-ssr">` (seulement sur la page Home). Ils devraient être absents dans l'inspecteur, remplacés par `<style type="text/css" data-jss data-meta="HomePage">`.

**Remarque**: En production, le `data-meta` is obscurci. Parfait ! :ok_hand:

Si vous passez la souris au dessus du label "Hover me", il devrait devenir rouge. Si vous modifiez la taille de la page de votre navigateur pour qu'elle soit inférieure à 800px, le label "Resize your window" devrait devenir rouge. Le bouton vert étend les classe CSS Bootstrap en utilisant la propriété `composes` de JSS.

Prochaine section: [09 - Travis, Coveralls, Heroku](09-travis-coveralls-heroku.md#readme)

Retour à la [section précédente](07-socket-io.md#readme) ou au [sommaire](https://github.com/naomihauret/js-stack-from-scratch#table-of-contents).
