Les services offerts par GrandLyon Smart Data
=============================================

La plateforme Smart Data vous permet d'accéder à différents services de consultation des données OpenData. Les services sont de deux types :
des services de visualisation qui vous permettent d'afficher des images (cartes) et des services de téléchargement qui vous permettent d'accéder directement à la donnée, selon des modalités et des formats variés.

Lors de la mise en oeuvre d'une application cartographique, on ne peut pas traiter toutes les données de la même manière. Il y a en effet des données qui habillent (fond de carte par ex.), d'autres qui renseignent sur le contexte (points d'intérêt divers : mairie, gare...) et d'autres enfin qui portent l'information importante, celle que l'on souhaite vraiment mettre en avant et exploiter dans l'application (disponibilité des vélos, localisation des bus en temps réel...). 

Le fond de plan sera toujours proposé au format image, non interactif, et très souvent issu d'un système de cache tuilé permettant la récupération rapide sous forme de petites tuiles (256 x 256 pixels) de son contenu. Ce sera toujours la couche la plus basse dans l'application, celle en dessous de toutes les autres, pour des raisons évidentes de visibilité. 

Les informations cruciales, importantes, porteuses de la valeur ajoutée de l'application seront les plus interactives possibles, pour que d'un survol une info-bulle donne accès à l'essentiel, qu'un clic ouvre un fiche complète, qu'un changement de style (taille, couleur) dynamique permette de souligner la sélection, l'association, la relation avec un autre élément. Pour ce faire il faut donc utiliser un service de téléchargement, seul à même de transmettre les données brutes permettant la mise en oeuvre d'une couche vectorielle dans l'application cartographique ou de donner accès à tous les attributs (informations associées à l'objet géographique) directement. 

Pour les autres couches de données il faut faire des arbitrages. On ne peut tout charger en vectoriel pour des raisons de lisibilité (trop de points/lignes qui clignotent, grossissent ou s'agitent en même temps rendent la carte inutilisable) et de performance (chaque point est inscrit dans le DOM de la page. Plusieurs millieurs de points deviennent très lourds à gérer pour le navigateur). Donc on transige. On utilise les services de visualisation (WMS, format image) pour les données dont l'emplacement l'information la plus importante (par ex: stationnement handicapé, il n'y a rien à mettre dans une info-bulle, l'important est que la place soit là où elle est, et il suffit donc de l'afficher) ou dont l'emprise spatiale a un intérêt particulier (contours d'un parc, d'une zone règlementée...). Et on choisit le format vectoriel pour quelques informations certes secondaires par rapport à l'objet principal de l'application, mais dont les caractérisques sont importantes à connaître (stations vélos, événement...)



Service WMS
-----------
Le service WMS est le service de visualisation par excellence. Il sert à "voir" la donnée géographique avec une mise en forme prédéfinie (couleurs, styles, symboles...). C'est le service à privilégier pour intégrer des jeux de données au format image. 

Il est accessible à partir de l'URL https://download.data.grandlyon.com/wms/[nom_du_service]

Généralement on fait appel à son opération getCapabilities pour connaître son contenu (liste des couches) :

https://download.data.grandlyon.com/wms/grandlyon?SERVICE=WMS&REQUEST=GetCapabilities&VERSION=1.3.0 renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service. 

Service WFS
-----------
Blabla
