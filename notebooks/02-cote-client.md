---
celltoolbar: Slideshow
jupytext:
  cell_metadata_filter: all,-hidden,-heading_collapsed,-run_control,-trusted
  notebook_metadata_filter: all,-language_info,-toc,-jupytext.text_representation.jupytext_version,-jupytext.text_representation.format_version
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
rise:
  autolaunch: true
---

+++ {"slideshow": {"slide_type": "slide"}}

# Côté client

+++ {"slideshow": {"slide_type": "subslide"}}

## Le client en quelques mots

Dans un architecture client-serveur il y a un client et un serveur, facile c'est dans le nom ! Dans ce notebook nous allons donc voir en quoi consiste concrètement un client. Pour cela nous allons voir comment nous pouvons facilement développer notre propre client réseau à l'aide bien évidemment de Python. 

Déjà revenons sur la définition d'un client. Un client ne fait fait que deux choses : (i) établir une connexion avec  un serveur ; (ii) recevoir, traiter et envoyer des données. En réalité sans peut-être le savoir vous utilisez un client tous les jours. Une idée ? Un indice : à l'instant même où vous lisez cette ligne vous utilisez un client !! Et oui votre navigateur internet (Chrome, Firefox, Edge, peu importe) n'est rien de plus qu'un client. Bon ok un client un peu évolué certes puisque disposant d'une interface graphique. Mais ce n'est pourtant rien de plus qu'un client qui se connecte à un serveur (caractérisé par une adresse url) reçoit de se serveur une page web et potentiellement renvoie des données. 

Dans la suite du notebook nous allons donc voir comment mettre en place notre propre client.

+++ {"slideshow": {"slide_type": "subslide"}}

## Un premier échange TCP

+++ {"slideshow": {"slide_type": "subslide"}}

Pour commencer nous allons faire un premier client TCP qui va interroger un serveur. Les informations du serveur sont les suivantes :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
#server_ip   = "54.38.35.132"
#server_port = 3000

server_ip = "127.0.0.1"
server_port = 3000
```

+++ {"slideshow": {"slide_type": "fragment"}}

La connection à ce serveur va se faire à l'aide d'un socket comme nous avons pu le voir dans le précédent notebook. Et donc Python, dans son génie, dispose d'un module s'appelant socket !

Mais avant cela il faut lancer le serveur ;)

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import subprocess
proc = subprocess.Popen("python ../sandbox/server_socket_tcp.py", shell=True)
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import socket 
```

+++ {"slideshow": {"slide_type": "fragment"}}

La connexion se fait alors par la création d'un object `socket.socket` de la manière suivante :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
params = (server_ip, server_port)
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(params)
```

+++ {"slideshow": {"slide_type": "fragment"}}

À partir de là, la connection avec le server est établie et nous pouvons donc communiquer avec ce dernier. La communication est simple elle se fait à l'aide de la méthode `send` du socket pour envoyer des données au serveur et avec la méthode `recv` pour recevoir des données du serveur.

+++ {"slideshow": {"slide_type": "subslide"}}

**Attention** au type de ce que l'on envoie. Il faut que les chaînes de caractère que vous envoyez au serveur soient encodées. Si ce n'est pas le cas vous allez avoir l'erreur suivante :

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
try:
    s.send("Hello server")
except Exception as e:
    print(e.args[0])
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
s.send(b"Hello server") ## ou s.send("Hello server".encode())
```

+++ {"slideshow": {"slide_type": "subslide"}}

La méthode `send` nous renvoie en sortie la taille du message que l'on a envoyé au serveur. À partir de là, il nous faut maintenant recevoir la réponse du serveur. Pour cela nous allons utiliser la méthode recv. Cependant il y a une particularité à la méthode `recv` qui est qu'elle nécessite en argument un entier: la taille du buffer. En effet, le client ne sait pas à l'avance la taille du message qu'il va recevoir du serveur, il est donc nécessaire de pré-allouer une taille de message fixée arbitrairement et de recevoir le message en plusieur morceaux si ce dernier est plus grand que la taille du buffer. Cette réception par morceaux peut se faire de la manière suivante :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
BUFF_SIZE = 1024
a = s.recv(BUFF_SIZE)
msg = a
while a != b"":
    a = s.recv(BUFF_SIZE)
    msg += a
    
