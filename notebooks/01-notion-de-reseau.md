---
celltoolbar: Diaporama
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
nbTranslate:
  displayLangs: ['*']
  hotkey: alt-t
  langInMainMenu: true
  sourceLang: en
  targetLang: fr
  useGoogleTranslate: true
rise:
  autolaunch: true
---

+++ {"slideshow": {"slide_type": "slide"}}

# Notion de réseau 

Dans ce notebook nous allons voir ce qu'est un réseau et nous allons surtout voir qu'il n'y a rien de mystique c'est même relativement bête en fait.

+++ {"slideshow": {"slide_type": "slide"}}

## Principe de base

+++ {"slideshow": {"slide_type": "subslide"}}

### Infrastructure 

Tout d'abord un réseau c'est quoi ? Et bien c'est une **infrastructure** que l'on utilise pour faire transiter des données. 

Dans sa version la plus élémentaire qui soit un réseau est composé de deux appareils reliés entre eux par un câble réseau par exemple. 

Le point important là-dedans c'est qu’un appareil connecté au réseau doit posséder une interface réseau, un composant capable de communiquer c'est-à-dire d'envoyer et recevoir un signal. 

Par exemple votre ordinateur portable possède deux interfaces réseau : la prise RJ45 et la carte wifi.  Le signal qui transite par l'interface réseau est un signal binaire. 

**L'appareil en lui-même n'a pas besoin de connaître la signification de ce signal, car c'est un programme tournant derrière l'interface réseau qui se chargera de traiter le signal en question.**

+++ {"slideshow": {"slide_type": "subslide"}}

Cette aptitude à envoyer et recevoir un signal binaire via une interface réseau constitue la première couche du modèle OSI. 

Le modèle OSI, pour Open System Interconnection, est une norme mise en place par le commité ISO en 1984 afin de standardiser les communications entre appareils sur un réseau. Ce modèle OSI établit notamment 7 couches qui caractérisent les normes communes devant être suivies par tout appareil devant se connecter à un réseau.

+++ {"slideshow": {"slide_type": "subslide"}}

Ces 7 couches sont les suivantes : 

