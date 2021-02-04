---
jupytext:
  cell_metadata_filter: all,-hidden,-heading_collapsed,-run_control,-trusted
  notebook_metadata_filter: all,-language_info,-toc,-jupytext.text_representation.jupytext_version,-jupytext.text_representation.format_version
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Introduction aux WebSocket

+++

## Un peu plus de souplesse

+++

Dans tout ce que nous venons de voir il y a un point qui vous a peut-être interpelé c'est la rigidité des communications via réseau. En effet dans tous les cas il y a un protocole bien établi à suivre qui est : 

1. Le client se connecte au serveur
2. Le client envoie un message au serveur
3. Le serveur réceptionne le message et répond au client
4. Le client reçoit la réponse du serveur. 
5. La connexion se ferme 

Plus particulièrement ce qui pourrait vous choquer c'est qu'afin que le serveur puisse envoyer un message au client il est impératif que ce dernier ait d'abord envoyé un message au serveur. On est toujours dans un modèle où le client amorce la discussion et le serveur ne fait que répondre.

+++

Fondamentalement ce n'est pas pénalisant vous allez me dire. Oui mais un peu quand même. Considérons par exemple le cas d'une application de messagerie instantanée, par exemple mattermost (je ne dis pas Slack par pur esprit de contradiction). Le principe est que l'on a un serveur et N clients. Chacun des N clients écrit sur le fil de discussion. Sauf que quand un client écrit les autres n'en savent rien et le serveur ne peut pas leur dire puisqu'il ne peut pas initier la discussion. Donc le seul moyen pour les clients de savoir si quelqu'un a écrit quelque chose est de faire une requête au serveur et il faut faire cette requête a intervalle régulier et relativement cours pour conserver une certaine fluidité dans la discussion. On a donc ici une des limites du modèle de communication traditionnel.

+++

C'est dans ce contexte qu'est apparu le protocole websocket. Le principe est très simple, il s'agit d'établir une connexion bidirectionnelle entre un client et le serveur, on parle de connexion *full-duplex*. Cela permet alors au serveur de pousser des informations vers le client sans que ce dernier n'est rien demandé. Il faut bien voir que le protocole websocket est relativement récent par rapport au HTTP. En effet la première normalisation de websocket date de 2011 tandis que le HTTP remonte lui à 1990 (la meilleure année !!)

+++

## Librairies websocket

+++

Websocket étant un protocole et non pas une librairie il en existe de très nombreuses implémentations dans différents langages. 

Dans le cadre de ce cours nous utiliserons pour le côté serveur le module tornado de Python qui est un framework web Python offrant de nombreuses fonctionnalités notamment un usage intensif de la programmation asynchrone ce qui permet de construire simplement des serveur web pouvant gérer un très grand nombre de connections. 

De la même manière pour le côté serveur nous utiliserons également tornado, mais nous pourrions utiliser un autre module.

+++

Par défaut le module `tornado` n'est pas installé dans vos distribution Python. Pour résoure ce problème il vous 
suffit comme d'habitude de faire 

```bash 
pip install tornado
```

+++

## Côté client

+++

Nous allons tout de suite voir comment les websocket s'utilisent côté client. Mais avant de nous lancer en Python faisons le test en javascript. En effet le websocket est une technologie issue du web donc le langage qui le supporte le plus simplement c'est le javascript.

+++

Pour commencer il nous faut un serveur. Pour cela nous allons lancer en tâche de fond un server websocket écrit avec tornado. Nous aurions pu utiliser un serveur de test disponible sur le net, par exemple `wss://echo.websocket.org` mais cela ne nous permettrait d'illustrer tout ce que l'on souhaite.

```{code-cell}
import subprocess
proc = subprocess.Popen("python ../sandbox/ws_echo_server.py", shell=True)
```

Voici ci dessous comment se fait la création d'un webscoket (côté client) en javascript. On constate qu'il n'y a que quatre méthodes à définir : 

* `onopen` qui est appelée à l'ouverture du websocket 
* `onclose` appelée à la fermeture du websocket
* `onerror` appelée en cas d'évènement imprévu
* `onmessage` qui est la fonction appelée à chaque fois que notre websocket reçoit un message du server

Les trois premières méthodes sont optionnelles, on peut très bien ne pas les définir il y a une implémentation par défaut. En revanche il faut nécessairement définir la méthode `onmessage` car c'est elle qui va vous permettre de définir le comportement que vous souhaitez pour votre programme.

+++

