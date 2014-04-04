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

Il est accessible à partir de l'URL 

https://download.data.grandlyon.com/wms/[nom_du_service]

Généralement on fait appel à son opération getCapabilities pour connaître son contenu (liste des couches) :

https://download.data.grandlyon.com/wms/grandlyon?SERVICE=WMS&REQUEST=GetCapabilities&VERSION=1.3.0 

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez le contenu à l'aide d'une requête GetMap de ce type : 

https://download.data.grandlyon.com/wms/grandlyon?LAYERS=adr_voie_lieu.adrcommune&FORMAT=image%2Fpng&EXCEPTIONS=application%2Fxml&TRANSPARENT=TRUE&VERSION=1.1.1&SERVICE=WMS&REQUEST=GetMap&STYLES=&SRS=EPSG%3A4171&BBOX=4.7969555930669,45.74564647571,4.8423155242545,45.794786401164&WIDTH=720&HEIGHT=780

.. image:: https://download.data.grandlyon.com/wms/grandlyon?LAYERS=adr_voie_lieu.adrcommune&FORMAT=image%2Fpng&EXCEPTIONS=application%2Fxml&TRANSPARENT=TRUE&VERSION=1.1.1&SERVICE=WMS&REQUEST=GetMap&STYLES=&SRS=EPSG%3A4171&BBOX=4.7969555930669,45.74564647571,4.8423155242545,45.794786401164&WIDTH=720&HEIGHT=780
   :alt: GrandLyon Smart Data : le service WMS
   :class: floatingflask

Ca peut vous sembler un peu compliqué et fastidieux... Mais les librairies cartographiques sont là pour vous aider. Leaflet et OpenLayers implémentent des classes WMS qui feront tout ça pour vous, en adaptant les requêtes selon les manipulations faites sur la carte. Il vous suffit généralement de renseigner l'URL de base du service et d'indiquer le nom de la couche que vous souhaitez utiliser comme l'illustrent nos :ref:`exemples`.


Service WFS
-----------
Les services WFS permettent de récupérer la donnée brute telle qu'elle est enregistrée dans la base de données. Il s'agit d'un service de téléchargement, même si ce téléchargement (récupération de la donnée) n'est pas forcément complet, c'est-à-dire qu'on peut l'effectuer sur une partie seulement du territoire, notamment pour les données les plus volumineuses. 

Il est accessible à partir de l'URL 

https://download.data.grandlyon.com/wfs/[nom_du_service]

Généralement on fait appel à son opération getCapabilities pour connaître son contenu (liste des couches) :

https://download.data.grandlyon.com/wfs/grandlyon?SERVICE=WFS&REQUEST=GetCapabilities&VERSION=1.1.0 

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez le contenu à l'aide d'une requête GetFeatures de ce type : 

https://download.data.grandlyon.com/wfs/grandlyon?SERVICE=WFS&REQUEST=GetFeature&typename=pvo_patrimoine_voirie.pvotronconwebcriter&VERSION=1.1.0

Mais là encore, rassurez-vous, les librairies cartographiques disposent des classes nécessaires à une utilisation simple de ce type de service. 


Services REST (en JSON)
-----------------------
Les services JSON de notre infrastructure permettent une navigation facile et rapide entre les différents jeux de données mis à disposition. Chaque service possède un point d'entrée dédié :

https://download.data.grandlyon.com/ws/grandlyon/all.json

et 

https://download.data.grandlyon.com/ws/smartdata/all.json

Ces documents listent l'ensemble des tables disponibles en consultation/téléchargement. Certaines peuvent avoir un accès restreint en fonction de vos droits. 

De liens en lien, vous pouvez alors naviguer vers la description des tables (par ex. https://download.data.grandlyon.com/ws/grandlyon/fpc_fond_plan_communaut.fpcplandeau.json), les différentes valeurs présentes dans un champ particulier (par ex. les essences des arbres de la métropole : https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json). Ce dernier mode dispose d'options particulières :

* compact : si false, décrit la valeur pour chacun des enregistrements, sinon liste les différentes valeurs trouvées dans la table. True par défaut.

* maxfeatures : indique le nombre maximal d'enregistrement à faire remonter par le service. 1000 par défaut. 

* start : indique l'index de départ, afin de pouvoir paginer les résultats. 1 par défaut. 

On peut ainsi demander au service les essences de 50 arbres à partir du 100e dans la base : 

https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json?compact=false&maxfeatures=50&start=101


On peut également accéder à la totalité du contenu de la table (ou paginer ce contenu) en utilisant une URL du type :

https://secure.grandlyon.webmapping.fr/ws/smartdata/jcd_jcdecaux.jcdvelov/all.json?compact=false

pour consulter l'intégralité des enregistrements. 


Les services REST-JSON sont ainsi particulièrement adaptés à la constition de listes de valeurs, de tableaux et de grilles paginés, d'interface de navigation au sein des données. 


Service OSM (OpenStreetMap)
---------------------------

La plateforme SmartData propose un service de fond de carte tuilé construit à partir des données `OpenStreetMap <openstreetmap.fr>`_ de la région Rhône-Alpes. Il est utilisable à partir de l'URL :

http://openstreetmap.data.grandlyon.com

.. image:: http://openstreetmap.data.grandlyon.com/?LAYERS=osm_grandlyon&SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&STYLES=&EXCEPTIONS=application%2Fvnd.ogc.se_inimage&FORMAT=image%2Fjpeg&SRS=EPSG%3A4326&BBOX=4.8484037210919,45.764534434461,4.8548554273902,45.770986140759&WIDTH=256&HEIGHT=256
   :alt: GrandLyon Smart Data : le service OSM
   :class: floatingflask

Le nom de couche à utiliser est tout simplement osm_grandlyon. La couche est disponibles dans les projections suivantes :

* ESPG:3857 et EPSG:900913 (Mercator Sphérique)

* EPSG:4326 (WGS84)

* EPSG:4171 (RGF93)

Veuillez noter que ces deux derniers systèmes sont définis en degrés et non en mètres, et que leur utilisation pour faire une carte (et non lire les données) aboutit à un résultat visuel un peu écrasé qui est tout à fait normal (puisque vous projetez de fait des coordonnées géographiques sphériques sur un plan, le fichier ou l'écran. Cette projection est nommée `plate-carrée <http://fr.wikipedia.org/wiki/Projection_cylindrique_%C3%A9quidistante>`_).
