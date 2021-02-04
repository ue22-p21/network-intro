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

# Introduction au framework Flask

+++

Pour finir notre tour d'horizon des programmes coopérants nous allons pour ces deux derniers cours nous intéresser au développement web, via notamment le *framework* Flask. En effet nous l'avons déjà évoqué plusieurs fois dans ce cours, le web symbolise le modèle coopérant le plus courant de nos jours avec un client (votre navigateur) et un serveur. Nous allons donc voir aujourd'hui comment utiliser un framework dédié pour mettre en place un service Web.

+++

## Un framework ??

+++

Première question que vous vous posez certainement, c'est quoi cette histoire de *framework* on a jamais vu ça dans les cours Python ? Alors pour faire simple un *framework* c'est un ensemble de composants logiciel (dans notre cas ce sera de modules Python). Quelle différence  avec une librairie ou un module alors ? (Que vous êtes curieux aujourd'hui !). On peut certainement distinguer de nombreuses différences entre une librairie et un framework mais à mon sens il y a en a une principale à retenir qui est : 

Une librairie (ou un module) offre des fonctionnalitées via des fonctions/classes et vous organisez votre applications en incluant ces fonctions/classes comme vous le souhaitez. Tandis qu'un framework offre un cadre de développement qui vous impose de suivre des patrons de conception (*design pattern*) dans le développement de votre application. Concrètement pour utiliser un framework il faut entrer dans le moule de ce dernier. 

Par exemple, vous avez tous fait du numpy. Et bien quand vous écrivez un programme Python utilisant numpy vous n'avez aucune contrainte venant de l'utilisation de ce module, si ce n'est respecter la syntaxe des fonctions/classes numpy. En d'autres mot l'utilisation de Numpy n'impact pas votre style de programmation. Et bien avec un framework ce ne sera pas pareil il vous faudra rentrer dans le moule du framework donc adapter votre style.

+++

Je vous entends déjà URLer (pardon pour la blague...) au scandale en m'expliquant que vous êtes des esprits libres anticonformistes et ne voulant pas vous plier à des régles et à un style imposé par autrui !

+++

Je peux comprendre vos revendications et convictions personnelles mais avant de jeter votre ordinateur par la fenêtre en jurant que jamais vous ne vous plierez aux standards laissez moi 5 minutes pour essayer de vous convaincre.

+++

Plus sérieusement, nous allons voir dans la suite que le fait d'utiliser un framework pour le développement web va nous permettre de très facilement mettre en place un service Web complet et de bénéficier de plein de petites choses qui vont grandement nous simplifer la vie. en gros on va pouvoir se concentrer sur les choses essentielles, ce qui fera le coeur de notre application et déléguer les aspects techniques pointus au framework.

+++

## Flask  pourquoi ?

+++

Alors comme énoncé dans le titre du notebook nous allons dans la suite de ce cours utiliser le framework Python flask pour faire du développement Web. Flask est un framework web Python développé depuis 2010, donc relativement récent, qui se veut minimaliste et léger.

+++

Alors pour répondre tout de suite à votre question qui est, Flask est-il le seul, non bien évidemment il existe d'autre framework Python pour le développement web. Le plus connu est certainement Django [https://www.djangoproject.com/](https://www.djangoproject.com/) qui est notamment caché derrière quelques "petits" sites que vous connaissez peut-être comme Instagram, Pinterest, ... Ah et pour l'anecdote aussi le site nbhosting que vous avez massivement utilisé pour faire du Python c'est du django caché derrière. 

Autre que Django il existe également tornado que nous avons utilisé dans le notebook précédent. L'objectif premier de tornado n'est pas tant d'offrir un cadre de développement simple que de permettre de déployer des services Web très performant pouvant traiter des milliers de requêtes simultanément, pour cela tornado est développer en grande partie en utilisant des instructions asynchrones. 

Bon et en fait on ne va pas faire toute la liste car ce serait un peu long et inutile, pour les curieux une liste relativement complète est disponible à l'adresse suivante [https://wiki.python.org/moin/WebFrameworks](https://wiki.python.org/moin/WebFrameworks)

+++

Vous vous posez alors certainement la question, pourquoi allons nous voir Flask et pas Django par exemple ? Et bien pour une simple et bonne raison, il ne nous reste plus que 2 séances de 1h30 de cours et Django a plein d'avantages mais à mon sens la simplicité de mise en oeuvre n'en fait pas partie. Alors qu'avec flask nous allons très rapidement pouvoir développer notre propre service web sans trop se faire de noeuds au cerveau. Et puis c'est bien d'être anti-conformiste des fois ;)

+++

## Une première application avec flask

+++

Après tout se blabla, nous allons enfin rentrer dans le vif du sujet à savoir comment j'utilise Flask pour faire une application web. La première étape, installer flask. Comme d'habitude `pip` est votre ami. 

```python 
pip install flask
```

+++

Le premier élément que l'on utilise dans Flask c'est la classe `Flask` qui va représenter l'ensemble de notre application, c'est en quelque sorte le chef d'orchestre.

```{code-cell}
from flask import Flask

app = Flask("main")
```

Il s'agit là du point de départ de notre framework et toute la suite va s'articuler autour de cette instance de la classe `Flask`. Pour faire un exemple de base nous allons juste créer la réponse à la requête du chemin `/` dans une url. Pour cela on définit une fonction `index()` (vous pouvez l'appeler comme vous voulez) et le point important c'est que l'on "décore" cette fonction à l'aide de la méthode `route` de notre instance de `Flask`. C'est ce décorateur qui nous permet d'enregistrer notre fonction `index` comme étant la réponse à une requête HTTP à l'url `/`. Notre fonction va ici envoyer un message brut.

```{code-cell}
@app.route("/")
def index():
    return "Hello Mines ParisTech"
```

Concrètement l'utilisation du décorateur permet d'enregistrer dans l'application Flask la fonction `index` comme étant la fonction fournissant la réponse à une requête GET à l'adresse `/`. Et donc à l'utilisation Flask s'occupe de regarder l'url et si il trouve dans ses routes un chemin concordant avec l'url et bien bingo il appelle la fonction correspondante.

+++

Pour lancer notre serveur il nous suffit alors d'appeler la méthode `run` de la classe `Flask`. Nous n'allons pas le faire dans le notebook, car cela bloquerait la cellule du notebook. Pour lancer le serveur vous avez dans le dossier `flask_demo/demo1` le code associé à cet exemple de base.

+++

Ainsi en 3 lignes nous avons un service web capable de répondre à une requête HTML. Alors ici vous pouvez me répondre, rien de neuf on a fait pareil avec le module `http` deux notebooks avant. Et oui vous avez raison. C'est pourquoi nous allons tout de suite voir un exemple un peu plus intéressant. Considérons par exemple un fichier HTML nécessitant des fichiers css et javascript.

+++

Pour servir un fichier HTML il faut le stocker dans un dossier nommé `templates` et utiliser la fonction `render_template` de Flask. Ca peut vous paraître un peu étrange, c'est normal j'explique un peu plus bas la vrai raison de cela. Pour le moment faite ce que je vous dis !

```{code-cell}
from flask import render_template
```

```{code-cell}
@app.route("/home")
def home():
    return render_template("index.html")
```

A partir de là chaque requête vers l'url `/home` nous renverra alors notre page html `index.html`. L'intérêt majeur d'avoir un framework est que si votre page HTML a elle même beson de fichiers stockés sur le serveur : fichier de style css, script javascript, image, ... Et bien vous n'avez pas à vous en préoccuper, il suffit que vous rangiez tout cela dans un dossier nommé `static` à la racine de votre projet et alors flask s'occupera d'envoyer les fichiers nécessaires au client. Et ça c'est vachemet pratique, si vous ne me croyez pas et vous avez le droit. Je vous invite à écrire votre serveur en utilisant uniquement `http` et d'essayer de servir des fichiers html incluant du css, js, ... Vous allez voir c'est moins sympa qu'avec Flask.  Si vous le faites et que ca fonctionne envoyez le moi que je maltraite votre code pour voir si c'est robuste !

+++

Dans le dossier `flask_demo/demo2` se trouve un exemple d'une application Flask servant un fichier (à l'url `wheel`), fichier nécessitant un css et un js qui se trouvent tout deux dans le dossier `static`. Vous reconnaitrez peut-être l'exemple il a été utilisé dans le cours HTML, CSS, JS.

+++

### Passage de paramètres à votre requête

+++

Nous avons vu dans les cours précédents, qu'un truc sympatique du HTTP c'est de pouvoir passer des paramètres à une requête GET. Je suis sur que vous vous demandez alors comment on peut faire cela dans Flask. Et bien c'est tout simple il y a deux méthodes.

+++

**Méthode 1 : HTTP query**

+++

La première méthode pour récupérer des paramètres dans Flask suit les standards du HTTP et consiste à parser les paramètres de la requête dans l'url. Mais c'est relativement simple car là encore Flask en bon framework qu'il est met à notre disposition tout ce qu'il faut. Il s'agit de l'objet `request` et de son attribut `args`. Il nous suffit alors dans l'une de nos fonctions de l'utiliser de la manière suivante : 

```python 
arg_val = request.args.get("arg_name")
```

La valeur retournée sera alors `None` si `arg_name` n'existe pas dans les paramètres de la requête et la valeur sous forme d'une chaîne de caractère sinon.

+++

**Méthode 2 : Flask developper friendly**

+++

La seconde méthode pour récupérer des paramètres est encore plus simple, elle repose sur le fait que dans les `route` de flask on peut spécifier des "url paramétrées". Par exemple nous pouvons définir une route de la manière suivante :

```{code-cell}
@app.route("/home/<int:user_id>")
def home_uid(user_id):
    ## do something according to user_id value 
    return ""
```

Par rapport à ce que l'on a pu faire précédemment on peut constater deux différences : (i) il y a une syntaxe bizarre dans le chemin donné pour la route avec des `<` et `>`. (ii) la fonction décorée, ici `home_user` prend un argument en entrée.

+++

La syntaxe bizarre à base de `<` et `>` c'est ce qui nous permet d'expliquer à Flask qu'il va y avoir quelque chose derrière `/home/` par exemple `/home/1265`. Et de cette manière flask au moment de la requête va automatiquement récupérer le `1265` et ensuite appeler la fonction `home_uid` en lui donnant en argument la valeur qu'il vient de récupérer à savoir 1265. Je sais c'est impressionant.... Vous remarquez également que l'on a ajouté devant `user_id` le symbole `int:` c'est pour préciser à flask que le quelque chose derrière `/home/` va être un nombre entier et donc la variable `user_id` que l'on récupère dans la fonction `home_uid` est directement un entier, pas besoin de faire une conversion Flask le fait pour vous. Il est vraiment fort ce Flask.

+++

Les types de paramètres disponibles dans Flask avec cette approche sont les suivants : 

* string : pour tout texte sans slash 
* int : valeur entière positive
* float : valeur flottante positive
* path : comme les string mais accepte les slashs

+++

Cette manière vous permet de définir plus simplement des chemin devant impérativement prendre des paramètres d'entrée.

+++

Dans les dossier, `flask_demo/demo3` et `flask_demo/demo4` se trouvent deux exemples d'application Flask prenant des arguments dans certains chemin. Mais attendez cinq minutes avant d'aller les voir, regardez avant la partie suivante sur le moteur de template comme ça vous comprendrez beaucoup mieux ce que ça fait ;)

