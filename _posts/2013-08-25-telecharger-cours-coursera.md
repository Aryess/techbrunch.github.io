---
layout: post
title: "Télécharger les cours de Coursera"
description: "Utilisation d'un script pour télécharger l'ensembre des contenus de cours sur Coursera.org"
categories: articles
tags: [tuto]
comments: true
---



Coursera est un site proposant des cours en ligne ouvert et massif (en anglais : massive open online course, MOOC). Ce site est tout simplement génial vous y trouverez des cours dans un très grand nombre de domaines et d'une très grande qualité. Vous trouveres des cours aussi bien sur la [Sécurité informatique](https://www.coursera.org/course/security) que sur la [Litérature](https://www.coursera.org/course/onlinegames) ou la [Psychologie](https://www.coursera.org/course/socialpsychology). Je vous recommande d'ailleur le cours [Introduction to Philosophy](https://www.coursera.org/course/introphil) de l'université d'Edimburgh qui est vraiment très intéressant et qui ne demande pas trop d'investissement.

Un avantage de coursera c'est qu'il est possible de télécharges les contenus mis à disposition sur le site afin de pouvoir suivre le cours sans forcément être connecté à internet, une fois les vidéos téléchargés vous pourrez donc les regarder sur votre smartphone ou votre tablette en allant au boulot. Par contre si vous souhaitez faire ça pour l'ensemble des cours cela peut vite devenir fastidieux de télécharger tous ces fichiers individuellement.

Heureusement quelqu'un a créé un [script python](https://github.com/dgorissen/coursera-dl) qui permet de télécharger l'ensemble des contenus mis à disposition sur le site, notamment les vidéos avec leur sous-titres, les quizzes et les pdf. J'ai un peu galéré à faire fonctionner le script sur l'ordinateur que j'utilise actuellement qui est sous windows 8 et donc je me suis dit qu'un petit tutoriel pourrait en intéresser certains.

## Prérequis

Pour faire fonctionner le script il faudra que vous ayez installé Python en version 2.7 (le script ne fonctionnera pas avec Python 3.3) et pip qui est un outil permettant l'installation et la gestion des packages python.

Pour installer python il suffit de télécharger le l'installer windows que vous trouverez sur [python.org](http://www.python.org/) la version actuelle étant la 2.7.5. Pour installer pip en revanche j'ai un peu plus galérer pour enfin touvez une solution simple et qui marche sur [stackoverflow](http://stackoverflow.com/questions/4750806/how-to-install-pip-on-windows/14407505#14407505).

Il va falloir tout d'abord télécharger deux scripts python :

* [Setuptools](https://bitbucket.org/pypa/setuptools/raw/92fa8285c9341b2d01b2ff270dcaa6073d97bbd5/ez_setup.py)
* [Pip](https://raw.github.com/pypa/pip/master/contrib/get-pip.py)

Il suffit ensuite d'exécuter dans l'ordre ces deux scripts.

{% highlight bash %}
python ez_setup.py
python get-pip.py
{% endhighlight %}

Vous devriez alors vous retrouver avec deux executables `easy_install.exe` and `pip.exe` dans le dossier `Scritps` du répertoire d'installation de Python (par défaut : `C:\Python27\Scripts`).

## Installation et Utilisation de coursera-dl

Maintenant que Pip est utilisable on va pouvoir installer coursera-dl.

{% highlight bash %}
pip.exe install coursera-dl
{% endhighlight %}

Une fois que le script est installé vous pouvez l'utiliser, pour connaitre le fonctionnement du script vous pouvez utiliser l'option `-h` :

{% highlight bash %}
C:\Python27\Scripts>coursera-dl -h
usage: coursera-dl-script.py [-h] [-u USERNAME] [-p PASSWORD] [-d DEST_DIR]
                             [-n IGNOREFILES] [-q PARSER] [-x PROXY]
                             [--reverse-sections] [--trim-path]
                             <course name> [<course name> ...]

Download Coursera.org course videos/docs for offline use.

positional arguments:
  <course name>       one or more course names from the url (e.g.,
                      comnets-2012-001)

optional arguments:
  -h, --help          show this help message and exit
  -u USERNAME         coursera username (.netrc used if omitted)
  -p PASSWORD         coursera password
  -d DEST_DIR         destination directory where everything will be saved
  -n IGNOREFILES      comma-separated list of file extensions to skip, e.g.,
                      "ppt,srt,pdf"
  -q PARSER           the html parser to use, see http://www.crummy.com/softwa
                      re/BeautifulSoup/bs4/doc/#installing-a-parser
  -x PROXY            proxy to use, e.g., foo.bar.com:3125
  --reverse-sections  download and save the sections in reverse order
  --trim-path         Trim path names to fit OS constraints (windows only)
{% endhighlight %}

Exemple avec la récupération du cours [Startup Engineering](https://www.coursera.org/course/startup) de Stanford :

{% highlight bash %}
coursera-dl -d / startup-001
{% endhighlight %}

Note : Vous ne pourrez télécharger les cours que si vous avez au préalable accepté le code d'honneur.

