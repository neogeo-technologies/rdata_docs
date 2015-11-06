.. _exemples:

Exemples et extraits de code
==============================

WMS avec OpenLayers
-------------------
Cet exemple montre l'utilisation du service WFS avec les bornes vélo'v en temps réel à travers la bibliothèque OpenLayers.

.. image:: _static/openlayers.png

Code source correspondant :

.. code-block:: html

    <html>
      <head>
        <title>Utilisation des services GrandLyon Data : OpenLayers</title>
        <script src="http://openlayers.org/api/OpenLayers.js"></script>
      </head>
        <body>
          <div style="width:100%; height:100%" id="map"></div>
          <script defer="defer" type="text/javascript">
            var map = new OpenLayers.Map('map');
            var osm = new OpenLayers.Layer.OSM('Simple OSM Map', null, {
              //conversion en valeurs de gris à la volée
              eventListeners: {
                tileloaded: function(evt) {
                  var ctx = evt.tile.getCanvasContext();
                  if (ctx) {
                    var imgd = ctx.getImageData(0, 0, evt.tile.size.w, evt.tile.size.h);
                    var pix = imgd.data;
                    for (var i = 0, n = pix.length; i < n; i += 4) {
                      pix[i] = pix[i + 1] = pix[i + 2] = (3 * pix[i] + 4 * pix[i + 1] + pix[i + 2]) / 8;
                    }
                    ctx.putImageData(imgd, 0, 0);
                    evt.tile.imgDiv.removeAttribute("crossorigin");
                    evt.tile.imgDiv.src = ctx.canvas.toDataURL();
                  }
                }
              }
            });
            //URL du service data
            var data_url = "https://download.data.grandlyon.com/wfs/rdata?";
            //Définition du proxy pour gérer le cross-domain. Voir le paragraphe Bonne Pratiques -> Proxyfication pour plus d'information
            OpenLayers.ProxyHost = "/cgi-bin/proxy.cgi?url=";
            
            //Styles pour le rendu
            var colors = ["green", "blue", "orange", "grey"];
            var context = {
                getColor: function(feature) {  
                    return colors[feature.data.availabilitycode - 1];
                }
            }  
            var template = {
              pointRadius: 15,
              fillColor: "${getColor}" // using context.getColor(feature)
            };
            var style = new OpenLayers.Style(template, {context: context});
            
            //Définition du layer WFS
            var wfs = new OpenLayers.Layer.Vector("WFS GL Data", {
                strategies: [new OpenLayers.Strategy.BBOX()],
                protocol: new OpenLayers.Protocol.WFS({
                    version: "1.1.0",
                    srsName: "EPSG:4326",
                    url: data_url,
                    featurePrefix : 'ms',
                    featureType: "jcd_jcdecaux.jcdvelov",
                    geometryName: "msGeometry",
                    formatOptions: {
                      xy: false
                    }
                }),
                styleMap: new OpenLayers.StyleMap(style),
                renderers: OpenLayers.Layer.Vector.prototype.renderers
            });
                     
            //gestion du click sur les markers
            var selectControl = new OpenLayers.Control.SelectFeature(wfs);
            map.addControl(selectControl);
            selectControl.activate();
            
            wfs.events.on({ 
              featureselected: function(event) {
                var feature = event.feature;
                feature.popup = new OpenLayers.Popup.FramedCloud("box",
                    feature.geometry.getBounds().getCenterLonLat(),
                    null,
                    '<div><b>'+feature.data.name + '</b> (station '+feature.data.number+')<br/>'
                    + 'Il reste <b>' + feature.data.available_bikes + '</b> v&eacute;los disponibles et '
                    + '<b>' + feature.data.available_bike_stands + ' </b>bornes libres</div>',
                    null,
                    true
                );
                while( map.popups.length ) {
                    map.removePopup( map.popups[0] );
                }
                map.addPopup(feature.popup);
                }
            });
    
            //Config de la map
            map.addLayers([osm, wfs]);
            var zoom = 15;
            var lonLat = new OpenLayers.LonLat(4.85,45.76);
            map.setCenter(
                lonLat.transform(
                    new OpenLayers.Projection("EPSG:4326"),
                    map.getProjectionObject()
                ), zoom
            ); 
    
            </script>
        </body>
    </html>