+++

## Le moteur de template Jinja2

+++

Un point fort de Flask est qu'il dispose d'un moteur de template, nommé Jinja2. Alors c'est quoi un moteur de template ? Alors grosso modo c'est un pseudo langage qui va nous permettre de générer de manière dynamique du code html.

+++

Plus concrêtement, l'intérêt du moteur de template va être d'amener un peu de dynamisme dans notre serveur. Classiquement, dans un site web nous avons les pages web, écrite dans des fichiers HTML dont le contenu est statique et le serveur ne sert qu'à envoyer ces pages web au client (votre navigateur). Le moteur de template va permettre de modifier le contenu de la page HTML avant que le serveur ne l'envoi au client. Par exemple nous pourrions imaginer que si Oasis était bien fait, vous pourriez avoir un message du genre "Bonjour Basile" en haut de la page une fois que vous êtes connecté et si vous vus appelez Basile. Il serait donc nécessaire que le serveur modifie le contenu de la page avant de vous l'envoyer.

+++

Pour faire cela Flask dispose de deux fonctions, utilisant Jinja2, qui sont `render_template` et `render_template_string`. Pour ces deux fonctions il faut les appeler avec un template (fichier html ou string) et les variables de contexte, i.e. les variables qui vont intervenir dans le template. Pour l'utilisation de fichier template il faut impérativement que ces derniers soient situé dans un dossier nommé `templates` à la racine de votre application flask (c'est-à-dire à côté de votre Python principal).