print(f"Le client a reçu du server :\n {msg.decode()}")
```

+++ {"slideshow": {"slide_type": "fragment"}}

Et enfin une fois notre échange fini nous pouvons fermer notre client.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
s.close()
```

+++ {"slideshow": {"slide_type": "subslide"}}

Suite à ce premier échange TCP entre notre client et un serveur nous pouvons faire plusieurs remarques. Tout d'abord la première chose que vous pouvez vous dire c'est que l'intéret semble assez limité en fait. En effet l'interaction entre le client et le serveur est assez pauvre puisqu'on ne peut envoyer qu'une chaîne de caractères encodée et ne recevoir qu'une donnée similaire. On est loin du Big Data, Web des données et tous ces termes à la mode. Mais pas de panique, inutile de fermer le notebook en vous disant une fois de plus que je suis un rigolo. Je vais tout de suite vous expliquer comment faire des choses un peu plus fun.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
proc.terminate()
```

+++ {"slideshow": {"slide_type": "slide"}}

## Sérialisation ou comment échanger plus

+++ {"slideshow": {"slide_type": "subslide"}}

Nous allons à présent faire une petite parenthèse pour parler du concept de sérialisation. La sérialisation en informatique consiste en une traduction d'un objet en une suite d'objets élémentaires. Cela étant dit ça ne nous avance pas des masses. En effet à partir de cette définition il y a une infinité de sérialisations possibles. L'intérêt de la sérialisation repose sur le fait qu'il existe des formats prédéfinis de sérialisation rendant cette opération interropérable entre différents programmes et différents environnements.

+++ {"slideshow": {"slide_type": "subslide"}}

### 100 % Python la sérialisation via pickle

La première solution de sérialisation envisageable est d'utiliser le module `pickle` de Python. Ce module est dédié à la sérialisation et désérisalisation d'objets Python. L'avantage c'est qu'il va nous permettre de sérialiser tout ce que l'on veut (y compris des classes, des fonctions,... ). En revanche cette généricité va se faire au détriment de la portabilité, en effet seul Python est capable de désérialiser la donnée. Cela impose donc d'avoir une **infrastructure 100% Python** que ce soit côté serveur et côté client.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import pickle
```

+++ {"slideshow": {"slide_type": "subslide"}}

Considérons pour commencer une liste de données hétérogènes. Avec la fonction `dumps` du module Pickle il est tout à fait possible de convertir cette liste en un objet de type `bytes` pouvant alors être envoyé via notre client/serveur TCP.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
data = [ (0,1,2), True, ["coucou", "byebye"]]
serialized_data = pickle.dumps(data)
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
serialized_data
```

```{raw-cell}
---
slideshow:
  slide_type: fragment
---
La désérialisation se fait alors via la fonction `loads` qui prend en entrée le `bytes` object contenant la donnée sérialisée et vous retourne en sortie votre donnée d'origine. 
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
pickle.loads(serialized_data)
```

+++ {"slideshow": {"slide_type": "subslide"}}

L'intérêt d'utiliser Pickle est que l'on peut envisager des applications où les échanges entre client et serveur portent directement sur des morceaux de code à exécuter. Par exemple on pourrait très bien imaginer une application où le client sur un ordinateur ayant peu de ressources envoie au serveur un ensemble de code Python et de données. Le code est alors exécuté sur le serveur (disposant de ressources bien plus importantes) et ce dernier renvoie le résultat au client pour un affichage par exemple.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
### Côté client 
class Calculator:
    def __init__(self, data):
        self._data = data
        self._result = 0.
        
    def run(self):
        self._result = self._data.mean()
        
    def result(self):
        return self._result
    
import numpy as np
c = Calculator(np.random.rand(100))
print(f"Results = {c.result()}")
serialized_c = pickle.dumps(c)
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
### Côté serveur 
import numpy as np
import pickle
receive_c = pickle.loads( serialized_c )
receive_c.run()
### On re-sérialise le tout et on renvoie au client 
send_c = pickle.dumps( receive_c )
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
### côté client à nouveau 
c_client_after_server = pickle.loads(send_c)
c_client_after_server.result()
```

