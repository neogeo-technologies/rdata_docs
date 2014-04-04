.. _authentification:

Authentification et bonnes pratiques
=============================================

Principes
-------------------

Certaines des données publiées par les services SmartData nécessitent une autorisation. Afin d'en obtenir une, vous devez ouvrir un compte sur http://smartdata.grandlyon.com/creation-de-compte/ et spécifier les différents jeux de données et modalités d'accès que vous souhaitez. 

Une fois ces opération réalisées, vous aurez un identifiant (généralement l'adresse email utilisée lors de la création du compte) et un mot de passe. Ceux-ci vous sont personnels, et leur utilisation dans le contexte du développement d'application pour les tiers doit donc être fait avec certaines précautions. 

Mais voyons d'abord comment utiliser ces éléments d'identification pour accéder à des données protégées. 

La méthode d'authentification utilisée est le `Basic Auth HTTP <http://fr.wikipedia.org/wiki/Authentification_HTTP#M.C3.A9thode_Basic>`_ Lors de l'accès à chacun des types de services, il est donc nécessaire d'utiliser le header HTTP nommé 'Authorization' dans lequel seront insérés login et mot de passe, séparés par deux points (":") et encodé en `base64 <http://fr.wikipedia.org/wiki/Base64>`_. Le header ressemble alors à ceci :

::

  Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
 
Mais ne vous y méprenez pas. L'encodage en base64 n'est pas un cryptage ! Le procédé est réversible et on peut donc retrouver les valeurs encodés très facilement. Couplé à un flux HTTPS, qui crypte les informations transmises par et vers le serveur, ils ne sont pas récupérables, mais écrits tels quels dans le code ils le sont. 


Exemple avec cURL
-------------------

L'utilisation du header authorization avec cURL est très simple. Imaginons un utilisateur doté des identifiants suivants :

* login : demo@demo.fr
* password : demo

L'instruction cURL à utiliser pour accéder à la donnée "donnee_privee" sur le service smartdata serait alors :

..
    cURL -u demo@demo.fr:demo curl https://download.data.grandlyon.com/ws/smartdata/donnee_privee/all.json?compact=false

sauf erreur, vous devriez alors recevoir un flux json. 
 