+++

Dans le dossier `flask_demo/demo3/` se trouve un exemple de fichier html templaté pour être modifié à l'appel de la page.

```{code-cell}
!ls flask_demo/demo3/
```

```{code-cell}
!cat flask_demo/demo3/templates/wheel.html
```

Vous voyez donc apparaître certaines lignes qui ne sont pas du html. Il s'agit des balises utilisées par Jinja2 pour détecter les parties du contenu HTML devant être adaptées.

+++

Les principales rêgles syntaxiques de Jinja2 sont les suivantes : 

**Substitution de variables** pour afficher dans le HTML le contenu d'une variable il faut entourer cette dernière par des doubles accolades dans du code HTML. Par exemple : 

```html 
<div> Bonjour {{ name }} </div> 
```

**Test if else** pour choisir d'afficher ou nom une partie de la page HTML vous pouvez utiliser des branchements de type *if*, *else if*, *else*. La syntaxe est la suivante 

```html 
{% if une_condition %}
<div> du html en pagaille </div>
{% elif une_autre_condition %}
<div> un autre fouilli de html </div>
{% else %}
<div> le html par défaut </div>
{% endif %} 
```

*Remarque* le `None` de Python se transforme en `none` dans Jinja2


**Boucle for** vous pouvez également effectuer des boucles *for*, l'intérêt majeur étant l'affichage dynamique de tableau. Les boucles for dans Jinja2 vous permettent d'itérer sur tout objet Python iterable. La syntaxe est la suivante 

