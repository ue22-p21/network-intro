---
celltoolbar: Slideshow
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
rise:
  autolaunch: true
---

+++ {"slideshow": {"slide_type": "slide"}}

# Côté serveur

+++ {"slideshow": {"slide_type": "slide"}}

## Introduction

+++ {"slideshow": {"slide_type": "slide"}}

Dans la partie précédente du cours nous avons donc vu comment mettre en place un client TCP ou HTTP à l'aide de Python. Et nous avons surtout vu comment cela peut nous permettre d'aller recherher de l'information sur le web. Mais il ne s'agit là que du côté lumineux de la force. En effet nous l'avons évoqué au tout début du cours les architectures que l'on étudie sont des architectures client-**serveur**. Nous allons donc maintenant passer du côté obscur et voir l'aspect serveur.

+++ {"slideshow": {"slide_type": "slide"}}

Tout d'abord un serveur c'est quoi ? Dans le contexte qui nous intéresse le serveur est juste un programme tournant en tâche de fond sur une machine connectée à un réseau. Ce serveur passe le plupart de son temps à ne rien faire si ce n'est être en attente que quelqu'un (un client en l'occurence) ne vienne lui parler. Ce n'est que lorsqu'un client commence à lui parler que le serveur se réveille et fait des choses.

+++ {"slideshow": {"slide_type": "slide"}}

Une particularité du serveur et nous verrons dans la suite comment cela est réaliser concrètement, c'est que dans une architecture client-serveur, le client ne se connecte qu'à un seul serveur tandis que le serveur peut-être contacté par des milliers de client simultanément. Prenons par exemple Google, il y a en moyenne 80 000 connexions par seconde au moteur de recherche Google, en d'autre mots 80 000 clients se connectent chaque seconde au serveur google. Ce n'est pas tout à fait vrai car pour pouvoir encaisser une telle charge il n'y a pas qu'un serveur google pour le monde entier. Mais vous voyez l'idée.

+++ {"slideshow": {"slide_type": "slide"}}

## Serveur TCP

+++ {"slideshow": {"slide_type": "slide"}}

### Serveur bas niveau

+++ {"slideshow": {"slide_type": "slide"}}

Pour commencer nous allons voir comment réaliser notre propre seveur TCP. Vous aurez certainement remarqué que le titre c'est serveur **bas niveau** alors pourquoi bas niveau ? Tout simplement parce que nous allons faire comme si Python était un langage pauvre et qu'il fallait que l'on se tape à la main toute la machinerie d'un serveur. Nous verrons juste après que bien évidemment Python étant la 8ème merveille du monde il y a tout ce qu'il faut pour faire ça bien plus facilement. Mais pourquoi faire le bas niveau alors ? Tout simplement pour que vous puissiez appréhender un peu mieux la logique cachée derrière un serveur.

+++ {"slideshow": {"slide_type": "slide"}}

Pour faire notre serveur bas niveau, tout se passe encore une fois à l'aide du module `socket` de Python.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import socket
```

+++ {"slideshow": {"slide_type": "fragment"}}

Il faut alors déterminer où est-ce que notre serveur va écouter. En effet un serveur écoute à un endroit particulier, caractérisé par une adresse IP pour spécifier la machine sur le réseau et un port, le port jouant en quelque sorte le rôle de porte d'entrée sur la machine serveur. Dans l'exemple que nous allons faire ici nous allons faire un serveur locale donc qui écoutera uniquement sur notre ordinateur portable, l'adresse ip est donc **127.0.0.1**. Pour le port là vous avez le choix évitez juste de prendre un port réservé par un autre service. Nous allons prendre ici le port 3010.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
ip = "127.0.0.1"
port = 3010

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((ip, port))
s.listen()
```

+++ {"slideshow": {"slide_type": "subslide"}}

A partir de là notre serveur est à l'écoute. En parallèle faisont une fonction client que l'on utilisera par la suite

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import time
def tcp_client(ip, port, msg):
    time.sleep(1)
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((ip, port))
    s.send( msg.encode() )
    BUFF_SIZE = 1024
    msg = s.recv(BUFF_SIZE)
    print(f"Response : {msg.decode()}")
```

+++ {"slideshow": {"slide_type": "subslide"}}

Nous avons maintenant ce qu'il faut pour faire le test de notre server. Sauf qu'il faut que côté serveur on traite le message que notre client va nous envoyer. Pour cela nous allons utiliser la méthode `accept` du socket serveur.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import threading 
t=threading.Thread(target=tcp_client, args=(ip, port, "Hello server"))
t.start()
conn, addr = s.accept()
print(f"Connexion du client {addr}")
msg = conn.recv(1024).decode()
print(f"Le serveur a recu {msg}")
conn.send(f"Serveur a recu {msg}".encode())
conn.close()
```

+++ {"slideshow": {"slide_type": "subslide"}}

Plusieur remarques. Tout d'abord vous voyez apparaitre le module `threading` et la création d'un `Thread`. Ne vous affolez pas ce n'est pas nécessaire en temps normal. C'est juste nécessaire ici afin de lancer simultanément le serveur et le client dans le même notebook. Du côté serveur on voit donc que la méthode `accept`, qui on le précise au passage est bloquante, retourne un objet de type socket et l'adresse ip du client, qui rien de surprenant, est la même que le serveur. L'objet `socket` ainsi créé côté serveur permet : (i) de recevoir le message émis par le client ; (ii) de le traiter (dans notre cas on ne fait que l'afficher côté serveur) ; (iii) d'envoyer un réponse au serveur. À la fin de l'échange on ferme alors le socket lié à la communication client-serveur.

+++ {"slideshow": {"slide_type": "subslide"}}

Si on synthétise notre serveur dans une seule fonction cela se résume à :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
def tcp_server( ip_server, port_server):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind((ip_server, port_server))
        s.listen()
        
        conn, addr = s.accept()
        print(f"Connexion du client {addr}")
        with conn:
            msg = conn.recv(1024).decode()
            print(f"Le serveur a recu {msg}")
            conn.send(f"Serveur a recu {msg}".encode())
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
client = threading.Thread(target=tcp_client, args=('127.0.0.1', 3011, "Hello TCP Server"))
client.start()
tcp_server("127.0.0.1", 3011)
```

+++ {"slideshow": {"slide_type": "fragment"}}

Tout d'abord on peut voir que dans la fonction `tcp_server` j'ai utilisé le mot clé `with` afin d'avoir un context qui se charge de fermer les sockets automatiquement une fois l'échange terminé. Vous pourriez alors me dire quel intérêt d'avoir un serveur qui accepte une communication et s'éteint. C'est très simple aucun. Mais en l'état ce serveur TCP n'a que peut d'intérêt car il ne gère pas l'aspect multi-client. En effet si deux clients cherchent à se connecter simultanément à ce serveur il va y avoir un blocage.

+++ {"slideshow": {"slide_type": "slide"}}

### Python c'est plus sympa que ça : Serveur haut niveau

+++ {"slideshow": {"slide_type": "slide"}}

Maintenant que nous avons vu comment faire de manière compliquée un serveur TCP pas du tout efficace nous allons nous intéresser à la réalisation d'un serveur TCP efficace et beaucoup plus simple. Pour cela nous allons utiliser le module `socketserver` et plus particulièrement les objets `BaseRequestHandler` et `TCPServer`.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import socketserver
```

+++ {"slideshow": {"slide_type": "fragment"}}

La démarche pour définir votre serveur tcp est alors de définir une classe héritant de `BaseRequestHandler` est surchargeant la méthode `handle`. Cette méthode `handle` est automatiquement appelée dès qu'une connexion avec un client est établie. Pour accéder à la requête reçu par le serveur il suffit d'accéder à l'attribut de la classe `self.request` qui est un objet de type `socket`.

+++ {"slideshow": {"slide_type": "subslide"}}

Par exemple si on reprend l'exemple de la partie précédente l'écriture du serveur se fait de la manière suivante :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
class HelloTCPHandler( socketserver.BaseRequestHandler ):
    def handle(self):
        msg = self.request.recv(1024).decode()
        print(f"Le serveur a recu {msg}")
        self.request.send( f"Serveur a recu {msg}".encode() )
```

+++ {"slideshow": {"slide_type": "subslide"}}

Ainsi nous avons défini notre classe *handler* qui va être chargée de traiter une requète. Pour lancer notre serveur TCP en se basant sur ce *handler* il nous suffit alors d'utiliser l'objet `socketserver.TCPServer`. En parallèle nous allons lancer notre client dans un thread comme précédemment.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
client = threading.Thread(target=tcp_client, args=('127.0.0.1', 3012, "Hello TCP Server"))
client.start()

server = socketserver.TCPServer(('127.0.0.1', 3012), HelloTCPHandler)
server.handle_request()
```

+++ {"slideshow": {"slide_type": "fragment"}}

*Attention* nous avons utilisé ici la méthode `handle_request` qui permet de gérer une connexion client et de fermer le serveur ensuite. C'est uniquement dans le cadre de ce notebook que cette méthode à de l'intérêt afin de ne pas bloquer le notebook. Dans le cas général on utilise à la place la méthode `serve_forever` qui comme son nom le laisse penser s'occupe de lancer le serveur dans un boucle infinie.

+++ {"slideshow": {"slide_type": "subslide"}}

Avec le module `socketserver` nous avons donc créer relativement simplement un serveur TCP qui se charge pour nous des aspects "obscur" de socket et nous permet de nous focaliser sur le traitement de la donnée. Cependant ce serveur en l'état à toujours une limitation .... il ne gère toujours pas le cas de plusieurs clients cherchant à se connecter simultanément au serveur. Mais pas d'inquiétude le module `socketserver` dispose de tout ce qu'il faut pour faire cela. 

Il s'agit de la classe `socketserver.ThreadingTCPServer` qui à chaque nouvelle requète reçu instancie un nouveau thread qui sera chargé de traiter la requête en question sans bloquer l'arrivée de nouvelles requêtes. Cela donne par exemple

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
client = threading.Thread(target=tcp_client, args=('127.0.0.1', 3013, "Hello TCP Server"))
client.start()

server = socketserver.ThreadingTCPServer(('127.0.0.1', 3013), HelloTCPHandler)
server.handle_request()
```

+++ {"slideshow": {"slide_type": "fragment"}}

Et donc tout simplement en quelques lignes nous avons mis en place un serveur TCP capable de gérer plusieurs clients simultanément et où l'on peut très simplement implémenter notre traitement de requête sans se préoccuper des aspects réseau.

+++ {"slideshow": {"slide_type": "slide"}}

## Serveur HTTP

+++ {"slideshow": {"slide_type": "subslide"}}

### Un serveur HTTP à la va vite


Nous allons à présent voir comment mettre en place notre serveur HTTP tout aussi simplement que le serveur TCP précédent.

Tout d'abord rappelons que le principe du HTTP repose sur des commandes "normalisées" GET, POST, DELETE, ... Et donc c'est la réponse à ces différentes commandes que nous allons devoir traiter dans notre serveur HTTP. Cela va s'avérer relativement simple toujours grâce au module `http`.

En effet le module `http` dispose d'un objet `BaseHTTPRequestHandler` qui comme dans le cadre de notre handler TCP ne nécessite qu'un héritage et la surcharge de quelques méthodes bien choisies pour faire le travail.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import http.server 
```

+++ {"slideshow": {"slide_type": "subslide"}}

Les méthodes à surcharger dans notre *handler* ont toutes un nom de la forme `do_<command>` par exemple pour gérer la réponse à une requète GET il faut implémenter la méthode `do_GET`. Voyons tout de suite un exemple. Et pour cela nous allons faire un serveur qui ne fait que renvoyer au client le contenu de la requête.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
class EchoHttpHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        client_addr = self.client_address
        request_path = self.path 
        
        self.send_response(200) 
        self.send_header(b'Content-type', b'text/html')
        self.end_headers()
        self.wfile.write( f"Server receive GET request {request_path} from {client_addr}".encode())
```

+++ {"slideshow": {"slide_type": "subslide"}}

Avant de lancer notre serveur HTTP nous allons comme dans le cas du TCP faire une fonction qui encapsule un client.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import requests
import time
def http_client(ip_server, port_server, cmd, url, data=None, json=None):
    time.sleep(1)
    if cmd=="GET":
        out = requests.get(f"{ip_server}:{port_server}{url}")
    elif cmd=="POST":
        if data:
            out = requests.post(f"{ip_server}:{port_server}{url}", data=data)
        elif json:
            out = requests.post(f"{ip_server}:{port_server}{url}", json=json)
    print( out.content.decode() )
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
import threading

t = threading.Thread(target=http_client, args=("http://127.0.0.1", 3022, "GET", "/fake/url"))
t.start()
server = http.server.HTTPServer(("127.0.0.1", 3022), EchoHttpHandler)
server.handle_request()
```

+++ {"slideshow": {"slide_type": "fragment"}}

On constate alors que notre requête GET est bien arrivée dans notre méthode `do_GET` qui nous a répondu par un echo. Essayons maintenant la même chose avec un POST. Pour cela vous vous en doutez il faut que l'on surcharge la méthode `do_POST` dans notre classe *handler*. L'accès aux données envoyées via le post va se faire via l'attribut `self.rfile` qui est un buffer contenant les données reçues par le serveur. Pour lire ces données il est nécessaire de spécifier la taille en bit à lire dans ce buffer.

+++ {"slideshow": {"slide_type": "subslide"}}

Cette taille est stockée dans le *header* de la requête reçue et est accessible via `self.headers['Content-Length']`. Voici ci-dessous la mise en oeuvre.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import json 
class EchoHttpHandlerWithPOST(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        client_addr = self.client_address
        request_path = self.path 
        
        self.send_response(200) 
        self.send_header(b'Content-type', b'text/html')
        self.end_headers()
        self.wfile.write( f"Server receive GET request {request_path} from {client_addr}".encode())
        
    def do_POST(self):
        client_addr = self.client_address
        request_path = self.path 
        data_size = int(self.headers["Content-Length"])
        self.log_message( f"data_size = {data_size}")
        data = self.rfile.read(data_size).decode()
        
        # = json.loads( data_json )
        self.send_response(200) 
        self.send_header(b'Content-type', b'text/html')
        self.end_headers()
        #self.wfile.write( f"Server receive POST request {request_path} from {client_addr} with data {data}".encode())
        self.wfile.write( f"Server receive POST request {request_path} from {client_addr} with data {data}".encode())
        
        
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
import threading

t = threading.Thread(target=http_client, args=("http://127.0.0.1", 3029, "POST", "/fake/url", {"list":[1,2,3,True], "fake":"A useless string"}, None))
t.start()
server = http.server.HTTPServer(("127.0.0.1", 3029), EchoHttpHandlerWithPOST)
server.handle_request()
server.server_close()
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import threading

t = threading.Thread(target=http_client, args=("http://127.0.0.1", 3029, "POST", "/fake/url", None, {"list":[1,2,3,True], "fake":"A useless string"}))
t.start()
server = http.server.HTTPServer(("127.0.0.1", 3029), EchoHttpHandlerWithPOST)
server.handle_request()
server.server_close()
```

+++ {"slideshow": {"slide_type": "subslide"}}

Et donc on voit sur cet exemple que l'on est ainsi capable de récupérer les données associées à une requête POST pour les traiter comme on le souhaite ensuite. Ah et au fait si on revient sur le traitement de la requête GET. Nous avons vu dans le cours côté client que dans un GET on peut donner des paramètres, que l'on met à la fin de l'url. Je suis sur que vous vous demandez déjà si l'on peut récupérer ces paramètres au niveau de notre serveur. La réponse est bien évidemment oui. Et c'est déjà le cas en fait, reprenons notre exemple :

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import threading

t = threading.Thread(target=http_client, args=("http://127.0.0.1", 3031, "GET", "/fake/url?a=1&b=coucou"))
t.start()
server = http.server.HTTPServer(("127.0.0.1", 3031), EchoHttpHandler)
server.handle_request()
server.server_close()
```

+++ {"slideshow": {"slide_type": "subslide"}}

On voit donc que nos paramètres sont bien présent dans l'url de la requête au niveau du serveur. Là vous devriez me répondre que c'est bien joli mais c'est pas super sympatique de devoir parser à la main l'url pour récupérer les paramèrtes. Je vous dirai alors que : (i) vous avez raison en plus vous feriez des erreurs j'en suis sur ; (ii) bien évidemment il y a déjà ce qu'il faut dans Python ... il s'agit du module `urllib` et plus particulièrement des fonctions `urlparse` et `parse_qs`.

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
url_bidon = "http://127.0.0.1/fake/url?a=1&b=coucou"
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
import urllib

url = urllib.parse.urlparse( url_bidon )
print(url)
```

+++ {"slideshow": {"slide_type": "fragment"}}

On constate donc que l'on a ainsi d'un côté le chemin `url.path` et de l'autre les arguments de la requête `url.query`. En revanche les arguments de la requête sont encore sous la forme d'une chaine de caractère unique ce qui n'est pas terrible je le reconnais. Mais pas d'inquiètude c'est ici qu'intervient la fonction `parse_qs`.

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
urllib.parse.parse_qs(url.query)
```

+++ {"slideshow": {"slide_type": "fragment"}}

Et donc à partir de maintenant vous avez tous les outils de base pour faire des applications réseaux relativement évoluées.

+++ {"slideshow": {"slide_type": "slide"}}

### Une application sympa : woof

+++ {"slideshow": {"slide_type": "subslide"}}

Afin d'illustrer tout ce qu'il est possible de faire avec les quelques éléments que l'on a vu dans ce notebook je vous propose que l'on s'intéresse à l'application woof. Le but de cet application est de permettre le téléchargement de fichier entre deux ordinateurs sans avoir besoin de passer par un service type Dropbox, nextCloud, ... Voici le scénario une personne Jean-Michel veut envoyer un fichier qui est sur son ordinateur à Marcel. Le fichier est trop volumineux pour passer en tant que pièce jointe dans un mail. Donc Jean-Michel va lancer le programme woof avec le nom du fichier que l'on souhaite envoyer à Marcel. Le programme nous donnes alors une url que l'on envoie à Marcel et en accédant à cette url avec son navigateur Marcel peut télécharger le fichier en question.

+++ {"slideshow": {"slide_type": "subslide"}}

La première chose à faire avant même de démarrer un serveur c'est de savoir à quel adresse IP on se trouve, c'est quelle est notre IP sur le réseau. Pour cela il y a un moyen simple il s'agit de se connecter en tant que client à l'une des trois adresses 192.0.2.0 ou 198.51.100.0 ou 203.0.113.0. Il s'agit des trois IP des serveurs TEST-NET-1,2,3 qui ont pour seul et unique but de permettre à un serveur de déterminer son adresse IP sur le réseau. Il suffit donc d'établire une connexion avec l'un de ces trois serveur pour obtenir l'IP de votre machine.

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
import socket 
def find_my_ip(): 
    candidates = []
    for test_ip in ["192.0.2.0", "198.51.100.0", "203.0.113.0"]:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        s.connect((test_ip, 80))
        ip_addr = s.getsockname()[0]
        s.close()
        if ip_addr in candidates:
            return ip_addr
        candidates.append(ip_addr)

    return candidates[0]
```

```{code-cell} ipython3
---
slideshow:
  slide_type: fragment
---
my_ip = find_my_ip()
print(f"My ip address is {my_ip}")
```

+++ {"slideshow": {"slide_type": "subslide"}}

Les plus attentifs d'entre vous auront peut-être remarqué qu'il y a une petite différence par rapport à ce que l'on a pu faire dans la partie client de ce cours. Non vous ne voyez pas... Il s'agit du `socket.SOCK_DGRAM` en effet dans le notebook précédent on utilisait à chaque fois `socket.SOCK_STREAM`. Quelle différence ? Et bien si vous vous rappelez des premiers notebook je vous ai expliqué qu'il y a en fait deux protocoles massivement utilisés TCP et UDP. Et bien le `socket.SOCK_DGRAM` signifie juste que l'on va utiliser le second, UDP. Je vous rappelle que le principe du protocole UDP est qu'il n'y a pas de vérification à la connexion du client. Donc n'essayez pas de vous connecter en tcp à ces trois adresses IP car vous n'auriez alors aucun retour et votre programme resterait bloqué.

+++ {"slideshow": {"slide_type": "subslide"}}

Une fois que l'on connait notre IP on peut mettre en place notre serveur HTTP qui va se charger d'exposer un fichier donné au téléchargement. Pour cela il est donc nécessaire d'écrire notre *handle* HTTP qui doit répondre à une requête GET en renvoyant le fichier.

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
from urllib.parse import urlparse
import pathlib as pl

class FileHTTPRequestHandler(http.server.BaseHTTPRequestHandler):
    filename = None
    def do_GET(self):
        url = urlparse(self.path)
        if pl.Path(url.path[1:]).absolute() != pl.Path(self.filename).absolute():
            ## The requested file is not the one allowed to download 
            txt = """                                                                                                                                                       
                <html>                                                                                                                                                       
                   <head><title>403 Not allowed</title></head>                                                                                                                     
                   <body>403 Not allowed</body>                                                                                                             
                </html>"""
            self.send_response(403)
            self.send_header("Content-Type", "text/html")
            self.send_header("Content-Length", str(len(txt)))
            self.end_headers()
            self.wfile.write(txt.encode())
        else:
            f = pl.Path(self.filename)
            self.send_response(200)
            self.send_header("Content-Type", "application/octet-stream")
            self.send_header("Content-Disposition", f"attachment;filename={f.name}")
            self.send_header("Content-Length",f.stat().st_size)
            self.end_headers()
            self.wfile.write( f.read_bytes() )
            
        return 
```

```{code-cell} ipython3
---
slideshow:
  slide_type: subslide
---
FileHTTPRequestHandler.filename = "00_Introduction.ipynb"

server_port = 3051

print(f"Download link {my_ip}:3051/{FileHTTPRequestHandler.filename}")

server = http.server.ThreadingHTTPServer((my_ip, 3051), FileHTTPRequestHandler)
server.handle_request()
server.server_close()
```

+++ {"slideshow": {"slide_type": "subslide"}}

Une fois le serveur lancé vous pouvez déjà tester que cela fonctionne bien en accédant à l'url depuis votre navigateur. Et dans un second temps vous pouvez essayer d'envoyer l'url à l'un de vos camarade confiné pour voir s'il arrive à télécharger depuis chez lui le document. **Attention** le succès ou non du transfert aves vos camarades dépend de beaucoup de chose (configuration de votre ordinateur, de votre box internet, ...) donc si ça ne marche pas immédiatement ne soyez pas surpris il se peut que vous soyez obligé de chercher un peu du côté de votre infrastructure.

+++ {"slideshow": {"slide_type": "fragment"}}

Bien évidemment le programme woof est un peu plus complexe que cela car il permet en plus de mettre à disposition un dossier, en utilisant des méthodes de compression, et également de faire de l'upload de fichier. Mais vous voyez ici qu'en seulement quelques lignes de Python nous avons pu créer notre propre serveur HTTP nous permettant de partager des documents via le réseau.

+++ {"slideshow": {"slide_type": "fragment"}}

Si vous avez un peu de temps à tuer, je vous invite à  faire votre propre version de woof en repartant de la base que l'on vient de faire ici et en y ajoutant des fonctionnalités : (i) téléchargement de dossier ; (ii) gestion des noms de fichiers bizarre (accent, espace, ...) `urllib.parse.quote` peut vous aider ; (iii) permettre de télécharger plusieurs fois une ressource ; (iv) mettre un nombre de téléchargement max ; ....