WFS avec Leaflet
----------------
Cet exemple montre l'utilisation du service WFS avec les bornes vélo'v en temps réel à travers la bibliothèque LeafLet.

.. image:: _static/leaflet.png

Code source correspondant :

.. code-block:: html

    <html>
      <head>
        <title>Utilisation des services GrandLyon Data : Leaflet</title>
        <meta charset="utf-8" />

        <meta name="viewport" content="width=device-width, initial-scale=1.0">
                
        <script src="leaflet.js"></script>
        <script src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
        
        <link rel="stylesheet" href="leaflet.css" />
        <style>
            body {
                    padding: 0;
                    margin: 0;
            }
            html, body, #map {
                    height: 100%;
            }
        </style>
      </head>
      <body>
        <div id="map"></div>
        <script>
            //Initialisation de la map
            var map = L.map('map').setView([45.76, 4.85], 14);
            //Layer WMS sur une orthophoto
            L.tileLayer.wms("https://download.data.grandlyon.com/wms/grandlyon",{
                    layers: '1840_5175_16_CC46',
                    format: 'image/png',
                    transparent: true,    
                    opacity: 0.6       
            }).addTo(map);
            //Layer WMS openstreetmap
            L.tileLayer.wms("http://openstreetmap.wms.data.grandlyon.com/default",{
                    layers: 'default',
                    format: 'image/png', 
                    transparent: true,    
                    opacity: 0.7       
            }).addTo(map);
            
            //Définition du proxy pour le WFS (cross domain). Voir le paragraphe Bonne Pratiques -> Proxyfication pour plus d'information
            var proxy = "proxy.php?url=";
            var data_url = "https://secure.grandlyon.webmapping.fr/wfs/rdata";
            var params = '?SERVICE=WFS
                &REQUEST=GetFeature
                &VERSION=1.1.0
                &TYPENAME=jcd_jcdecaux.jcdvelov
                &outputformat=geojson';
            
            var VertIcon = L.icon({
                iconUrl: 'images/cycling_Vert.png',
                iconSize:     [33, 21]
            });
            var OrangeIcon = L.icon({
                iconUrl: 'images/cycling_Orange.png',
                iconSize:     [33, 21]
            });
            var BleuIcon = L.icon({
                iconUrl: 'images/cycling_Bleu.png',
                iconSize:     [33, 21]
            });
            var GrisIcon = L.icon({
                iconUrl: 'images/cycling_Gris.png',
                iconSize:     [33, 21]
            });
		
            $.get(proxy + encodeURIComponent(data_url + params), function(json){
                var obj = $.parseJSON(json);
                // Add markers
                for(i=0;i<obj.features.length;i++) {
                    //create feature from json
                    var ftr = obj.features[i];
                    // set marker options from properties
                    var options = {
                        gid: ftr.properties.gid,
                        number: ftr.properties.number,
                        name: ftr.properties.name,
                        available_bikes: ftr.properties.available_bikes,
                        available_bike_stands: ftr.properties.available_bike_stands
                    };
                    //set marker icon from availability
                    switch(ftr.properties.availability){
                        case 'Vert':
                            options.icon = VertIcon;
                            break;
                        case 'Orange':
                            options.icon = OrangeIcon;
                            break;
                        case 'Bleu' :
                            options.icon = BleuIcon;
                            break;
                        default :
                            options.icon = GrisIcon;
                    }
                    //ajout du marker à la map
                    var point = L.marker(
                        [ftr.geometry.coordinates[1],ftr.geometry.coordinates[0]],
                        options
                    ).addTo(map);
                    //définition de la popup sur le click
                    point.bindPopup(
                        '<b>'+ point.options.name + '</b> (station '+point.options.number+')<br/>'
                        + 'Il reste <b>' + point.options.available_bikes + '</b> v&eacute;los disponibles'
                        + ' et <b>' + point.options.available_bike_stands + ' </b>bornes libres',
                        {
                        closeButton: false
                        }
                    );
                        
                }
            });

            </script>
        </body>
    </html>


KML avec l'API Maps de Google
------------------------------------

Cet exemple montre l'utilisation du service KML avec les bornes vélo'v à travers l'API Google Maps v3. Nécessite une clé pour l'API.

.. image:: _static/google.png

Code source correspondant :