```html 
{% for x in ma_liste %}
<div> Iteration {{ x }} </div> 
{% endfor %}
```

+++

Une fois notre page HTML templaté par ce que l'on veut, il faut expliquer à notre application Flask que l'on doit retourner une page templaté. Pour cela on utilise la fonction `render_template`. Si on reprend l'exemple de la demo3 on a :

```{code-cell}
!cat flask_demo/demo3/app.py
```

Vous voyez donc que l'on appelle la fonction `render_template` avec comme premier argument le fichier HTML template et ensuite derrière les arguments nécessaire à la compilation du template. **Attention** il faut nommer les arguments que l'on donne et bien évidemment les noms donnés dans la fonction `render_template` doivent correspondre aux noms de variables utilisés dans le fichier HTML template.

+++

Avec ceci on a déjà largement de quoi faire. En effet avec ces quelques mécanismes, d'apparence simple, on peut très facilement avoir des pages générés de manière dynamique. Bien évidemment je suis sur que certain d'entre vous ce seront alors dit qu'il serait possible avec le mécanisme de template d'avoir un site web complet écrit dans un seul et unique fichier HTML. A coup de if, else if, else ca marche. Oui c'est vrai, pour le sport c'est faisable mais je le déconseille quand même car vous allez vite avoir un html de plusieurs milliers de lignes et je ne sais pas vous mais personnellement je trouve le HTML suffisament moche et illisible comme ça. Pas besoin d'en rajouter une couche ;)

+++

