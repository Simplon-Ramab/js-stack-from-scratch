# 07 - Socket.IO

Le code pour ce chapitre est disponible [ici](https://github.com/verekia/js-stack-walkthrough/tree/master/07-socket-io).

> :bulb: **[Socket.IO](https://github.com/socketio/socket.io)** est une bibliothèque qui nous permet de travailler facilement avec les Websockets. Socket.IO fournit une API pratique et des fallbacks (solutions de rechange :fr: ) pour les navigateurs qui ne supportent pas les Websockets.

Dans ce chapitre, nous allons créer un échange de message basique entre le client et le serveur. Afin de ne pas ajouter plus de page et de composant (ce qui ne serait pas en lien avec la fonctionnalité qui nous intéresse dans ce chapitre), cet échange aura lieu dans la console du navigateur. Pas de truc en rapport avec l'UI (User Interface - Interface utilisateur :fr: ) dans ce chapitre :wink: !


- Lancez `yarn add socket.io socket.io-client`

## Côté serveur

- Éditez votre fichier `src/server/index.js` de la manière suivante :

```js
// @flow

import compression from 'compression'
import express from 'express'
import { Server } from 'http'
import socketIO from 'socket.io'

import routing from './routing'
import { WEB_PORT, STATIC_PATH } from '../shared/config'
import { isProd } from '../shared/util'
import setUpSocket from './socket'

const app = express()
// flow-disable-next-line
const http = Server(app)
const io = socketIO(http)
setUpSocket(io)

app.use(compression())
app.use(STATIC_PATH, express.static('dist'))
app.use(STATIC_PATH, express.static('public'))

routing(app)

http.listen(WEB_PORT, () => {
  // eslint-disable-next-line no-console
  console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
    '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
})
```

Notez que pour que Socket.IO fonctionne, vous avez besoin d'utiliser `Server` de `http` pour écouter (`listen`) en cours et pas du `app` de Express. Heureusement, ça ne change pas grand chose à notre code. Tous les détails pour Websocket sont externalisés dans un fichier différent appelé avec `setUpSocket`.

- Ajoutez les constantes suivantes à votre fichier `src/shared/config.js` :

```js
export const IO_CONNECT = 'connect'
export const IO_DISCONNECT = 'disconnect'
export const IO_CLIENT_HELLO = 'IO_CLIENT_HELLO'
export const IO_CLIENT_JOIN_ROOM = 'IO_CLIENT_JOIN_ROOM'
export const IO_SERVER_HELLO = 'IO_SERVER_HELLO'
```

Il y a le *type de messages* que votre client et votre serveur vont échanger. Nous vous suggérons de les préfixer avec soit `IO_CLIENT`, soit `IO_SERVER` afin de savoir clairement *qui* envoie le message. Sinon, les choses pourraient devenir assez confuses une fois que vous aurez beaucoup de messages.

Comme vous pouvez le voir, nous avons un `IO_CLIENT_JOIN_ROOM`, ('client IO vient de rejoindre la salle/room/channel' :fr:) puisque pour le bien de la démo, nos clients rejoindrons une room (comme une chatroom). Les rooms sont utiles pour diffuser des messages à des groupes d'utilisateurs spécifiques.

- Créez un fichier `src/server/socket.js` contenant :

```js
// @flow

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_JOIN_ROOM,
  IO_CLIENT_HELLO,
  IO_SERVER_HELLO,
} from '../shared/config'

/* eslint-disable no-console */
const setUpSocket = (io: Object) => {
  io.on(IO_CONNECT, (socket) => {
    console.log('[socket.io] A client connected.')

    socket.on(IO_CLIENT_JOIN_ROOM, (room) => {
      socket.join(room)
      console.log(`[socket.io] A client joined room ${room}.`)

      io.emit(IO_SERVER_HELLO, 'Hello everyone!')
      io.to(room).emit(IO_SERVER_HELLO, `Hello clients of room ${room}!`)
      socket.emit(IO_SERVER_HELLO, 'Hello you!')
    })

    socket.on(IO_CLIENT_HELLO, (clientMessage) => {
      console.log(`[socket.io] Client: ${clientMessage}`)
    })

    socket.on(IO_DISCONNECT, () => {
      console.log('[socket.io] A client disconnected.')
    })
  })
}
/* eslint-enable no-console */

export default setUpSocket
```

Okay, donc dans ce fichier, nous implémentons *comment notre serveur devrait réagir quand des clients se connectent et lui envoient des messages* :

- Quand le client se connecte, on l'affiche dans la console du serveur et gagnons accès à l'objet  `socket`, que nous pouvons utiliser pour répondre au client.
- Quand un client envoie `IO_CLIENT_JOIN_ROOM`, on le fait rejoindre la `room` qu'il souhaite. Une fois qu'il l'a rejoint, on envoie 3 messages de démo : 1 à chaque utilisateur, 1 à chaque utilisateurs de cette room et enfin, 1 message à ce client seulement.
- Quand le client envoie `IO_CLIENT_HELLO`, on affiche son message dans la console du serveur.
- Quand le client se déconnecte, on l'affiche aussi.

## Côté client

Côté client, nous allons faire quelque chose de très similaire :

- Éditez votre fichier `src/client/index.jsx` comme ceci :

```js
// [...]
import setUpSocket from './socket'

// [at the very end of the file]
setUpSocket(store)
```
Comme vous pouvez le voir, on passe un store Redux à `setUpSocket`. De cette façon, dès qu'un message Websocket venant serveur et qui altère le state Redux du client, on peut `dispatch` (expédier/envoyer/distribuer :fr:) les actions. Mais on ne va pas `dispatch` n'importe quoi dans cet exemple.

- Créez un fichier `src/client/socket.js` contenant :

```js
// @flow

import socketIOClient from 'socket.io-client'

import {
  IO_CONNECT,
  IO_DISCONNECT,
  IO_CLIENT_HELLO,
  IO_CLIENT_JOIN_ROOM,
  IO_SERVER_HELLO,
  } from '../shared/config'

const socket = socketIOClient(window.location.host)

/* eslint-disable no-console */
// eslint-disable-next-line no-unused-vars
const setUpSocket = (store: Object) => {
  socket.on(IO_CONNECT, () => {
    console.log('[socket.io] Connected.')
    socket.emit(IO_CLIENT_JOIN_ROOM, 'hello-1234')
    socket.emit(IO_CLIENT_HELLO, 'Hello!')
  })

  socket.on(IO_SERVER_HELLO, (serverMessage) => {
    console.log(`[socket.io] Server: ${serverMessage}`)
  })

  socket.on(IO_DISCONNECT, () => {
    console.log('[socket.io] Disconnected.')
  })
}
/* eslint-enable no-console */

export default setUpSocket
```
Ce qui se passe ici ne devrait pas trop vous surprendre si vous avez bien compris ce que nous avons fait sur le serveur :

- Dès que le client est connecté, on l'affiche dans la console du navigateur et rejoignons la room `hello-1234` avec un message `IO_CLIENT_JOIN_ROOM`.
- On envoie un `Hello!` avec un message `IO_CLIENT_HELLO`.
- Si le serveur nous envoie un message `IO_SERVER_HELLO`, on l'affiche dans la console du navigateur.
- On affiche aussi n'importe quelle déconnexion.

:checkered_flag: Lancez `yarn start` et `yarn dev:wds` Rendez-vous sur `http://localhost:8000`. Ouvrez la console de votre navigateur et regardez le terminal où tourne votre serveur Express. Vous devriez voir la communication Websocket entre votre serveur et votre client :+1 !

Prochaine section : [08 - Bootstrap, JSS](08-bootstrap-jss.md#readme)

Retour à la [section précédente](06-react-router-ssr-helmet.md#readme) ou au [sommaire](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
