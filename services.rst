Les services offerts par GrandLyon Data
=============================================

La plateforme Data vous permet d'accéder à différents services de consultation des données OpenData. Les services sont de deux types :
des services de visualisation qui vous permettent d'afficher des images (cartes) et des services de téléchargement qui vous permettent d'accéder directement à la donnée, selon des modalités et des formats variés.

Lors de la mise en oeuvre d'une application cartographique, on ne peut pas traiter toutes les données de la même manière. Il y a en effet des données qui habillent (fond de carte par ex.), d'autres qui renseignent sur le contexte (points d'intérêt divers : mairie, gare...) et d'autres enfin qui portent l'information importante, celle que l'on souhaite vraiment mettre en avant et exploiter dans l'application (disponibilité des vélos, localisation des bus en temps réel...).

Le fond de plan sera toujours proposé au format image, non interactif, et très souvent issu d'un système de cache tuilé permettant la récupération rapide sous forme de petites tuiles (256 x 256 pixels) de son contenu. Ce sera toujours la couche la plus basse dans l'application, celle en dessous de toutes les autres, pour des raisons évidentes de visibilité.

Les informations cruciales, importantes, porteuses de la valeur ajoutée de l'application seront les plus interactives possibles, pour que d'un survol une info-bulle donne accès à l'essentiel, qu'un clic ouvre une fiche complète, qu'un changement de style (taille, couleur) dynamique permette de souligner la sélection, l'association, la relation avec un autre élément. Pour ce faire il faut donc utiliser un service de téléchargement, seul à même de transmettre les données brutes permettant la mise en oeuvre d'une couche vectorielle dans l'application cartographique ou de donner accès à tous les attributs (informations associées à l'objet géographique) directement.

Pour les autres couches de données il faut faire des arbitrages. On ne peut tout charger en vectoriel pour des raisons de lisibilité (trop de points/lignes qui clignotent, grossissent ou s'agitent en même temps rendent la carte inutilisable) et de performance (chaque point est inscrit dans le DOM de la page. Plusieurs milliers de points deviennent très lourds à gérer pour le navigateur). Donc on transige. On utilise les services de visualisation (WMS, format image) pour les données dont l'emplacement l'information la plus importante (par ex : stationnement handicapé, il n'y a rien à mettre dans une info-bulle, l'important est que la place soit là où elle est, et il suffit donc de l'afficher) ou dont l'emprise spatiale a un intérêt particulier (contour d'un parc, d'une zone règlementée...). Et on choisit le format vectoriel pour quelques informations certes secondaires par rapport à l'objet principal de l'application, mais dont les caractérisques sont importantes à connaître (stations vélos, événement...)



Service WMS
-----------

Le service WMS est le service de visualisation par excellence. Il sert à "voir" la donnée géographique avec une mise en forme prédéfinie (couleurs, styles, symboles...). C'est le service à privilégier pour intégrer des jeux de données au format image.

Il est accessible à partir de l'URL

https://download.data.grandlyon.com/wms/[nom_du_service]

Généralement on fait appel à son opération GetCapabilities pour connaître son contenu (liste des couches) :

https://download.data.grandlyon.com/wms/grandlyon?SERVICE=WMS&REQUEST=GetCapabilities&VERSION=1.3.0

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez le contenu à l'aide d'une requête GetMap de ce type :

https://download.data.grandlyon.com/wms/grandlyon?LAYERS=adr_voie_lieu.adrcommune&FORMAT=image%2Fpng&EXCEPTIONS=application%2Fxml&TRANSPARENT=TRUE&VERSION=1.1.1&SERVICE=WMS&REQUEST=GetMap&STYLES=&SRS=EPSG%3A4171&BBOX=4.7,45.6,5,45.9&WIDTH=720&HEIGHT=780

.. image:: https://download.data.grandlyon.com/wms/grandlyon?LAYERS=adr_voie_lieu.adrcommune&FORMAT=image%2Fpng&EXCEPTIONS=application%2Fxml&TRANSPARENT=TRUE&VERSION=1.1.1&SERVICE=WMS&REQUEST=GetMap&STYLES=&SRS=EPSG%3A4171&BBOX=4.7,45.6,5,45.9&WIDTH=720&HEIGHT=780
   :alt: GrandLyon Data : le service WMS
   :class: floatingflask

Ca peut vous sembler un peu compliqué et fastidieux... Mais les librairies cartographiques sont là pour vous aider. Leaflet et OpenLayers implémentent des classes WMS qui feront tout ça pour vous, en adaptant les requêtes selon les manipulations faites sur la carte. Il vous suffit généralement de renseigner l'URL de base du service et d'indiquer le nom de la couche que vous souhaitez utiliser comme l'illustrent nos :ref:`exemples`.

Service WFS
-----------

Les services WFS permettent de récupérer la donnée brute telle qu'elle est enregistrée dans la base de données. Il s'agit d'un service de téléchargement, même si ce téléchargement (récupération de la donnée) n'est pas forcément complet, c'est-à-dire qu'on peut l'effectuer sur une partie seulement du territoire, notamment pour les données les plus volumineuses.

Il est accessible à partir de l'URL

https://download.data.grandlyon.com/wfs/[nom_du_service]

Généralement on fait appel à son opération GetCapabilities pour connaître son contenu (liste des couches) :

https://download.data.grandlyon.com/wfs/grandlyon?SERVICE=WFS&REQUEST=GetCapabilities&VERSION=1.1.0

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez le contenu à l'aide d'une requête GetFeatures de ce type :

https://download.data.grandlyon.com/wfs/grandlyon?SERVICE=WFS&REQUEST=GetFeature&typename=pvo_patrimoine_voirie.pvotronconwebcriter&VERSION=1.1.0

Mais là encore, rassurez-vous, les librairies cartographiques disposent des classes nécessaires à une utilisation simple de ce type de service.

Le format généralement utilisé en WFS est le GML (Geographic Markup Language) qui est un dérivé du XML. Toutefois, ce format n'est pas forcément le plus simple à utiliser dans le cadre d'une application web. Aussi, tout en utilisant le WFS, il est possible de recevoir un flux au format GeoJSON en ajoutant le paramètre OUTPUTFORMAT=geojson :

https://download.data.grandlyon.com/wfs/grandlyon?SERVICE=WFS&REQUEST=GetFeature&typename=pvo_patrimoine_voirie.pvotronconwebcriter&VERSION=1.1.0&OUTPUTFORMAT=geojson

Service WCS
-----------

Les services WCS (Web Coverage Service) permettent de récupérer directement les données brutes des couches raster (comme les orthophotos et les MNT). Le terme Coverage (couverture) correspond au jeu de données raster.
Il s'agit donc d'un service de téléchargement dans lequel il est possible de filtrer le jeu de données à récupérer sur une partie seulement du territoire.

Il est accessible à partir de l'URL :

https://download.data.grandlyon.com/wcs/[nom_du_service]

De même que pour les WMS ou le WFS, on fait appel à son opération GetCapabilities pour connaître son contenu (liste des couches disponibles ) :

https://download.data.grandlyon.com/wcs/grandlyon?service=WCS&request=GetCapabilities&version=1.1.0

renvoie ainsi un document XML listant (entre autres) les couches mises à disposition par le service, dont vous obtiendrez la description détaillée à l'aide d'une requête DescribeCoverage de ce type :

https://download.data.grandlyon.com/wcs/grandlyon?service=WCS&request=DescribeCoverage&version=1.1.0&identifiers=MNT2015_Altitude_10m_WCS,road_ln_p

Les informations retournées ne concernent plus que les couches spécifiées dans le paramètre identifiers (ici MNT2015_Altitude_10m_WCS et road_ln_p) et sont un peu plus détaillées que dans le GetCapabilities.

Enfin, pour obtenir la couverture souhaitée, on utilise une requête GetCoverage de ce type :

https://download.data.grandlyon.com/wcs/grandlyon?SERVICE=WCS&VERSION=1.0.0&REQUEST=GetCoverage&FORMAT=GTiff&COVERAGE=road_ln_p&BBOX=1836243.96544679999351501,5162352.9513221001252532,1842093.96544679999351501,5168132.9513221001252532&CRS=EPSG:3946&RESPONSE_CRS=EPSG:3946&WIDTH=585&HEIGHT=578

Encore une fois, c'est un service standardisé et les librairies cartographiques disposent des classes nécessaires à une utilisation simple de ce type de service.

Service CSW
-----------

Les services CSW (Catalog Services for the Web) permettent d'interagir avec le catalogue de métadonnées de GrandLyon Data.

Ils recouvrent 2 grands types d'usage : la consultation et l'édition des métadonnées. Dans le cas présent, seules les fonctionnalités de consultation sont concernées puisqu'il n'y a pas lieu de mettre à jour le catalogue de la plateforme GrandLyon Data.
Les requêtes CSW vont ainsi permettre de rechercher des données et d'accéder à la fiche descriptive détaillée d'une donnée.

Comme pour les services précédemment décrit, la découverte du service se fait via le GetCapabilities :

https://download.data.grandlyon.com/catalogue/srv/fre/csw?SERVICE=CSW&request=GetCapabilities&service=CSW&version=2.0.2

Pour effectuer une recherche, on utilise l'opération GetRecords, dans laquelle on peut spécifier des critères de recherche. Par exemple :

https://download.data.grandlyon.com/catalogue/srv/fre/csw?SERVICE=CSW&request=GetRecords&service=CSW&version=2.0.2&resultType=results&OUTPUTSCHEMA=http://www.opengis.net/cat/csw/2.0.2&ELEMENTSETNAME=brief%20&CONSTRAINTLANGUAGE=CQL_TEXT&typeNames=csw:Record&maxRecords=1000&constraint_language_version=1.0.0

Notez le paramètre ELEMENTSETNAME qui permet de choisir le type d'élements retournés (brief, summary ou full). L'utilisation de startPosition et maxRecords permet de gérer la pagination pour ne pas charger d'un coup les plus de 500 fiches. Les critères de recherche peuvent être renseignés soit avec CQL, soit avec OGC FE (Filter Encoding).

L'opération GetRecordById permet d'accéder à une métadonnée à partir de son identifiant, donc d'obtenir le contenu détaillée pour une fiche précise :

https://download.data.grandlyon.com/catalogue/srv/fre/csw?SERVICE=CSW&request=GetRecordById&service=CSW&version=2.0.2&resultType=results&OUTPUTSCHEMA=http://www.opengis.net/cat/csw/2.0.2&ELEMENTSETNAME=full%20&id=3e6cd8af-5adb-4d9c-8638-f22db9b121fd

L'utilisation de ce service n'est pas simple au premier abord mais il est très performant et permet de retrouver toutes les fonctionnalités de recherche et de consultation disponibles sur le catalogue de la plateforme afin de les intégrer dans un client externe. Enfin, c'est un service standard et diverses documentations beaucoup plus détaillées sur le CSW sont facilement accessibles sur le web.


Services REST (en JSON)
-----------------------

Pour accéder aux données sous forme alphanumérique (par opposition aux services cartographiques), notre infrastructure dispose de services JSON permettant une navigation facile et rapide entre les différents jeux de données mis à disposition.

Le point d'entrée de chaque service est construit sur le pattern suivant : 

``https://download.data.grandlyon.com/ws/<service>/all.json``

Les services actuellement disponibles sont "grandlyon" et "rdata" :

``https://download.data.grandlyon.com/ws/grandlyon/all.json``

et

``https://download.data.grandlyon.com/ws/rdata/all.json``

Ces documents listent l'ensemble des tables disponibles en consultation/téléchargement. Certaines peuvent avoir un accès restreint en fonction de vos droits.

**Exemple de résultat** : 

:: 
  
  {
      
      results: [{
      
         table_schema: "abr_arbres_alignement",
         
         href: "https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre.json",
         
         table_name: "abrarbre"
      
      },{
         
         table_schema: "adr_voie_lieu",
         
         href: "https://download.data.grandlyon.com/ws/grandlyon/adr_voie_lieu.adradresse.json",
         
         table_name: "adradresse"

      },{
      
         ...
         
      }]

   }

A chaque table est associée une URL de la forme : 

``https://download.data.grandlyon.com/ws/<service>/<table_schema>.<table_name>.json``

De lien en lien, vous pouvez alors naviguer vers la description des tables.

*Exemple* : https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre.json

::

   {
      
      requested_table: "abr_arbres_alignement.abrarbre",
      
      nb_records: 92216,
      
      database_href: "https://download.data.grandlyon.com/ws/grandlyon/all.json",
      
      nb_results: 26,
      
      results: [{
      
         is_pk: false,
         
         column_type: "varchar",
         
         precision: 50,
         
         is_nullable: "YES",
         
         href: "https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json",
         
         column_name: "essencefrancais"
      
      },{
         
         is_pk: false,
         
         column_type: "int4",
         
         precision: 32,
         
         is_nullable: "YES",
         
         href: "https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre/circonference_cm.json",
         
         column_name: "circonference_cm"
      
      },{
      
         ...
         
      }]

   }

Liste des champs affichés :

* **is_pk**: est-ce l’identifiant de la couche 

* **column_type**: type de champ (numérique, texte, etc.)

* **precision**: longueur du champ

* **is_nullable**: peut il y avoir des valeurs nulles ?

* **href**: valeurs distinctes possible de l’attribut ciblé 

* **column_name**: nom du champ

L'url contenue dans href permet de consulter les différentes valeurs présentes dans un champ particulier (par ex. les essences des arbres de la métropole).

*Exemple* : https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json

::

   {
      
      fields: [
         
         "essencefrancais"
      
      ],
      
      nb_results: 401,
      
      values: [
         
         "Magnolia à grandes fleurs",
        
         "Erable rouge 'Schlesingeri'",
         
         "Arbre puant des Chinois",
         
         "Chène rouge d'Espagne",
         
         "Frêne d'Amérique",
         
         "Orme champêtre",
         
         "Chêne pédonculé fastigié, Chêne pyramidal",
         
         ...
      
      ]
   
   }

Ce dernier mode dispose d'options particulières :

* **compact** : si false, décrit la valeur pour chacun des enregistrements, sinon liste les différentes valeurs trouvées dans la table. True par défaut.

* **maxfeatures** : indique le nombre maximal d'enregistrement à faire remonter par le service. 1000 par défaut.

* **start** : indique l'index de départ, afin de pouvoir paginer les résultats. 1 par défaut.

On peut ainsi demander au service les essences de 50 arbres à partir du 100e dans la base :

https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre/essencefrancais.json?compact=false&maxfeatures=50&start=101


On peut également accéder à la totalité du contenu de la table (ou paginer ce contenu) en utilisant une URL du type :

https://download.data.grandlyon.com/ws/rdata/jcd_jcdecaux.jcdvelov/all.json?compact=false

pour consulter l'intégralité des enregistrements. 

Il faut noter que sur l'appel de all.json (affichage de tous les champs), seul le mode compact est disponible. 

Le nombre d’objet renvoyé par défaut est fixé à 1000 pour des raisons de performances. Il est possible d’outrepasser ce retour grâce au paramètre « maxfeatures ».

*Exemple* : 
https://download.data.grandlyon.com/ws/grandlyon/gip_proprete.gipdecheterie/all.json?maxfeatures=10

Il est également possible de filtrer les objets renvoyés selon une valeur d'attribut avec une url de la forme : 

``https://download.data.grandlyon.com/ws/<service>/<table_schema>.<table_name>/all.json?field=<attribut>&value=<valeur>``

*Exemple* : 
https://download.data.grandlyon.com/ws/grandlyon/abr_arbres_alignement.abrarbre/all.json?field=essencefrancais&value=Marronnier%20de%20Virginie

all.json contient aussi des informations supplémentaires liées à la pagination, à savoir des liens vers les pages précédentes et suivantes sous la forme d'une URL reprenant la valeur de maxfeatures utilisée  pour la page en cours et modifiant la valeur du paramètre "start" en fonction de la page en cours. 

*Exemple* : 
https://download.data.grandlyon.com/ws/grandlyon/gip_proprete.gipdecheterie/all.json?maxfeatures=5&start=10

Cette URL retourne les enregistrements 10 à 15 de la couche déchetterie.

Les services REST-JSON sont ainsi particulièrement adaptés à la constitution de listes de valeurs, de tableaux et de grilles paginés, d'interface de navigation au sein des données.

Services REST (en CSV)
----------------------

*Exemple* :
https://download.data.grandlyon.com/ws/grandlyon/gip_proprete.gipdecheterie/all.csv?maxfeatures=5&start=10

De la même façon que l'on requête le service JSON, on peut demander un extrait CSV en remplaçant l'extension ".json" de l'URL par ".csv".

Il est possible de remplacer le séparateur décimal en ajoutant 'ds=,' ou 'ds=.' dans la requête.

Le séparateur de colonne peut aussi être changé en utilisant l'option "separator=;" par exemple.

Un paramètre supplémentaire "geometry=on" (off par défaut) ajoute une colonne "WKT" contenant la géométrie de l'objet au format [WKT](https://fr.wikipedia.org/wiki/Well-known_text)

Export Shapefile
---------------
L'export shapefile est utilisable depuis les service JSON par l'utilisation d'une extension .shp au niveau de l'appel de couche (par exemple : https://download.data.grandlyon.com/ws/rdata/pvo_patrimoine_voirie.pvotrafic.shp). Cela renvoie alors à l'utilisateur un zip contenant un shapefile (SHP + SHX + DBF) de la couche. 

En outre, plusieurs paramètres sont à la disposition de l'utilisateur :

* srsname, qui permet de spécifier le système de coordonnées pour l'export (de la forme EPSG:XXXX, à choisir entre 4171, 3946, 2154, 4326 et 4258)

* une couche tierce de découpage, qui va être identifiée par un attribut mask_layer (nom de la couche de découpage), mask_field (champs sur lequel filtrer cette couche) et enfin mask_value (valeur du champ mask_field pour générer le polygone de découpe). Cela permet de récupérer une entité géométrique qui va servir de filtre à la couche principale demandée. 

Exemple https://download.data.grandlyon.com/ws/grandlyon/gic_collecte.gicsiloverre.shp?mask_db=grandlyon&mask_layer=adr_voie_lieu.adrcommune&mask_field=insee&mask_value=69034 

La couche des silos verre sera ici découpée pour n'en récupérer que les objets situés dans le polygone ayant l'attribut insee à 69034 de la couche adr_voie_lieu.adrcommune


Service WMTS
------------

La plateforme Data propose un service de fonds de carte tuilés au standard WMTS. Deux couches y sont proposées, d'une part l'orthophotographie 2015 de la Métropole et d'autre part une couverture construite à partir des données `OpenStreetMap <http://www.openstreetmap.fr>`_ des régions Auvergne-Rhône-Alpes et Bourgogne. Le service WMTS est utilisable à partir de l'URL :

https://openstreetmap.data.grandlyon.com/wmts/

.. image:: https://openstreetmap.data.grandlyon.com/wmts/?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=osm_grandlyon&STYLE=default&TILEMATRIXSET=GoogleMapsCompatible&TILEMATRIX=16&TILEROW=23379&TILECOL=33653&FORMAT=image%2Fpng
   :alt: GrandLyon Data : le service WMTS OpenStreetMap
   :class: floatingflask

.. image:: https://openstreetmap.data.grandlyon.com/wmts/?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=ortho2015&STYLE=default&TILEMATRIXSET=GoogleMapsCompatible&TILEMATRIX=16&TILEROW=23378&TILECOL=33652&FORMAT=image%2Fjpeg
   :alt: GrandLyon Data : le service WMTS Orthophotographie 2015
   :class: floatingflask

Le nom des couches à utiliser est respectivement osm_grandlyon et ortho2015. Les couches sont disponibles en projection Mercator Sphérique (EPSG:3857 et EPSG:900913) et sont donc à ce titre compatibles avec d'autres services du même type, GoogleMaps ou API-IGN.
Pour utiliser le service WMTS dans QGIS veillez à utiliser l'URL du GetCapabilities comme URL du service :
https://openstreetmap.data.grandlyon.com/wmts/?REQUEST=GetCapabilities&SERVICE=WMTS

Service WMTS (Orthophotographies)
---------------------------------

Des services WMTS/WMS supplémentaires existent diffusant des flux d'orthophotographies.

Ils sont accéssibles ici:
* https://imagerie.data.grandlyon.com/all/wmts?service=WTMS&request=getcapabilities
* https://imagerie.data.grandlyon.com/2154/wmts?service=WTMS&request=getcapabilities
* https://imagerie.data.grandlyon.com/3857/wmts?service=WTMS&request=getcapabilities
* https://imagerie.data.grandlyon.com/3946/wmts?service=WTMS&request=getcapabilities

Ces flux disposent d'un cache et sont à privilégier par rapports aux flux WMS disponibles sur https://download.data.grandlyon.com/wms/grandlyon

Services OpenMapTiles
---------------------

Ce service propose des tuiles OpenMaptiles à utiliser pour des fonds de carte.

Client de démonstration: https://openmaptiles.data.grandlyon.com/data/v3/#8.37/45.796/4.592

Ces tuiles sont mise à jour de façon hebdomadaires en utilisant les données OpenStreetMap.

https://openmaptiles.data.grandlyon.com/data/v3/1/1/0.pbf

Ces tuiles peuvent être utilisées par les principaux framework SIG web (MaplibreGL, Leaflet,...) par exemple : https://openmaptiles.org/docs/website/maplibre-gl-js/


Services SOS (données de capteurs)
----------------------------------

Le service **SOS** s'inscrit dans la collection de standards *Sensor Web Enablement* (SWE) de l'OGC. Il permet la découverte des capteurs et de leurs mesures. Le standard SOS est décliné en deux versions : ``1.0.0`` et ``2.0.0``.

Le Grand Lyon diffuse deux services SOS. Le premier concerne les données de stations de mesure de bruit (1) dans l’agglomération ; le deuxième la disponibilité de vélos libres sur les stations Vélo'V réparties entre Lyon et Villeurbanne.

\(1\) https://download.data.grandlyon.com/sos/bruit?service=SOS&request=GetCapabilities

\(2\) https://download.data.grandlyon.com/sos/velov?service=SOS&request=GetCapabilities

Pour le moment, les services SOS du Grand Lyon sont disponibles en version ``1.0.0``.

L’interrogation des services garde la logique des services de type OGC :

* La requête *GetCapabilities* retourne une description du service interrogé.

* Le *DescribeSensor* offre une description du capteur lui-même, autrement dit les métadonnées de capteur. Le document de réponse est encodé dans le standard de l'OGC *Sensor Model Language* (SensorML).

* Enfin le *GetObservation* permet de récupérer les données en appliquant un filtre de sélection (par exemple une période temporelle). Ces dernières sont encodées suivant le standard de l'OGC *Observations & Measurements* (OGC:O&M).

Représentation graphique des données
************************************

L'exploitation des données de capteurs n'est pas évidente. Aussi le Grand Lyon propose un service de représentation graphique simple de l'évolution temporelle des mesures pour chaque service SOS : l'évolution temporelle des niveaux sonores de l'Observatoire Acoustique (1) ; l'évolution temporelle de la disponibilité des vélos et des stands (2).

URL du service : ``http://demo.data.grandlyon.com/graph/<bruit|velov>?``

Le service prend deux paramètres (*query string*) :

* ``offering`` : Nom identifiant du réseau de capteur
* ``procedure`` : Nom identifiant du capteur

Les valeurs sont données dans les capacités (*GetCapabilities*) des services SOS.

\(1\) Pour les données de l'observatoire acoustique :

* http://demo.data.grandlyon.com/graph/bruit/?offering=observatoire_acoustique_grandlyon&procedure=AF01
* http://demo.data.grandlyon.com/graph/bruit/?offering=observatoire_acoustique_grandlyon&procedure=AF02

\(2\) Pour les données de disponibilité des stations Vélo'V  :

* http://demo.data.grandlyon.com/graph/velov/?&procedure=velov-1001&offering=reseau_velov
* http://demo.data.grandlyon.com/graph/velov/?&procedure=velov-1002&offering=reseau_velov

Représentation cartographique (WMS-Time)
****************************************

Les données sont aussi disponibles à travers un service WMS étendu (voir plus haut plus une description du WMS).

* https://download.data.grandlyon.com/wms/ldata?

Ce service a la particularité de proposer des couches supportant les requêtes temporelles. Les couches concernées (ou ``LAYER`` dans le vocabulaire OGC:WMS) prennent le suffixe ``_time``. Ainsi vous trouverez les paires de couches WMS suivantes :

* ``bruit.stations_observatoire_acoustique`` / ``bruit.stations_observatoire_acoustique_time`` pour les stations de mesure du bruit.
* ``velov.stations`` / ``velov.stations_time`` pour les stations Vélo'V.

Le Grand Lyon propose deux démonstrateurs cartographiques représentant respectivement les mesures du réseau de capteurs de l'observatoire acoustique (1) et la disponibilité de vélos par station Vélo'V (2) :

* \(1\) http://demo.data.grandlyon.com/wmst/observatoire_acoustique_grandlyon.html
* \(2\) http://demo.data.grandlyon.com/wmst/reseau_velov.html

Chaque station (bruit comme Vélo'V) est interrogeable (par un simple clique sur la carte) et permet d'accéder par un hyperlien à la représentation graphique des données. Un *slider* permet de naviguer sur l'axe temporel.



Services KML
------------
Le GrandLyon publie ses données au format KML. Les données de chaque service sont accessibles via l'url suivante : 
https://download.data.grandlyon.com/kml/[nom_du_service]/?request=layer&typename=[schema].[name]

*Exemple* : https://download.data.grandlyon.com/kml/grandlyon/?request=layer&typename=pvo_patrimoine_voirie.pvostationvelov

Ce format est adapté pour Google Earth et pour l'API Javascript Maps de Google.

Attention, pour des questions de performance, le KML embarque pour également les URLs WMS des services et les utilise à la place des objets vecteur lorsque le nombre d'objets à afficher est trop important (supérieur à 1000).

Enfin, il est également possible de consulter l'ensemble des couches dans une même arboresence en utilisant le paramètre ``request=list`` (au lieu de request=layer).
*Exemple* : https://download.data.grandlyon.com/kml/grandlyon?request=list

Services MVT
------------

Les jeux de donnés vectoriels sont disponible au format MVT (Mapbox Vector Tile)[https://docs.mapbox.com/vector-tiles/specification/]

https://download.data.grandlyon.com/mvt/grandlyon?LAYERS=car_care.carencadrmtloyer_latest&map.imagetype=mvt&tilemode=gmap&tile=1052+730+11&mode=tile

Ce format est comparable au WFS mais est tuilé et les geométries sont simplifiés. Le but est d'être beaucoup plus rapide que le WFS en permettant en plus d'être mis en cache. En sortie on obtient une tuile encodée en utilisant le format (protobuf)[https://fr.wikipedia.org/wiki/Protocol_Buffers] (équivalenet plus compact du JSON)

Ces tuiles peuvent être utilisés par les client web comme MapboxGL, MapLibre ou OpenLayers.

QGis peut aussi lire ces tuiles en utilisant le plugin "Vector Tiles Reader"


Geocoder Photon
---------------

Ce service permet d'effectuer des géocodages directes (conversion d'une adresse postale ou nom de lieu en coordonnées géographiques) et inversés (conversion de coordonnées géographiques en adresse postale ou nom de lieu).

Il est propulsé par l'outil libre Photon (cf. https://github.com/komoot/photon), alimenté par les données OpenStreetMap relatives à l'ancienne région Rhône-Alpes (cf. https://download.geofabrik.de/europe/france/rhone-alpes.html). 

La documentation officielle de l'API de recherche de Photon est renseignée sur GitHub, https://github.com/komoot/photon#search-api.

Le lien pour réaliser une requête est le suivant (remplacer les .. par le lieu à géocoder): 
https://download.data.grandlyon.com/geocoding/photon/api?q=...
Exemples : 
https://download.data.grandlyon.com/geocoding/photon/api?q=lyon 
https://download.data.grandlyon.com/geocoding/photon/api?q=%22Rue%20garibaldi%22


Statistiques liées à un jeu de données
--------------------------------------

Pour interroger les statistiques, les requêtes prennent la forme suivante :
`https://download.data.grandlyon.com/statistiques/dataset?UUID=ad771ef2-7a40-11ea-ae2d-dbfa27ebc23c&start=2019-04-09&end=2020-04-09&granularity=week`

```json
[
  {"date": "2019-04-08", "service":"wms", "count":362},
  {"date": "2019-04-08", "service":"wfs", "count":123},
  {"date": "2019-04-08", "service":"ws", "count":12},
  {"date": "2019-04-08", "service":"kml", "count":2},
  {"date": "2019-04-15", "service":"wms", "count":364},
  {"date": "2019-04-15", "service":"wfs", "count":125},
  {"date": "2019-04-15", "service":"ws", "count":10},
  {"date": "2019-04-15", "service":"kml", "count":4},
  ...
  {"date": "2020-03-30", "service":"wms", "count":462},
  {"date": "2020-03-30", "service":"wfs", "count":223},
  {"date": "2020-03-30", "service":"ws", "count":22},
  {"date": "2020-03-30", "service":"kml", "count":50},
  {"date": "2020-04-06", "service":"wms", "count":202},
  {"date": "2020-04-06", "service":"wfs", "count":113},
  {"date": "2020-04-06", "service":"ws", "count":22},
  {"date": "2020-04-06", "service":"kml", "count":7},
]
```


Les paramètres demandés (tous insensibles à la casse):

 * UUID l'uuid du jeux de données
 * start : format YYYY-MM-DD la date de début, si la granularité est la semaine ou le mois et que la date est "dans" la semaine/mois, je prends le début de la semaine/mois
 * end : format YYYY-MM-DD la date de fin, si la granularité est la semaine ou le mois et que la date est "dans" la semaine/mois, je prends la fin de semaine/mois
 * granularity : day, week, year.

Pour la réponse :

Tableau de dictionnaire : un dictionnaire par élément de granularité (jour, semaine, mois)
Le dictionnaire contient:
 * date: format YYYY-MM-DD la date de début de élément de granularité
 * count: le nombre de fois que la ressource a été vue, indépendamment du service (WMS/WFS...)