Bien évidemment il ne s'agit là que d'un léger aperçu de ce qu'il est possible de réaliser avec Jinja2. Pour aller plus loin je vous recommande vivement de faire un tour dans le documentation du module [https://jinja.palletsprojects.com/en/2.11.x/](https://jinja.palletsprojects.com/en/2.11.x/) vous verrez alors qu'il ya  bien d'autre fonctionnalités disponibles et très très utiles.

+++

## Flask WTF !!

+++

Alors là vous vous dites, ça y est le confinement a eu raison de lui. Non par WTF je veux dire WTForm le module permettant de gérer facilement les formulaires Web en Python. Par formulaire on entend toute page où l'utilisateur doit saisir des données devant ensuite être envoyées au serveur. Page de connexion, inscription, recherche, ... 

Et donc Flask-WTF est le module qui permet d'intégrer simplement WTForm dans une application flask. Pour l'utiliser il faut comme toujour commencer par l'installer. Pour cela

+++

```bash 
pip install flask-wtf
```

+++

L'utilisation de Flask-WTF se fait en définissant son propre formulaire en créant une classe héritant de la class `FlaskForm`.

```{code-cell}
from flask_wtf import FlaskForm
```

La classe que l'on doit alors créer est relativement minimaliste puisqu'elle n'est constituée que d'attributs qui doivent tous être nécessairement des instances de classes définis dans le module `WTForm`. Ces classes spécifiques permettent de spécifier le type des champs requis pour le formulaire.

Par exemple un formulaire de login pourrait s'écrire de la manière suivante :

```{code-cell}
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    submit = SubmitField('Sign In')
```

Les différents types prédéfinis dans `WTForm` sont les suivants : 
    
* BooleanField : représente un booléen
* DateField : représente une date 
* FileField : pour la sélection de fichier
* MultipleFileField : pour la sélection multiple 
* FloatField 
* IntegerField
* DecimalField
* SelectField : choix parmi une liste d'option 
* SubmitField : le bouton de soumission du formulaire
* PasswordField : champs pour le mot de passe (affiche des étoiles)
* TextAreaField : champs de saisie de texte libre

+++

Bien évidemment vous pouvez également définir vos propre type de champs. Il y a plusieurs intérêt à utiliser flask-WTF. Le premier est que pour chaque champs vous pouvez spécifier des validateurs, c'est-à-dire des objets qui vérifieront que les valeurs que l'utilisateur donne à chaque champs de votre formulaire respectent certains critères. Ci-dessous quelques validateurs disponibles nativement.  

* DataRequired : champs obligatoire
* Email : le champs est une adresse email
* EqualTo : test d'égalité
* NumberRange : valeur numérique dans un intervalle
* Optional : champs optionnel 

Le second avantage à utiliser flask-WTF est que l'on va pouvoir utiliser notre classe, `LoginForm` par exemple, pour générer automatiquement la page html associée. Par exemple pour générer notre page de login nous pourrions écrire le fichier login.html suivante : 

```html 
<html>
<head>
<title> Flask WTF </title>
</head>
<body>

<hr>
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}
        </p>
        <p>{{ form.submit() }}</p>
    </form>
</body>
</html>
```

+++

Le fichier HTML une fois bidouillé par Flask et Jinja2 arrive au niveau de votre navigateur avec le contenu suivant :
    
```html
<head>
<title> Flask WTF </title>
</head>
<body>

<hr>
 <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        <input id="csrf_token" name="csrf_token" type="hidden" value="ImI0ODg5NjE3NzdiYjM5NWJlZWRiYzE3MDlmZjBhNjFkMDhlMjE4M2Ii.Xq_IiQ.GG9q2vWBhqbZGuGGJue2MwDIQwI">
        <p>
            <label for="username">Username</label><br>
            <input id="username" name="username" required size="32" type="text" value="">
        </p>
        <p>
            <label for="password">Password</label><br>
            <input id="password" name="password" required size="32" type="password" value="">
        </p>
        <p><input id="submit" name="submit" type="submit" value="Sign In"></p>
    </form>
</body>
```

+++