.. code-block:: html
   
    <html>
      <head>
        <title>Utilisation des services GrandLyon Data : Google API</title>
        <meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
        <style type="text/css">
          html { height: 100% }
          body { height: 100%; margin: 0; padding: 0 }
          #map-canvas { height: 100% }
        </style>
        <script type="text/javascript"
            src="https://maps.googleapis.com/maps/api/js?key=API_KEY&sensor=false">
        </script>
    
        <script type="text/javascript">
          function initialize() {
            //Init map
            var mapOptions = {
              center: new google.maps.LatLng(45.76, 4.85),
              zoom: 13
            };
            var map = new google.maps.Map(document.getElementById("map-canvas"),
                mapOptions);
            
            //Ajout WMS sur l'aménagement cyclable
            var urlWMS = "https://download.data.grandlyon.com/wms/grandlyon?"
                    + "&REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&CRS=EPSG:4171"
                    + "&LAYERS=pvo_patrimoine_voirie.pvoamenagementcyclable"
                    + "&FORMAT=image/png&TRANSPARENT=TRUE&WIDTH=256&HEIGHT=256";
            //L'API Google Map ne gère pas directement le WMS, il faut passer par un ImageMapType        
            var WMS_Layer = new google.maps.ImageMapType({
                getTileUrl: function (coord, zoom) {
                    var projection = map.getProjection();
                    var zoomfactor = Math.pow(2, zoom);
                    var LL_upperleft = projection.fromPointToLatLng(
                        new google.maps.Point(
                            coord.x * 256 / zoomfactor,
                            coord.y * 256 / zoomfactor
                        )
                    );
                    var LL_lowerRight = projection.fromPointToLatLng(
                        new google.maps.Point(
                            (coord.x + 1) * 256 / zoomfactor,
                            (coord.y + 1) * 256 / zoomfactor
                        )
                    );
                    var bbox =  "&bbox="
                        + LL_lowerRight.lat() + "," + LL_upperleft.lng() + ","
                        + LL_upperleft.lat() + "," + LL_lowerRight.lng();						   
                    var url = urlWMS + bbox;
                    return url;
                },
                tileSize: new google.maps.Size(256, 256),
                isPng: true
            });
            
            map.overlayMapTypes.push(WMS_Layer);
            
            //Ajout KML layer
            var KML_Layer = new google.maps.KmlLayer({
              url: 'https://download.data.grandlyon.com/kml/grandlyon/?'
                +'request=layer&typename=pvo_patrimoine_voirie.pvostationvelov'
            });
            KML_Layer.setMap(map);
      
          }
          google.maps.event.addDomListener(window, 'load', initialize);
        </script>
        
      </head>
      <body>
        <div id="map-canvas"/>
      </body>
    </html>
    
    
Utilisation du WCS
-------------------
Cet exemple montre l'utilisation du service WCS pour obtenir une ortophoto sur une zone de travail.

**Phase 1** : lecture des capacités du service

https://download.data.grandlyon.com/wcs/grandlyon?SERVICE=WCS&REQUEST=GetCapabilities&VERSION=1.0.0

.. code-block:: xml

	<WCS_Capabilities xmlns="http://www.opengis.net/wcs" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:gml="http://www.opengis.net/gml" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="1.0.0" updateSequence="0" xsi:schemaLocation="http://www.opengis.net/wcs http://schemas.opengis.net/wcs/1.0.0/wcsCapabilities.xsd">
		<Service>
			<description>
			Base de la plateforme d'intégration de données géographiques du Grand Lyon. Données sous Licence Ouverte (Etalab).
			</description>
			<name>MapServer WCS</name>
			<label>Serveur WCS du GrandLyon</label>
			<responsibleParty>
				<individualName>Diffusion de données et géoservices</individualName>
				<organisationName>
				Grand Lyon - Direction des systèmes d'informations et de télécommunications
				</organisationName>
				<positionName>owner</positionName>
				<contactInfo>
				<address>
					<deliveryPoint>20, rue du Lac - BP 31 03</deliveryPoint>
					<city>Lyon cedex 03</city>
					<administrativeArea>Rhône-Alpes</administrativeArea>
					<postalCode>69399</postalCode>
					<country>France</country>
					<electronicMailAddress>smartdata@grandlyon.org</electronicMailAddress>
				</address>
				<onlineResource xlink:type="simple" xlink:href="https://download.data.grandlyon.com/wcs/grandlyon"/>
				</contactInfo>
			</responsibleParty>
			<fees>no conditions apply</fees>
			<accessConstraints>None</accessConstraints>
		</Service>
		<Capability>...</Capability>
		<ContentMetadata>
			<CoverageOfferingBrief>
				<name>Ortho2009_vue_ensemble_16cm_CC46</name>
				<lonLatEnvelope srsName="urn:ogc:def:crs:OGC:1.3:CRS84">
					<gml:pos>4.66488945660669 45.5384488998787</gml:pos>
					<gml:pos>5.17955354166403 45.9426997122181</gml:pos>
				</lonLatEnvelope>
			</CoverageOfferingBrief>
			<CoverageOfferingBrief>
				<name>1830_5155_16_CC46</name>
				<lonLatEnvelope srsName="urn:ogc:def:crs:OGC:1.3:CRS84">
					<gml:pos>4.66596079716618 45.5819080900619</gml:pos>
					<gml:pos>4.73140908341002 45.6278468986858</gml:pos>
				</lonLatEnvelope>
			</CoverageOfferingBrief>
			<CoverageOfferingBrief>...</CoverageOfferingBrief>
		</ContentMetadata>
	</WCS_Capabilities>
	
