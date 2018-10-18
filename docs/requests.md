# HTTP Request

- [Introduction](#introduction)
- [Accéder à la Request](#accéder-à-la-request)
- [Récupération des données](#récupération-des-données)
  - [Via des Closures de route](#via-des-closures-de-route)
  - [Via un controlleur](#via-un-controlleur)
- [Chemin de requête et méthode](#Chemin-de-requête-et-méthode)
  - [Récupérer le chemin de requête](#récupérer-le-chemin-de-requête)
  - [Récupérer la methode de requête](#récupérer-la-methode-de-requête)
  - [Vérifier si la requete est en AJAX](#vérifier-si-la-requete-est-en-ajax)
- [Récupérer une entrée](#récupérer-une-entrée)
- [Fichier uploader](#fichier-uploader)
  - [Validation des téléchargements réussis](#validation-des-téléchargements-réussis)
  - [Savegarder le fichier](#savegarder-le-fichier)

## Introduction

Une requête HTTP c'est le message que l'utilisateur envoie à l'application Bow via son navigateur ou d'autre outil telque `curl`, `httpie` et autre.

Ce message est traite par Bow à travers la classe [`Bow\Http\Request`]000(https://bowphp.github.com/api/master/Bow/Http/Request.html). Cette classe vous permet d'avoir les informations sur la requête envoie par l'utilisateur comme son adresse ip, le type du message, les information d'un formulaire etc...

## Accéder à la Request

Pour capturer un message ou requête HTTP, vous pouvez premièrement utiliser le helper `request()` qui vous donne une instance de [`Bow\Http\Request`](https://bowphp.github.com/api/master/Bow/Http/Request.html), deuxièmement vous pouvez passer par l'injecteur de dépendance via une action de contrôleur.

## Récupération des données

En utilisant quelques méthodes simples, vous pouvez accéder à toutes les entrées utilisateur de votre application.

### Via des Closures de route

- Avec le helper `request`:

```php
$app->get('/', function ()
{
  $request = request();

  $name = $request->get('name');
  //
});
```

- Par l'injection de dépendance

```php
use Bow\Http\Request;

$app->get('/', function (Request $request)
{
  return $request->get('name');
});
```

### Via un controlleur

Si votre méthode de contrôleur attend également une entrée d'un paramètre de route, vous devez répertorier les paramètres de votre route après vos autres dépendances. Par exemple, si votre route est défini comme suit:

```php
$app->get('/users/:id', 'UserController::show');
```

Vous pouvez toujours taper la requête [Bow\Http\Request](https://bowphp.github.com/api/master/Bow/Http/Request.html) et accéder à l'id de votre paramètre de route en définissant votre méthode de contrôleur comme suit:

```php
<?php

namespace App\Controllers;

use App\Controllers\Controller;
use App\User;
use Bow\Http\Request;

class UserController extends Controller
{
  /**
   * Afficher le profil pour l'utilisateur donné
   *
   * @param Request $reuest
   */
  public function show(Request $reuest, $id)
  {
    $user = User::where('id', $id)->first();

    $user->name = $request->get('name');

    $user->save();

    return $this->render('user/profile', ['user' => $user]);
  }
}
```

## Chemin de requête et méthode

### Récupérer le chemin de requête

La méthode `path` renvoie les informations de chemin de la requête. Ainsi, si la requête entrante est ciblée sur `http://example.com/bar/zar`, la méthode path retournera `bar/zar`:

```php
$uri = $request->path();
```

La méthode `is` vous permet de vérifier que le chemin de requête entrant correspond à un modèle donné. Vous pouvez utiliser le caractère `*` comme caractère générique lorsque vous utilisez cette méthode:

```php
if ($request->is('users/*')) {
  //
}
```

### Récupérer la methode de requête

La méthode `method` retournera le verbe HTTP pour la requête. Vous pouvez utiliser les méthodes `isPost`, `isGet`, `isPut`, `isDelete`, `isOptions`, `isPutch` pour vérifier que le verbe HTTP correspond à une methode HTTP de la requête donnée:

```php
$method = $request->method();

if ($request->isPost()) {
  // code ici
}

if ($request->isGet()) {
  // code ici
}

if ($request->isPut()) {
  // code ici
}

if ($request->isDelete()) {
  // code ici
}

if ($request->isOptions()) {
  // code ici
}

if ($request->isPatch()) {
  // code ici
}
```

### Vérifier si la requete est en AJAX

Pour determiner si une requete a été envoyé via AJAX la methode `isAjax` vous permet de vérifier ça.

```php
if ($request->isAjax()) {
  //
}
```

### Récupérer une entrée

Souvent la requête envoyé par le client HTTP contient des données souvent demander pour actioner un mecanisme par exemple celui d'une inscription.

Bow framework dans son démarrage optimise le donnée envoyés au serveur en remplaçant les valeur vide par `null` et supprimer les spaces blancs dans les valeurs de chaque champs avec la fonction php `trim`.

```php
$name = $request->get('name');
```

ou

```php
$name = $request->name;
```

Ceci permet juste de récupérer le contenu du champ `name` d'un formulaire ou d'un url en fonction du type de requête.

### Récupération d'une partie des données d'entrée

Si vous devez extraire un sous-ensemble des données d'entrée, vous pouvez utiliser les méthodes `ignore` et `only`. Ces deux méthodes acceptent un seul tableau ou une liste d'arguments dynamique:

```php
$input = $request->only(['name', 'lastname']);

$input = $request->only('name', 'lastname');

$input = $request->ignore(['password']);

$input = $request->ignore('password');
```

### Déterminer si une valeur d'entrée est présente

Vous devez utiliser la méthode `has` pour déterminer si une valeur est présente sur la requête. La méthode `has` renvoie `true` si la valeur est présente dans la requête:

```php
if ($request->has('name')) {
  //
}
```

### Déterminer l'adresse IP de client

Souvent vous allez vouloir l'adresse IP du client qui à envoyer la requête, la methode `ip` permet comme son nom l'indique de récupérer l'adresse ip de client:

```php
$ip = $request->ip();

if ($ip == 'xxx.xxx.xxx.xxx') {
  // code ici
}
```

### Déterminer le Port de client

La methode `port` permet comme son nom l'indique de récupérer le port de client:

```php
$port = $request->port();

// Code ici
```

### Déterminer le protocol de client

La methode `protocol` permet comme son nom l'indique de récupérer le protocol de client:

```php
$protocol = $request->protocol();
// Code ici
```

Il y a aussi la methode `isProtocol` qui permet de faire de vérification sur le protocol:

```php
if ($request->isProtocol('http')) {
  // code ici
}
```

Et `isSecure` vous permet de vérifier si la requête est sécurisé.

```php
if ($request->isSecure()) {
  // code ici
}
```

### Récupérer les entête-https

Vous avez la possibilité de récupérer les entête-https du client avec la methode `getHeader` et `hasHeader` pour vérifier l'existance d'une entête-http.

```php
$header = $request->getHeader('content-type');

echo $header;
// text/html

if ($request->hasHeader('x-proxy-key')) {
  // code ici
}
```

### Récupérer le IP du server

Vous avez la possibilité de récupérer l'adresse IP de serveur sur lequel votre application Bow.

```php
$hostname = $request->hostname();

echo $hostname;
// exemple.com
```

### Récupérer le domaine Origin

Le domaine dit `origin` est l'adresse récupérer avec la methode `hostname` ensuite associé avec le protocol de la requête.

```php
$hostname = $request->origin();

echo $hostname;
// https://exemple.com
```

### Récupérer le temps de la requête

Il est souvent interssent de voir combien de temps à faire une requête. La methode `time` nous permet de le faire et elle retourne un timestamp.

```php
$time = $request->time();

echo $time;
```

## Fichier uploader

Souvent les requêtes sont associés à des fichiers envoyés par l'utilisateur, vous pouvez traiter ces fichiers sur [`Bow\Http\Request`](https://bowphp.github.com/api/master/Bow/Http/Reqeust.html) les methodes `file` et le helper du même nom.

considérons le formulaire suivant:

```html
<form action="/upload" method="post" enctype="multipart/form-data">
  <input type="file" name="photo" accept="image/png"><br/>
  <button type="submit">Uploader</button>
</form>
```

Comment avoir accès au fichier envoyé ? Ici nous pouvons utiliser la methode `file` qui retourne `null` ou une instance de [`Bow\Http\UploadFile`](https://bowphp.github.com/api/master/Bow/Http/UploadFile.html[`Bow\Http\UploadFile`](https://bowphp.github.com/api/master/Bow/Http/UploadFile.html)).

```php
use Bow\Http\Request;

$app->post('/upload', function(Request $request) {
  $file = $request->file('photo');
  // ou
  $file = file('photo');

  debug($file);
});
```

```php
// Debug output
UploadFile {#92 ▼
  -file: array:5 [▼
    "name" => "23270116.png"
    "type" => "image/png"
    "tmp_name" => "/tmp/phpellwCx"
    "error" => 0
    "size" => 7388
  ]
}
```

Vous pouvez déterminer si un fichier est présent sur la requête en utilisant la méthode `hasFile`:

```php
if ($request->hasFile('photo')) {
  // code ici
}
```

### Validation des téléchargements réussis

Vous pouvez vérifier qu'il n'y a pas eu de problèmes lors du téléchargement avec la méthode `isUploaded`:

```php
$file = $request->file('photo');

if ($file->isUploaded()) {
  // Code ici
}
```

### Savegarder le fichier

Cela consiste juste à déplacer le fichier uploader dans une autre dossier et ceci grace à la methode `moveTo`.

```php
$file = $request->file('photo');

$file->moveTo("/path/to/your/store/directory", $filename = null);
```

Si `$filename` est `null`, sa valeur sera le nom du fichier hasher avec la methode [`getHashName`](https://bowphp.github.io/api/master/Bow/Http/UploadFile.html#method_getHashName).

Souvent dans le cas où vous attendez des fichiers envoyé via un formulaire dont le champs est sous la forme `photo[]`. la methode `file` retourne une [`Bow\Support\Collection`](https://bowphp.github.com/api/master/Bow/Support/Collection.html):

```html
<form action="/upload" method="post" enctype="multipart/form-data">
  <label>Fichier 1: <input type="file" name="photo[]" accept="image/png"></label><br/>
  <label>Fichier 2: <input type="file" name="photo[]" accept="image/png"></label><br/>
  <button type="submit">Uploader</button>
</form>
```

Dans cette exemple, la methode `file` retournera une [`Bow\Support\Collection`](https://bowphp.github.com/api/master/Bow/Support/Collection.html) qui contiendra pour chaque oruccencre une instance de [`Bow\Http\UploadFile`](https://bowphp.github.com/api/master/Bow/Http/UploadFile.html).

```php
use Bow\Http\Request;
use Bow\Http\UploadFile;

$app->get('/upload', function (Request $request)
{
  $files = $request->file('photo');

  if (!$files->isEmpty()) {
    $files->each(function(UploadFile $file) {
      $file->moveTo("/path/to/your/store/directory");
    });
  }
});
```

> N'hésitez pas à donner votre avis sur la qualité de la documentation ou proposez des correctifs.