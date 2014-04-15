.. _bonnespratiques:

=================
Bonnes Pratiques
=================

-----------
Principes
-----------

Que vous souhaitiez développer une application web ou embarquée dans un smartphone, vous avez sans doute le souci de garantir son bon fonctionnement et sa disponibilité. Voici quelques conseils destinés à vous faire utiliser notre plateforme de manière optimale.

------------------------------------
Accès direct aux données protégées
------------------------------------

Comme vous l'avez sans doute remarqué, certaines de nos données ne sont utilisables que si vous disposez d'un compte vous y autorisant. Dès lors que vous implémentez ces informations dans votre code, il vous appartient de veiller à leur sécurité, leur non-divulgation et leur facilité d'emploi tout au long de leur cycle de vie (mise à jour, suppression...)

En Javascript/Ajax
-------------------

Une des particularité du Javascript est d'être chargé et de s'exécuter côté client. Si vous intégrez vos identifiants dans votre code javascript, ils seront donc forcément accessibles à l'utilisateur de votre site. Employer ruses et contournements ne fera que rendre plus difficile leur récupération, mais à coup sûr ils apparaîtront tôt ou tard dans les requêtes que vous allez émettre vers nos services. Pour tout développement en Javascript/Ajax, il faut donc utiliser les technique de `Proxyfication`_ des requêtes décrites plus bas. 

En PHP, Python, Java
---------------------

Pas de problèmes avec ces languages qui s'exécutent côté serveur. Il suffit d'implémenter les `exemples`_ proposés.

Sur plateforme mobile
----------------------

Les développements effectués sur plateforme mobile sont relativement sûrs dans la mesure où ils sont compilés et sont donc distribués à vos utilisateurs sous une forme qui rend extrêmement difficile la récupération des informations qui s'y trouvent. Par contre, il vous faut considérer un aspect pratique lié au cycle de vie de vos identifiant. Pour peu que vous changiez de mot de passe, ou même de compte, comment va se passer la diffusion du nouveau couple login/mot de passe sur les terminaux de vos utilisateurs ? Devront-ils faire une mise à jour ? Pourrez-vous provoquer cette mise à jour obligatoire ? Cette réflexion devra avoir été menée avant de diffuser votre solution. 

Performances
--------------

Une autre question importante est celle des performances de notre plateforme. Elle n'est pas dimensionnée pour tenir la même charge que www.google.fr. Si votre application a du succès, si un tweet la propulse vers des sommets de popularité très rapidement, êtes vous sûrs que nos infrastructures tiendront la charge et permettront à vos utilisateurs d'avoir une bonne expérience avec votre application ? 

Conclusion
--------------

Pour chacune des situations ci-dessus, il vous faut veiller à la mise en oeuvre des pratiques conseillées. Le plus simple est parfois de récupérer directement la donnée.  


-----------------------------------
Récupération des données
-----------------------------------

Vos droits d'accès vous permettent d'accéder à la donnée et de la stocker sur vos serveurs. Leur fréquence de récupération doit être adaptée à leur fréquence de mise à jour. Une fois intégrée à votre infrastructure, la résolution des problèmes évoqués précédemment est beaucoup plus simple :

* Sur plateforme mobile, les utilisateurs accèdent directement à votre version des données. Si vos login/mot de passe change, cela affecte votre script de récupération des données uniquement et non l'application déployée chez les clients, qui de son côté peut utiliser un système d'authentification/autorisation qui vous est propre. 

* Pour les performance : la récupération ponctuelle, même si elle est fréquente, affecte peu les performances de notre plateforme. Pour peu que votre infrastructure soit dimensionnée pour absorber la charge générée par votre application, vous n'aurez pas de problème à craindre de notre côté en vous y prenant ainsi.

-----------------------------------
Proxyfication
-----------------------------------

Que les données soient installées au sein de votre infrastructure ou que votre application accède directement aux données sur la nôtre, il est des cas d'utilisation ou le recours à une proxyfication va néanmoins être indispensable. C'est surtout le cas en environnement Javascript avec lequel vous avez deux obstacles :

* L'impossiblité de cacher les login/mot de passe à transmettre pour accéder aux données. Vous pouvez néanmoins avoir mis en oeuvre, après avoir récupéré les données, un mécanisme d'authentification qui est propre à vos utilisateurs, auquel cas vous réglez une partie du problème

* L'accès à un domaine tiers en Ajax. Ceci est strictement interdit par "Same Origine Policy" qui impose au contenu appelé par une requête Ajax d'être situé sur le même domaine et le même port que la page principale dans laquelle se situe la requête. Cette règle ne vaut pas pour les images, ce qui explique que l'on puisse utiliser les flux WMS externes directement, mais vaut pour tous nos autres types de services : WFS, JSON ou KML. 

Pour contourner ces restrictions, vous pouvez employer l'astuce de la proxyfication. Le principe est de transmettre les requêtes AJAX à un script situé sur votre propre serveur (donc mêmes domaine et port que la page principale), qui transmettra les requêtes au service externe, sans être soumis aux mêmes contraintes d'un client web. Cela poeut également vous servir pour distinguer les comptes d'accès à votre serveur de celui utilisé pour vous connecter à notre infrastructure. 


     CLIENT WEB ------> SCRIPT PROXY ------> SERVICE EXTERNE
	  user:toto.........user:my_account......>
	  
	  
Implémenter un script de proxyfication n'est pas très compliqué, mais il faut respecter certaines règles. Ne pas le laisser ouvert à tous vents par exemple, car sinon n'importe qui pourrait venir s'en servir pour naviguer sur internet de manière anonyme. Ne pas le laisser faire n'importe quoi non plus, afin qu'un utilisateur ne puisse pas envoyer n'importe quelle requête à votre serveur. 

	 