Bon et minute culturelle au passage, vus voyez apparaître un machin illisible qui est le `csrf_token`. C'est la méthode `form.hidden_tag` qui a généré cette ligne. Alors d'un point de vue fonctionnelle elle ne sert pas à grand chose mais en revanche d'un point de vue sécurité elle est importante. C'est ce qui va permettre à Flask de se prémunir des attaque de type *cross-site request forgery*. Pour plus d'information à ce sujet Google reste votre ami.

+++

Et enfin le dernier avantage à utiliser ce module est que l'on va pouvoir facilement traiter la soumission du formulaire dans notre application. Par exemple : 

```python 
@app.route("/", methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        "Log in requested for {form.username.data} with passord {form.password.data}")
        ## Add function here to check password
        
        return redirect("/home")
    return render_template("login.html", form=form)
```

Plusieurs remarques à faire à ce niveau: (i) vous voyez apparaître dans le décorateur un attribut `method`, il est nécessaire de le spécifier ici puisque l'on doit traiter la soumission d'un formulaire qui se fait par l'intermediaire du requête POST. Il faut donc que notre route `/` accepte les requêtes GET **et** POST. (ii) On voit que notre classe `LoginForm` nous permet de récupérer les valeurs saisies dans le formulaire et envoyées par le client afin de pouvoir facilement les traiter ensuite, dans le cas présent le traitement serait vérifier si le couple utilisateur/mot de passe est bon ou pas.

+++

Dans le dossier `flask_demo/demo5` vous trouverez les sources de l'application flask avec le formulaire de login. Je vous invite donc à le tester en vrai et le traffiquer selon vos envie, en ajoutant des champs par exemple ! Il y a également dans cet exemple un traitemant un peu plus avancé de la réponse du serveur.

+++

### Une application : résolution d'EDO

+++

Bon alors juste pour le fun, je sais ca n'a pas l'air si fun que ça mais attendez un peu. Je vous propose de prendre 5 minutes pour faire une application Flask destinée à résoudre une équation différentielle ordinaire du premier ordre, vous l'avez déjà tous fait en Python donc vous avez le code sous la main. Il faut juste que vous fassiez un formulaire pour saisir les paramètres de l'EDO et à la soumission du formulaire votre serveur Flask calcul la solution de l'EDO et la renvoie à une page web qui trace une jolie courbe (quelque lignes de javascript suffisent pour tracer une courbe, il y a plein de librairie javascript disponible). 

A vos clavier !

+++

## Flask et les websocket

+++

Pour finir cette très brève introduction à Flask nous allons maintenant voir comment nous pouvons intégrer les websocket dans une application Flask. En effet nous avons vu dans le cours précédent que le protocol WebSocket est une des plus belle invention dans le monde du Web et il serait intéressant que l'on puisse intégrer cela dans notre application flask.

+++

Pas de problème, c'est bien évidemment possible. Il suffit pour cela d'utiliser le module `Flask-SocketIO`. Étape une on installe le module en question. 

```bash 
pip install flask-socketio
```

+++

L'utilisation de websocket avec Flask se fait de manière très simple. Il suffit tout d'abord de créer notre serveur websocket à l'aide de la classe `SocketIO` que l'on attache à notre application Flask.

```{code-cell}
from flask_socketio import SocketIO
socketio = SocketIO(app)
```

A partir de là on retrouve les même mécanismes que Flask, à savoir que pour enregistrer un *handler* dans notre application il nous suffit juste de décorer une fonction à l'aide cette fois ci de `@socketio.on("event name")`. Cela donne par exemple :

```{code-cell}
@socketio.on("message")
def receive_message( msg ):
    ## I do something 
    ## ...
    ### and I send a response if needed
    socketio.emit("message", data_to_send)
```

L'évènement `message` est l'évènement par défaut du protocol websocket mais avec Flask-SocketIO il est tout à fait possible de définir vos propres évènement.

+++

Par exemple reprenons l'exemple du chat temps réel que l'on a abordé dans le notebook précédent. Nous allons facilement pouvoir l'intégrer dans une application Web maintenant. Pour cela il nous suffit des quelques lignes suivantes :