Si vous exécutez la cellule ci-dessous vous allez voir apparaitre des fenêtres (correspondant à l'utilisation de `alert`).

```{code-cell}
%%javascript

var socket = new WebSocket("ws://localhost:3060/ws")
socket.onopen = function(e) {
  alert("[open] Connection established");
  alert("Sending to server");
  socket.send("My name is John");
};

socket.onmessage = function(event) {
  alert(`[message] Data received from server: ${event.data}`);
};

socket.onclose = function(event) {
  if (event.wasClean) {
    alert(`[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`);
  } else {
    // e.g. server process killed or network down
    // event.code is usually 1006 in this case
    alert('[close] Connection died');
  }
};

socket.onerror = function(error) {
  alert(`[error] ${error.message}`);
};
```

Si maintenant on exécute la cellule suivante qui va avoir pour effet d'éteindre le serveur websocket vous allez voir apparaître un nouveau message d'alerte lié au fait que le client est informé que le serveur est éteint.

```{code-cell}
proc.terminate()
```

Nous allons maintenant voir comment dans l'environnement Python il est possible de mettre en place un client WebSocket. Tout d'abord je tiens à préciser qu'il existe plein de module Python différents permettant de faire cela. Nous allons ici le faire avec tornado histoire de ne pas avoir d'autres installation à faire. De plus j'ajouterai qu'autant faire le côté serveur en Python semble naturel et nous verrons plus tard que c'est relativement simple, autant le client websocket est plus généralement fait en javascript, ça arrive que l'on ait besoin dans une application donnée d'avoir un client websocket Python mais dans la majorité des cas on va plutôt utiliser du javascript côté client.

+++

La création du client WebSocket se fait à l'aide de la fonction `websocket_connect` du module `tornado`. Avant de l'utiliser il faut que je vous mettes en garde sur quelques points. Tout d'abord le module tornado est concu afin d'être très performant et pour cela il utilise de la programmation asynchrone. Pour celles et ceux qui sont intéréssé par la programmation asynchrone je vous conseille d'aller faire un tour sur le MOOC Python de l'Inria. Et pour ceux que ça n'intéresse pas j'espère que vous n'êtes pas dans les groupes Python pour les cours d'apprentissage de la programmation car c'est ce que vous allez faire à la dernière séance !! Tout ça pour dire qu'il va y avoir quelques bizarrerie dans la syntaxe des `await` entre autres, non ne vous inquiétez pas le confinement ne m'a pas rendu fou c'est lié à cet aspect asynchrone.

```{code-cell}
from tornado.websocket import websocket_connect
```

```{code-cell}
import subprocess
proc = subprocess.Popen("python ../sandbox/ws_echo_server.py", shell=True)
```

Nous allons tout d'abord définir une fonction `on_message` qui va nous servire de *callback*, c'est-à-dire que cette fonction sera appelée à chaque fois que notre client recevra un message. Et ensuite on peut initialiser notre client

```{code-cell}
def on_message( msg ):
    print(f"[In on message] {msg}")

ws = await websocket_connect("ws://localhost:3060/ws", on_message_callback=on_message)
```

```{code-cell}
print(ws)
```

On constate donc que l'on récupère un objet de type `WebSocketClientConnection` qui est notre client websocket. À partir de là il nous suffit simplement d'utiliser la méthode `write_message` de notre client pour envoyer un message à notre serveur de test qui je le rappelle ne fait qu'un echo.

```{code-cell}
await ws.write_message("coucou")
await ws.write_message("byebye")
await ws.write_message("vive la MMC")
```

On constate donc que l'on obtient bien le résultat attentu.Si maintenant on coupe le serveur websocket nous allons voir que l'on reçoit alors un message sans contenu ce qui nous permet si on le souhaite d'intercepter au niveau du *callback* la coupure du serveur.

```{code-cell}
proc.terminate()
```

À partir de ces quelques éléments nous avons tout ce qu'il nous faut à disposition pour réaliser des applications, côté client exploitant les websockets. Car bien évidemment le contenu du message peut-être bien plus qu'un simple `"coucou"` il peut s'agit d'un ensemble de données sérialisé en json.

+++

## Côté serveur

+++

Maintenant que l'on sait créer un client websocket voyons un peu comment faire le serveur qui va avec. Pour cela le module tornado nous met à disposition tout ce qu'il faut pour qe cela se fasse sans douleur.

+++

La démarche à suivre est très similaire à ce que l'on a fait dans le notebook précédent à savoir créer une classe fille implémentant quelques méthodes pré-définies pour avoir un serveur en quelques lignes. Et donc pour définir notre serveur websocket il nous suffit ici de définir une classe héritant de `tornado.websocket.WebSocketHandler` dans laquelle nous allons être amené à définir les méthodes suivantes : 

* open : qui permet d'effectuer une action à la connection d'un nouveau client. 
* on_close : qui permet d'effectuer une action à la déconnexion d'un client
* on_message : qui permet de traiter un message reçu 

Par exemple l'écriture d'un serveur echo se fait de la manière suivante

```{code-cell}
from tornado.websocket import WebSocketHandler
```

```{code-cell}
class EchoWebSocketHandler(WebSocketHandler):
    def open(self):
        print("Open connection")
        self.write_message("Welcome")
        
    def on_message(self, message):
        self.write_message(u"You said: " + message) 

    def on_close(self):
        print("close connection")
```

Il nous suffit alors d'instancier une `Application` tornado en spécifiant que notre handler websocket est accessible à l'adresse `/ws`.

```{code-cell}
from tornado.web import Application

app = Application([(r'/ws', EchoWebSocketHandler)])
server = app.listen(3080)
```

Créons maintenant un client websocket comme nous l'avons vu précédemment

```{code-cell}
def on_message( msg ):
    print(f"[In on message] {msg}")

ws = await websocket_connect("ws://localhost:3080/ws", on_message_callback=on_message)
```

On constate donc que l'on recoit bien directement le message d'acceuil du server. Nous pouvons alors envoyer divers messages au serveur.

```{code-cell}
await ws.write_message("coucou")
await ws.write_message("comment ça va")
```

Et pour finir nous pouvons fermer la connexion du côté client.

```{code-cell}
ws.close()
```

Ainsi en quelques lignes nous avons pu mettre en place très simplement un client et un server websocket, il ne vous reste plus qu'à laisser libre cours à votre imagination !!

```{code-cell}
server.stop()
```

## Une messagerie instantannée

+++

Afin de donner une illustration un peu plus concrète de l'intérêt des websockets, je vous propose de faire une mini-application de messagerie instantanée.

+++

Pour commencer réalisons le serveur de nortre messagerie. Le rôle de ce serveur est tout simple, il n'est là que pour recevoir un message d'un client et le transmettre à tous les clients connectés. Pour cela nous allons avoir besoin de conserver la liste des tous les clients connectés à notre serveur. On va réaliser d'une manière peu élégante et peu originale en créant une variable globale `cnx` qui va contenir toutes les instances de notre *handler*

```{code-cell}
cnx = []

class MessagerieWebSocketHandler(WebSocketHandler):
    def open(self):
        if self not in cnx:
            cnx.append(self)
            
        for client in cnx:
            client.write_message("New user logged in")
        
    def on_message(self, message):
        for client in cnx:
            client.write_message( message )
            
    def on_close(self):
        print("close connection")
        cnx.remove(self)
        
app = Application([(r'/ws', MessagerieWebSocketHandler)])
server = app.listen(3090)
```

Maintenant que notre serveur est prêt et à l'écoute nous allons réaliser le client. Tout comme le serveur le client n'a pas besoin d'être bien compliqué il est juste là pour envoyer et recevoir les message. Une implémentation possible est la suivante :

```{code-cell}
class Client:
    def __init__(self, ws_url, name):
        self._name = name 
        self._ws_url = ws_url 
        
    async def connect(self):
        self._ws = await websocket_connect(self._ws_url, on_message_callback=self.on_message)
        
    def on_message(self, msg):
        print(f"[{self._name} terminal] {msg}")
        
    async def write_message(self, msg):
        await self._ws.write_message(f"[{self._name}]: {msg}")
        
    def quit(self):
        self._ws.close()
```

```{code-cell}
clientA = Client("ws://localhost:3090/ws", "Toto")
await clientA.connect()
clientB = Client("ws://localhost:3090/ws", "Tutu")
await clientB.connect()
clientC = Client("ws://localhost:3090/ws", "Tata")
await clientC.connect()
```

```{code-cell}
await clientA.write_message("coucou")
```

```{code-cell}
clientA.quit()
```

```{code-cell}
await clientB.write_message("Tu as vu Toto est parti")
```

```{code-cell}
server.stop()
```

Ainsi en quelques lignes nous avons pu d'évelopper un système de messagerie instantanée acceptant autant de client que nécessaire.

+++

Pour ceux qui douteraient encore de l'apport significatif des websocket pour ce genre d'application je vous invite a essayer de refaire l'application de messagerie mais sans websocket, uniquement avec du HTTP. Et une fois que ça marche on en rediscute ;)

+++

## Pour vous occuper

+++

Pour finir sur cette introduction à Websocket je vous propose un petit défi, pour celles et ceux qui ne sont pas débordé par les "impératifs" de la campagne BDE. Avec websocket on a fait une application de messagerie en 15 lignes grosso modo. Je vous propose donc de mettre en place un jeu en ligne, le jeu du morpion ou tic tac toe. Le défi est que vous me fassiez un jeu multi-joueur pouvant géré simultanément différentes parties. Vous êtes libre sur l'implémentation, les plus inventifs sont envouragés à faire une interface via html.

+++

Qui dit défi dit récompense. Si vous faites un jeu que j'estimerai être digne de ce nom, je vous l'installerai sur un serveur pour que vous puissiez y jouer (en dehors des heures de cours évidemment) !

```{code-cell}

```