+++ {"slideshow": {"slide_type": "subslide"}}

Cela permet donc de faire des choses beaucoup plus évoluées que ce que l'on a vu au début de ce notebook !! Mais pourquoi ne pas le faire en vrai, c'est-à-dire avec la communication vers le serveur, vous demandez vous. Parce qu'en fait cette application est le parfait exemple de ce qu'il ne faut surtout **pas faire** ! Pourquoi ? Mais pour la simple et bonne raison que c'est une énorme faille de sécurité pour le serveur. Heu... vous ne voyez vraiment pas ? Et bien si je résume, le client envoie au serveur une instance de classe qui a une méthode `run`. Côté serveur on pourrait donc coder quelque chose du genre :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import pickle
receive_c = pickle.loads( serialized_c )

if hasattr(receive_c, "run"):
    print("Le truc que j'ai reçu a une méthode run donc je l'exécute")
    receive_c.run()
else:
    print("Pas de méthode run, je fais autre chose comme envoyer un message au client pour dire que j'ai pas reçu ce qu'il faut ")
    
```

+++ {"slideshow": {"slide_type": "fragment"}}

Le problème avec ça c'est que du côté serveur on ne sait pas ce qu'il y a dans la méthode `run` et donc ça peut être un bête calcul de moyenne comme illustré dans l'exemple. Ou alors des choses un peu plus méchantes, comme la suppression de toutes les données du serveur, la récupération des données du serveur et leur envoi vers un autre serveur, ... C'est ce que l'on appelle de l'injection de code et c'est jamais joli !! 

C'est donc pour cette raison que je n'ai pas fait cet exemple en vrai car cela aurait ouvert une énorme faille de sécurité sur le serveur.

+++ {"slideshow": {"slide_type": "subslide"}}

### La sérialisation 100% Web, le json

Maintenant que nous avons vu `pickle` nous savons sérialiser des données et donc les envoyer via notre connexion TCP sans problème. La limitation de l'approche `Pickle` repose, comme nous l'avons déjà dit, sur le fait qu'elle n'est compatible qu'avec du Python. Il faut donc que le client et le serveur soient tous deux en Python. Or cela est quand même fâcheux car tout l'intérêt de la coopération par réseau est d'avoir des environnements hétérogènes utilisant divers langages. Donc Pickle ne semble pas être optimal.

+++ {"slideshow": {"slide_type": "subslide"}}

Mais pas de souci il y a une solution. C'est le format de donnée json, pour Javascript Object Notation. Alors oui il y a javascript dans le nom mais non ce n'est pas limité à ce seul langage. Pour la petite histoire le json tire son inspiration des object javascript, d'où son nom, et s'est imposé depuis maintenant un peu plus de 10 ans comme une référence dans le domaine de la sérialisation de donnée orienté Web, mais pas que. On retrouve désormais du json dans de nombreuses applications. L'avantage est qu'il existe des librairies json dans la plupart des langages de programmation ce qui va ainsi nous permettre des architecture hétérogène et une intéropérabilité accrue.

+++ {"slideshow": {"slide_type": "subslide"}}

Le principe du json est très simple, il est constitué d'une arborescence de type clé-valeur et/ou de listes ordonnées permettant de représenter une structure de données. Ce qu'il est important de souligner à ce niveau c'est que les valeurs vont être typées, les types disponibles en json sont les types de base présents dans la quasi-totalité des langages à savoir : 

* Chaîne de caractères : une séquence de N caractères unicode. Elles sont obligatoirement entourées de guillemets ;
* Nombre : un nombre décimal signé. Le json ne fait aucune distinction entre un entier et un flottant ;
* Booléen : true ou false sont utilisés pour définir l'état du booléen ;
* Type null : une valeur vide, utilisant le mot clé null ;
* Tableau.

Contrairement au Pickle on ne peut donc pas sérialiser une classe ou une fonction. Cependant les cinq types énoncés précédemment permettent largement de couvrir l'intégralité des besoins.

+++ {"slideshow": {"slide_type": "subslide"}}

Bien évidemment Python dispose d'un module permettant de travailler avec le format json.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import json 
```