```{code-cell}
from flask import Flask, render_template
from flask_socketio import SocketIO

app = Flask(__name__)
app.config['SECRET_KEY'] = 'it-is-secret'
socketio = SocketIO(app)

@app.route('/')
def sessions():
    return render_template('session.html')

@socketio.on('receive_msg')
def handle_my_custom_event(json, methods=['GET', 'POST']):
    print('received my event: ' + str(json))
    socketio.emit('my response', json)
```

Le serveur étant prêt, oui il n'y avait pas grand chose à faire je vous  l'accorde, il nous fait maintenant faire le client. Pour cela il suffit d'une page html toute simple, avec un formulaire de base. Et les quelques lignes de javascript suivantes pour gérer le websocket. 

```js 
var socket = io.connect('http://' + document.domain + ':' + location.port);

$( 'form' ).on( 'submit', function( e ) {
    e.preventDefault()
    var user_name = $( 'input.username' ).val()
    var user_input = $( 'input.message' ).val()
         socket.emit( 'receive_msg', {
             user_name : user_name,
             message : user_input
         } )
     $( 'input.message' ).val( '' ).focus()
} )
socket.on( 'the_response', function( msg ) {
    console.log( msg )
    if( typeof msg.user_name !== 'undefined' ) {
          $( 'h3' ).remove()
          $( 'div.message_holder' ).append( '<div><b style="color: #000">'+msg.user_name+'</b> '+msg.message+'</div>' )
        }
      })
```

+++

On constate donc deux étapes : (i) on attache à la soumission du formulaire l'action d'envoyer via le websocket un evénemment de type `receive_msg` avec comme contenu le nom d'utilisateur et le message saisie ; (ii) on définit l'action à la réception d'un évènement de type `the_response` comme devant ajouter une ligne dans le div `message_holder`. 

Le code complet de cet exemple est à votre disposition dans le dossier `flask_demo/demo6`. Par exemple pour vous faire la main, ce serait sympa qu'un message annoncant un nouvel utilisateur apparaisse quand quelqu'un se connecte ... Indice dans websocket il y a un évènement particulier qui s'appelle `connect`...

+++

## Tout ce qui n'est pas dans ce cours

+++

Au début de ce notebook je vous ai dit que l'on fait du Flask car on a pas beaucoup de temps. Et bien mauvaise surprise cela ne suffit pas pour voir tout ce que l'on peut faire avec Flask. Il existe en effet de nombreux autres modules disponibles autour de Flask permettant d'enrichir les fonctionnalités. Parmi ces modules que l'on ne verra pas il y a la gestion de l'authentification, l'interaction avec les bases de données, ... Plein de choses intéressantes mais bon on a pas le temps... Si vous êtes déçu allez vous plaindre à la DE pour avoir plus d'heures d'informatique. Néanmoins, si cela vous intéresse voici ci-dessous quelques liens qui vous permettront de vous avancer sur le sujet. 

[https://flask.palletsprojects.com/en/1.1.x/](https://flask.palletsprojects.com/en/1.1.x/)

[https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)

Si vous lisez tout ça vous serez des experts de Flask ;)

+++

## A réfléchir pour le dernier cours

+++

Pour finir, je vais vous laisser réfléchir à un projet pour la semaine prochaine. Pas d'affolement, rien d'obligé c'est pour ceux que ca intéresse. Je suppose que j'ai perdu 90% de la promo avec la phrase d'avant ... Vous vous souvenez du super Hackaton que vous avez fait au mois de Janvier. Le sujet c'était de refaire dans le langage de votre choix parmi c++/python/java un jeu de Rogue. Et bien pour le prochain cours je vous invite à essayer de faire la même chose mais en utilisant flask, et un rendu à base de html, css, javascript. Et si l'un d'entre vous ce sent de faire le code en direct au prochain cours avec plaisir. Enjoy ;)

+++

Bon et vu que selon mes estimations il doit vous rester un peu de temps sur cette séance vous pouvez commencer tout de suite !

```{code-cell}

```