**Phase 2** : détail d'une coverage 

https://download.data.grandlyon.com/wcs/grandlyon?SERVICE=WCS&REQUEST=DescribeCoverage&VERSION=1.0.0&COVERAGE=1830_5155_16_CC46

.. code-block:: xml

	<CoverageDescription xmlns="http://www.opengis.net/wcs" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:gml="http://www.opengis.net/gml" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="1.0.0" updateSequence="0" xsi:schemaLocation="http://www.opengis.net/wcs http://schemas.opengis.net/wcs/1.0.0/describeCoverage.xsd">
		<CoverageOffering>
			<name>1830_5155_16_CC46</name>
			<lonLatEnvelope srsName="urn:ogc:def:crs:OGC:1.3:CRS84">
				<gml:pos>4.66596079716618 45.5819080900619</gml:pos>
				<gml:pos>4.73140908341002 45.6278468986858</gml:pos>
			</lonLatEnvelope>
			<domainSet>
				<spatialDomain>
					<gml:Envelope srsName="EPSG:4326">
						<gml:pos>4.66596079716618 45.5819080900619</gml:pos>
						<gml:pos>4.73140908341002 45.6278468986858</gml:pos>
					</gml:Envelope>
					<gml:Envelope srsName="EPSG:3946">
						<gml:pos>1830000 5155000</gml:pos>
						<gml:pos>1835000 5160000</gml:pos>
					</gml:Envelope>
					<gml:RectifiedGrid dimension="2">
						<gml:limits>
							<gml:GridEnvelope>
								<gml:low>0 0</gml:low>
								<gml:high>31249 31249</gml:high>
							</gml:GridEnvelope>
						</gml:limits>
						<gml:axisName>x</gml:axisName>
						<gml:axisName>y</gml:axisName>
						<gml:origin>
							<gml:pos>1830000 5160000</gml:pos>
						</gml:origin>
						<gml:offsetVector>0.16 0</gml:offsetVector>
						<gml:offsetVector>0 -0.16</gml:offsetVector>
					</gml:RectifiedGrid>
				</spatialDomain>
			</domainSet>
			<supportedCRSs>
				<requestResponseCRSs>EPSG:3946</requestResponseCRSs>
				<nativeCRSs>EPSG:3946</nativeCRSs>
			</supportedCRSs>
			<supportedFormats>
				<formats>GTiff</formats>
			</supportedFormats>
			<supportedInterpolations default="nearest neighbor">
				<interpolationMethod>nearest neighbor</interpolationMethod>
				<interpolationMethod>bilinear</interpolationMethod>
			</supportedInterpolations>
		</CoverageOffering>
	</CoverageDescription>

**Phase 3** : obtention de l'image sur une zone

https://download.data.grandlyon.com/wcs/grandlyon?SERVICE=WCS&VERSION=1.0.0&REQUEST=GetCoverage&FORMAT=GTiff&COVERAGE=1830_5155_16_CC46&BBOX=1832784,5156714.08000000007450581,1834141.43999999994412065,5158023.36000000033527613&CRS=EPSG:3946&RESPONSE_CRS=EPSG:3946&WIDTH=849&HEIGHT=819

.. image:: _static/wcs.png