+++ {"slideshow": {"slide_type": "fragment"}}

Si l'on reprend alors l'exemple précédent de la sérialisation d'une liste hétérogène, on obtient :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
data = [ (0,1,2), True, ["coucou", "byebye"]]
data_json = json.dumps( data )
print(data_json)
```

+++ {"slideshow": {"slide_type": "fragment"}}

Mais il n'y a aucune différence !!! Si ! en regardant attentivement vous verrez que : (i) les parenthèses sont devenues crochets car json ne sait pas ce qu'est un tuple il le traduit en liste ; (ii) le `True` est devenu true.  Et surtout si on regarde le type des données :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
print(f"type(data) = {type(data)}")
print(f"type(data_json) = {type(data_json)}")
```

+++ {"slideshow": {"slide_type": "subslide"}}

Notre donnée `data` est donc maintenant représentée sous la forme d'une chaîne de caractères que l'on peut encoder et envoyer à un serveur. Vous pourriez me dire: quel intérêt de faire du json on aurait pu faire la chaîne de caractères nous-même. Oui c'est vrai mais alors vous auriez choisi un formalisme que vous seul auriez compris et vous seriez incapable de communiquer proprement avec un serveur que vous n'auriez pas vous même implémenté.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
json.loads(data_json)
```

+++ {"slideshow": {"slide_type": "subslide"}}

## Retour à notre client TCP


Maintenant que l'on sait comment sérialiser des données, je vous propose un petit exemple de client qui a pour objectif d'intéragir avec un serveur qui va jouer le rôle d'un carnet de contact.

+++ {"slideshow": {"slide_type": "subslide"}}

Première étape: lancer le serveur

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
import subprocess
proc=subprocess.Popen("python ../sandbox/server_contact.py", shell=True)
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
import socket
import json 
class ContactClient:
    def __init__( self, host, port):
        self.__host = host
        self.__port = port 
        
    def connect(self):
        self.__socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.__socket.connect((self.__host, self.__port))
        
    def command(self, cmd, **args):
        
        self.connect()
        data = {"cmd": cmd, "args": args}
        
        to_send = json.dumps(data).encode()
        
        sz = self.__socket.send( to_send )
        
        buff = self.__socket.recv(1024).decode()
        data_srv = buff
        while buff != "":
            buff = self.__socket.recv(1024).decode()
            data_srv += buff

        ret = json.loads(data_srv)
        if ret["status"] is False:
            print("ERROR : " + ret["msg"])
            return 
        else:
            return ret["msg"], ret["data"]        
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
c = ContactClient("localhost", 3001)
c.command("list")
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
c.command("add", name="Basile Marchand", mail="basile.marchand@mines-paristech.fr")
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
c.command("list")
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
proc.terminate()
```

+++ {"slideshow": {"slide_type": "subslide"}}

Ainsi avec une telle approche nous pouvons déjà réaliser des applications client-serveur relativement interactives et permettant une large gamme de possibilités. 

En revanche on commence a voir apparaître une difficulté qui est que pour réaliser notre application contact il est nécessaire de se définir une syntaxe. Dans notre cas cette syntaxe passe par la définition d'une structure avec certains arguments requis. L'avantage de cette approche c'est la modularité, vous pouvez faire tout ce que vous voulez. En revanche l'inconvénient majeur c'est que vous êtes seul au monde, car votre syntaxe n'est pas nécessairement celle de votre voisin(e). Donc dans le cas d'une application client-serveur que vous faites vous-même from scratch et qui ne devra jamais interagir avec le monde extérieur pas de problème. En revanche, si vous souhaitez vous inscrire dans une démarche plus modulaire et interopérable avec le monde extérieur il va nous falloir un peu plus que cela. C'est à cela que sert la couche 7 du modèle OSI, la couche applicative.