1. La couche « physique » est chargée de la transmission effective des signaux entre les interlocuteurs. Son service est limité à l'émission et la réception d'un bit ou d'un train de bit continu (notamment pour les supports synchrones (concentrateur)).
1. La couche « liaison de données » gère les communications entre 2 machines directement connectées entre elles, ou connectées à un équipement qui émule une connexion directe (commutateur).
1. La couche « réseau » gère les communications de proche en proche, généralement entre machines : routage et adressage des paquets.
1. La couche « transport » gère les communications de bout en bout entre processus (programmes en cours d'exécution).
1. La couche « session » gère la synchronisation des échanges et les « transactions », permet l'ouverture et la fermeture de session.
1. La couche « présentation » est chargée du codage des données applicatives, précisément de la conversion entre données manipulées au niveau applicatif et chaînes d'octets effectivement transmises.
1. La couche « application » est le point d'accès aux services réseaux, elle n'a pas de service propre spécifique et entrant dans la portée de la norme.


On peut généralement répartir ces 7 couches en deux groupes les couches 1 à 3 que l'on peut qualifier de couches hardware donc gérées par la machine et le système d'exploitation tandis que les couches 4 à 7 sont plutôt des couches de haut niveau qui seront donc gérées par les applications tournant sur les serveurs. 

Dans la suite de ce cours nous ne détaillerons pas toutes les couches du modèle OSI nous nous contenterons d'en survoler quelques unes.

+++ {"slideshow": {"slide_type": "subslide"}}

### Adressage 

Le principe de l'adressage est d'associer à chaque interface de chaque machine sur un réseau une adresse unique pour permettre les communications. Cette addresse peut être temporaire (définie dynamiquement à la connexion) ou bien fixe (définie par l'administrateur réseau). C'est ce qu'on appelle l'adresse IP, pour *Internet Protocol*. L'adresse IP d'une interface réseau s'écrit comme une combinaison de quatre nombres compris entre 0 et 255. 

<img src="../media/adresseip.png" style="width: 60%">

+++ {"slideshow": {"slide_type": "subslide"}}

**Remarque :** en 2011 est apparu un léger problème technique à savoir **l'épuisement des adresse IP** disponibles... Et oui ça devait arriver un jour. Il a donc été mis en place le protocol IP v6 (l'ancien protocole était le v4). Le principe est simple passer d'une adresse définie sur 32 bits à une adresse sur 128 bits (écrites en haxadecimal) par exemple `2001:0db8:0000:85a3:0000:0000:ac1f:8001`

+++ {"slideshow": {"slide_type": "subslide"}}

Pour une analogie, un peu simpliste certes mais toujours efficace, vous pouvez considérer l'envoi de courrier par La Poste. Si, lorsque vous envoyez une lettre, vous n'indiquez que le nom de la personne à qui vous destinez cette lettre il y a une chance quasi nulle que la lettre arrive à destination. Car le nom n'est pas unique il y a des homonymes. Alors qu'une adresse postale complète elle est unique. Et bien l'adressage sur le réseau c'est le même principe.

+++ {"slideshow": {"slide_type": "subslide"}}

Au détail prêt que connaitre l'IP du serveur ne vous permet pas encore de communiquer avec l'application qui se trouve sur ce serveur. En effet pour cela il vous faut une fois arriver devant le serveur frapper à la bonne porte. Car en effet à tout cela s'ajoute la notion de port, c'est ce qui permet sur un serveur donné de faire tourner (et écouter sur le réseau) différents programmes la distinction se faisant sur le port d'écoute. En gros suivant la porte d'entrée par où on passe on arrive pas sur la même application côté serveur. 

$$ 2^{16} = 65 536\;\;\text{port sur une machine} $$

+++ {"slideshow": {"slide_type": "fragment"}}

Quelques port normalisés : 
* 22 : SSH 
* 25 : SMTP 
* 80 : HTTP 
* 443 : HTTPS
* ...

+++ {"slideshow": {"slide_type": "subslide"}}

Même principe ou presque en fait ... En effet prenons par exemple Internet,  si ce dernier n'était constitué que d'un seul et unique réseau ce serait simple. Mais ce n'est pas le cas et heureusement, pourquoi heureusement ? Je vous laisse réfléchir à la question et on en reparle après le confinement ;) Mais donc Internet ce n'est pas un réseau mais un réseau de réseaux. Et donc suivant ce que l'on fait il faut parfois passer d'un sous-réseau à un autre et pour cela nous avons l'interconnexion de réseaux.

+++ {"slideshow": {"slide_type": "subslide"}}

### Interconnexion 

Considérons un exemple très simple, vous sur votre ordinateur portable, facile à imaginer je sais. Nous allons maintenant analyser deux situations : 

*Situation 1 :* Depuis votre ordinateur connecté en wifi à votre box vous voulez accéder aux films (téléchargés légalement évidemment) stockés sur votre NAS (Network Attached Storage) relié lui en filaire à votre box. Dans ce cas pas d'interconnexion puisque depuis une machine de votre réseau local (le réseau de votre box) vous cherchez à atteindre votre NAS qui est dans le même réseau. 

<img src="../media/local.png" style="width: 40%">

+++ {"slideshow": {"slide_type": "subslide"}}

*Situation 2 :* Depuis votre ordinateur connecté en wifi à votre box vous voulez accéder à un site web quelconque. Dans ce cas le site web que vous recherchez n'est pas dans votre réseau local, votre box s'en rend compte et elle transmet alors votre requête à une passerelle (une machine ayant des interfaces dans plusieurs réseaux distincts) en lui demandant si la destination que vous souhaitez ne serait pas dans son autre réseau. Si c'est le cas c'est gagné, sinon la passerelle va elle même interroger une autre passerelle, ...

<img src="../media/remote.png" style="width: 80%">

+++ {"slideshow": {"slide_type": "subslide"}}

Ce routage se fait notamment en utilisant la fameuse adresse IP (Internet Protocol) qui permet d'adresser des machines dans différents réseaux. 

Donc si l'on résume, l'interconnexion qui constitue en fait la troisième couche du modèle OSI gère trois éléments : 
1. Le routage qui détermine le chemin entre deux machines dans des réseaux différents, chemin passant par les passerelles : ces fameuses machines ayant des interfaces dans deux réseaux distincts.
1. Le relayage qui s'occupe, une fois la route déterminée, de faire transiter l'information de la machine A à la machine B 
1. Le contrôle de flux, une fonctionnalité optionnelle mais néanmoins essentielle qui permet de décongestionner l'ensemble du réseau (au sens large). Un peu le Waze du transit de données.

+++ {"slideshow": {"slide_type": "subslide"}}

### Et les noms de domaines ?

+++ {"slideshow": {"slide_type": "fragment"}}

Vous conviendrez que retenir que une adresse IP n'est pas ce qu'il y a de plus simple ! C'est là qu'interviennent les noms de domaines. Par exemple le domaine `minesparis.psl.eu` correspond à l'adresse ip `77.158.173.58`. C'est quand même plus simple de retenir `minesparis.psl.eu`. Le truc magique qui permet de faire le lien entre une IP et un nom de domaine est ce qu'on appel le DNS *Domain Name System* qui va fournir à votre ordinateur l'adresse IP qui se cache derrière un nom de domaine. Un moyen d'interroger à la main le DNS est d'utiliser la commande nslookup (disponible dans le paquet dnsutils sous debian/ubuntu) 

```
$ nslookup www.wikipedia.fr

Non-authoritative answer:
Name:   www.wikipedia.fr
Address: 51.254.200.228
Name:   www.wikipedia.fr
Address: 2001:41d0:302:2100::9ee
```

+++ {"slideshow": {"slide_type": "slide"}}

## Protocoles bas niveau

+++ {"slideshow": {"slide_type": "subslide"}}

Nous avons donc survolé les 3 premières couches du modèle OSI. Et là vous vous dites c'est bien beau mais je n'ai toujours aucune idée de comment je fais pour communiquer sur le réseau. Et je ne pourrai que confirmer ce que vous pensez. Mais ce n'est pas pour autant une raison pour partir. 

Donc pour rentrer un peu plus dans le concret nous allons maintenant nous attaquer à la couche 4 du modèle OSI à savoir la couche transport. Cette couche est celle qui s'occupe réellement de prendre une information envoyée par la machine A et de l'acheminer jusqu'à la machine B qui la reçoit. Pour réaliser cela il existe deux protocoles le TCP et le UDP.

+++ {"slideshow": {"slide_type": "subslide"}}

### TCP 

Le protocole TCP (Transmission Control Protocol) est **le** protocole historique, qui doit sa longévité par sa robustesse et sa fiabilité. Aujourd'hui lorsque vous naviguez sur le web la plupart des échanges qui ont lieu entre votre navigateur et les sites web sont basés sur du TCP. 

Le principe du TCP est très simple et se décompose en trois étapes: établissement de la connexion, transfert de données et fin de la connexion.

+++ {"slideshow": {"slide_type": "subslide"}}

La connexion d'un client à un serveur TCP se décompose en trois étapes (le *three way handshake*) de la manière suivante : 
* Client : Hello le serveur tu m'entends ?
* Serveur : Oui je t'entends et toi ?
* Client : Oui c'est bon je t'entends

<img src="../media/handshake.png" style="width: 20%">

+++ {"slideshow": {"slide_type": "subslide"}}

Une fois cette phase de connexion faite nous pouvons envoyer un message au serveur qui nous répondra en retour. Nous verrons dans le notebook suivant comment concrètement on peut envoyer un message au serveur et recevoir sa réponse. 

Enfin pour finir il faut fermer la connexion cette étape de clôture se réalise en quatre phases : 

* Client : j'ai fini
* Serveur : Ok c'est noté
* Serveur : moi aussi je n'ai plus rien à te dire
* Client : Ok à la prochaine

<img src="../media/tcpClose.png" style="width: 20%">

+++ {"slideshow": {"slide_type": "subslide"}}

Vous pouvez donc voir qu'avec cette approche la connexion est extrêmement fiable et il y a peu de chances d'avoir des loupés. En revanche cette fiabilité n'est pas gratuite, elle s'accompagne d'un coût en terme d'échanges relativement élevé. C'est pour cela qu'il existe une alternative au TCP.

+++ {"slideshow": {"slide_type": "subslide"}}

### UDP 

Le protocole UDP (User Datagram Protocol) est complémentaire au protocole TCP. En effet nous venons de voir que TCP est utilisé lorsque que l'on souhaite s'assurer que l'échange a bien lieu. Le but du protocole UDP est tout autre, il est fait justement pour le cas où l'arrivée à destination du message n'est pas impérative. L'intérêt et que dans ce cas on peut alors s'affranchir des phases de connexion/déconnexion qui sont coûteuses en échange. 

**On privilégie la vitesse à la robustesse**

+++ {"slideshow": {"slide_type": "subslide"}}

Dans quel cas ce type de protocole est utile ?? 

Beaucoup ... Mais il y a un particulièrement qui vous intéresse c'est le streaming vidéo ! En effet tous les sites de streaming (Youtube, Netflix, Amazon Prime Video, Dailymotion si ça existe encore, ...) utilisent ce principe. 

Lorsque vous regardez une vidéo bien évidemment la vidéo est transférée en même temps que vous la regardez, avec une petite avance de phase. Et donc étant donné la qualité globale des vidéos si au cours du chargement on perd une image ou deux ou un peu de la bande son on ne s'en rendra même pas compte au final. C'est entre autre à cela que sert le protocole UDP.

+++ {"slideshow": {"slide_type": "slide"}}

## Protocoles haut niveau

+++ {"slideshow": {"slide_type": "subslide"}}

Pour finir notre introduction théorique au réseau nous allons faire un bond dans les couches et passer directement à la couche 7 du modèle OSI. Non pas que les couches 5 et 6 soient inutiles mais disons que dans le cadre de ce cours ce n'est pas pertinent de les aborder. 

Et donc la 7ème couche du modèle OSI est celle qui caractérise les protocoles d'accès aux services réseaux. Parmi ces protocoles celui que vous connaissez à coup sûr : le HTTP. Mais ce n'est pas le seul loin de là on trouve par exemple : 

* Des protocoles orientés transfert de fichier : FTP
* Des protocoles orientés messagerie : SMTP, POP, IMAP
* Des protocoles orientés sessions distantes : SSH, TELNET

+++ {"slideshow": {"slide_type": "subslide"}}

### HTTP 

Le HTTP est probablement l'un des protocoles que vous utilisez le plus et cela sans le savoir. En effet il est caché derrière tout affichage de page Web. Le HTTP signifie HyperText Transfer Protocol. Le principe est de définir un syntaxe précise des échanges pouvant s'effectuer entre un serveur HTTP (Apache, nginx, .... ) et un client. En parlant de client, pour information votre navigateur n'est rien de plus qu'un client HTTP haut de gamme. Ces règles spécifient notamment comment demander une ressource à un serveur (page web mais pas que), comment envoyer une nouvelle ressource au serveur...

Nous verrons dans le notebook suivant comment communiquer simplement avec un serveur HTTP. Nous ne nous intéresserons pas du tout sur la manière de récupérer une page web, le navigateur sait le faire mieux que nous. En revanche nous verrons comment récupérer de la donnée sur internet. J'en parlerai plus tard mais c'est dans la donnée disponible qu'est la vrai valeur ajoutée du net pas dans le HTML5 et le CSS même si c'est bien utile je le reconnais.

+++ {"slideshow": {"slide_type": "subslide"}}

### SSH

Un autre protocole de la couche application est le SSH, que vous connaissez peut-être moins mais qui pourtant est très très répandu. SSH signifie Secure SHell. Le principe est de permettre à un utilisateur sur une machine A de se connecter via un terminal à une machine B et ainsi pouvoir travailler sur la machine B comme s'il était en local sur son propre ordinateur. C'est notamment utilisé pour administrer des serveurs qui ne sont pas physiquement en notre présence. L'intérêt majeur du protocole SSH est que toutes les communications sont chiffrées d'un bout à l'autre ce qui garantit une certaine sécurité.

+++ {"slideshow": {"slide_type": "subslide"}}

### WebSocket 

Pour finir cette balade parmi les protocoles de la couche application, nous allons dire deux mots du protocole WebSocket. Websocket, relativement récent (2011) est très particulier en son genre dans le sens où, par rapport à HTTP, il a la particularité de permettre une connexion bi-directionnelle. C'est-à-dire qu'il permet, une fois la connexion établie, au serveur d'envoyer des messages au client sans que ce dernier n'ait rien demandé. Avec WebSocket on sort complètement du schéma où le client envoie un message et le serveur répond. 

L’intérêt d'un tel mode de communication c'est que cela a permis de mettre en place des applications beaucoup plus interactives, puisque le serveur peut envoyer quand cela lui chante des données au client. Nous verrons dans le dernier notebook de cette série comment utiliser les websocket pour faire des applications interactives.