+++ {"slideshow": {"slide_type": "slide"}}

### Protocol HTTP ou un peu de normalisation des échanges

+++ {"slideshow": {"slide_type": "subslide"}}

Pour le moment nous avons vu comment initialiser une connexion réseau via un socket et envoyer un message quelconque sous la forme de bytes. Cependant c'est loin d'être suffisant pour se lancer dans le monde obscur du Web. En effet faisons un test, nous allons nous connecter via un socket à la page d'accueil de google et voir ce que l'on peut faire.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import socket 

params = ("www.google.com", 80)
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
client.connect(params)
```

+++ {"slideshow": {"slide_type": "fragment"}}

Ok on est connecté mais qu'est ce qu'on fait du coup ? On peut essayer de lui envoyer un Hello pour voir ce qui se passe.

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
### Du code annexe pour mettre un timeout sur le bloc suivant 
from contextlib import contextmanager
import signal

def raise_timeout(signum, frame):
    raise TimeoutError


@contextmanager
def timeout(time):
    # Register a function to raise a TimeoutError on the signal.                      
    signal.signal(signal.SIGALRM, raise_timeout)
    # Schedule the signal to be sent after ``time``.                                  
    signal.alarm(time)

    try:
        yield
    except TimeoutError:
        print("Raise a TimeOutError")
    finally:
        # Unregister the signal so it won't be triggered                              
        # if the timeout is not reached.                                              
        signal.signal(signal.SIGALRM, signal.SIG_IGN)
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
import sys 
if not sys.platform.startswith("win"):
  with timeout(10):
      client.send(b"Hello Google")
      retour = client.recv(4096)
      print(f"Reponse du server : {retour}")
else:
  print("Invalid cell for windows")
```

+++ {"slideshow": {"slide_type": "subslide"}}

Bon et bien en fait il ne se passe rien, le serveur ne sait pas quoi faire car il ne comprend pas notre requête (il n'est pas très poli) et donc il ne donne aucune réponse. En effet si on veut interroger un site web il faut utiliser le protocole HTTP et celui ci est plutôt strict sur le format des messages.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
# send some data 
request = "GET / HTTP/1.1\r\nHost:%s\r\n\r\n" % params[0]
client.send(request.encode())  
# receive some data 
response = client.recv(100*4096)

    
http_response = response
http_response_len = len(http_response)
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
from IPython.display import HTML

print(response.decode("utf-8", "replace"))
```

+++ {"slideshow": {"slide_type": "subslide"}}

Nous avons donc pu récupérer la page Web de l’accueil google et l'afficher. Mais nous ne pouvons pas aller beaucoup plus loin. En effet pour interagir avec des ressources web via le protocole HTTP il est nécessaire de construire des requêtes HTTP, c'est-à-dire ces fameux messages que l'on envoie comprenant de nombreuses informations additionnelles. De plus vous remarquez sans doute qu'il y a quelques problèmes de rendu dans la page que l'on a affichée. C'est parce que nous avons reçu une page web mais que nous n'avons pas fait la demande pour les feuilles de style (les fichiers css) associés.

+++ {"slideshow": {"slide_type": "subslide"}}

Nous allons donc voir comment générer des requètes HTTP en Python de manière simple. Il existe deux approches possible pour gérer des requêtes HTTP en Python. La première utilise le module `http` tandis que la seconde utilise le module `request`.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import http
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
connection = http.client.HTTPConnection("www.bmarchand.fr")
connection.request("GET", "/")
response = connection.getresponse()
print("Status: {} and reason: {}".format(response.status, response.reason))
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
from IPython.display import HTML

data = response.read()

#HTML(response.read())
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
HTML(data.decode("utf-8", "replace"))
```

+++ {"slideshow": {"slide_type": "subslide"}}

On peut facilement obtenir le même résultat à l'aide du module `requests` de Python.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import requests

out = requests.get("http://bmarchand.fr")
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
out.headers
```

+++ {"slideshow": {"slide_type": "subslide"}}

On peut alors observer un certain nombre de choses. La première c'est que comme dans notre exemple avec le client TCP bas niveau la page n'est pas complètement chargée car il manque les feuilles de styles, les images, ... Et c'est normal, car nous ne les avons pas demandées, en effet quand vous consultez un site web via votre navigateur préféré c'est ce dernier qui s'occupe de parser le fichier html reçu pour regarder s'il n'a pas besoin d'autres fichiers (css, js, ...) et si c'est le cas c'est le navigateur qui fait la requête pour chaque fichier nécessaire. Donc quand vous chargez une page web votre navigateur ne fait pas une seule requête mais il en fait plusieurs dizaines généralement.

+++

#### Les codes de retour

+++

Lorsque l'on fait une requête à un serveur via http/https ce dernier nous renvoie en premier lieu un code de retour. Ces codes sont normalisés. Voici un extrait non complet des codes possibles : 

- 200 : ok tout s'est bien passé
- 301/302 : redirection de la page 
- 401 : il faut s'authentifier 
- 403 : minute papillon tu n'as pas le droit d'accéder à ça ! 
- 404 : ce que tu me demande n'existe pas ! 
- 5XX : la c'est un problème de serveur

+++

Et donc la première chose à faire lorsque vous faites une requête à un serveur c'est de vérifier que le code de retour est bien 200 car sinon pas la peine de continuer !

+++

#### Les requêtes !

+++ {"slideshow": {"slide_type": "subslide"}}

Vous avez peut être remarqué que précédemment nous avons utilisé `requets.get` mais c'est quoi ce `get`. Et bien en fait cela signifie que l'on envoie au serveur une requête de type `GET`. Dans le monde HTTP(S) il existe différents types de requêtes. 

- `GET` : requêtes pour obtenir du serveur une ressource (fichier html/css/js, image, video, données, ...)
- `POST` : requêtes pour envoyer des données au serveur en vu d'un traitement (ajout d'un utilisateur dans une base de donnée, ...)
-  `PATCH` : requêtes pour modifier partiellement une ressource du serveur (mettre à jour l'addresse mail d'un utilisateur dans la base de donnée)
- `DELETE` : requêtes pour supprimer une ressource du serveur (supprimer un commentaire sur un article, ... ) 

Il s'agit là des principaux types de requêtes mais il en existe d'autre, pour la liste complète vous pouvez faire un tour [ici](https://fr.wikipedia.org/wiki/Hypertext_Transfer_Protocol).

+++ {"slideshow": {"slide_type": "subslide"}}

Regardons un peu comment cela fonctionne en utilisant le module `requests`. Pour cela nous allons utiliser le site [http://httpbin.org](http://httpbin.org) qui met à disposition un serveur de test relativement utile.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
get_out = requests.get("http://httpbin.org/get")
print( get_out.content.decode())
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
get_out = requests.get("http://httpbin.org/get?a=10&b=100")
print( get_out.content.decode())
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
post_out = requests.post("http://httpbin.org/post", data={"name": "Basile Marchand", "mail": "basile.marchand@mines-paristech.fr"})
print(post_out.content.decode())
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
delete_out = requests.delete("http://httpbin.org/delete")
print(delete_out.content.decode())
```

+++ {"slideshow": {"slide_type": "subslide"}}

### Un mot sur la notion d'API web

+++ {"slideshow": {"slide_type": "subslide"}}

La question que vous vous posez peut-être c'est pourquoi s’embêter à faire des requêtes HTTP en Python si on n'est même pas capable avec ça de récupérer une page Web correctement. Pour la simple et bonne raison que la page web en elle-même n'a aucune intérêt !  Ce qui compte ce sont les données cachées derrière. Le plus bel exemple pour illustrer cela c'est Google !

+++ {"slideshow": {"slide_type": "subslide"}}

Regardez leur page web [www.google.com](www.google.com) la page web en soit, en plus de ne pas être particulièrement sexy, n'a aucun intérêt. Et pourtant Google aujourd'hui c'est plus de 80.000 employés dans le monde et une capitalisation boursière de 1000 milliards de dollars. Et c'est vrai pour tout, aujourd'hui le trafic internet n'est qu'en très faible partie constitué du transfert de page web. Le plus gros du transfert c'est de la donnée "brute". Par exemple aujourd'hui en France un quart de la bande passante Internet est utilisée pour du Netflix et ce qui consomme tant la dedans ce n'est pas l'envoi de la page Web pour vous connecter à votre compte Netflix, c'est le transfert de votre vidéo en streaming. 
 
Tout ça pour dire que ce qui compte dans le Web ce n'est pas l'emballage mais la donnée. Et cette donnée on y accède grâce à une API.

+++ {"slideshow": {"slide_type": "subslide"}}

Une API, pour Application Programming Interface, c'est ce qui permet de définir comment un programme consommateur va pouvoir exploiter les fonctionnalités données d'un programme fournisseur. La notion d'API n'est pas limitée au domaine du web. Elle existe dans tous les domaines de la programmation.

Dans le domaine particulier du Web l'API se définit en fait à partir d'une URL. En effet l'accès à la ressource se fait en effectuant une requête GET sur un url particulière. 


Considérons par exemple le cas d'un serveur générant des listes de nombres aléatoires à la demande. L'api d'un tel serveur pourrait être 

* `/api/integer` renvoie un nombre aléatoire entier
* `/api/float`   renvoie un nombre aléatoire flottant 
* `/api/integer?n=100` renvoie 100 nombres aléatoires entiers
* ...

+++ {"slideshow": {"slide_type": "subslide"}}

Pour lancer le serveur il faut exécuter la cellule suivante :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import subprocess
proc=subprocess.Popen("python ../sandbox/server_http_random.py", shell=True)
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
out = requests.get("http://localhost:3003/api/random/int")
print( out.json() )
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
out = requests.get("http://localhost:3003/api/random/float")
print( out.json() )
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
out = requests.get("http://localhost:3003/api/random/int?n=20")
print( out.json() )
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
out = requests.get("http://localhost:3003/api/random/int?n=20")
print( out.json() )
```

+++ {"slideshow": {"slide_type": "fragment"}}

Pour finir il faut penser à fermer le serveur en tâche de fond.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
proc.terminate()
```

+++ {"slideshow": {"slide_type": "subslide"}}

Donc pour conclure sur les API il s'agit d'un moyen très simple pour offrir une interface vers des ressources et données distantes. La seule difficulté dans ce domaine c'est la définition et surtout la documentation des API. Donc si vous mettez en place un service Web disposant d'une API et que vous souhaitez ouvrir votre service vers l'extérieur merci de prendre le temps de documenter votre API.

+++ {"slideshow": {"slide_type": "slide"}}

## Application

+++ {"slideshow": {"slide_type": "subslide"}}

Maintenant que vous maîtrisez parfaitement le côté ~lumineux de la force~ client je vous propose un exercice. 

Je vous ai mis en place un serveur minimaliste offrant une API permettant : 
1. Lister l'ensemble des utilisateurs de la base de donnée 
2. Mettre à jour le status d'un utilisateur 
3. Envoyer un message à un utilisateur 
4. Récupérer les messages qui m'ont été envoyés.

+++ {"slideshow": {"slide_type": "subslide"}}

L'idée est alors que vous réalisiez les actions suivantes : 

1. A l'aide d'un programme python : 
    1. faire une requète `GET` permettant de trouver quel est votre ID d'utilisateur 
    2. faire une requète `PATCH` pour mettre à jour votre status 
    3. faire des requètes `GET`/`POST` pour vous envoyer des messages entre vous
2. A l'aide du combo HTML/CSS/JS 
    1. Faire le client web de ce serveur 🤗 !

```{code-cell} ipython3

```
