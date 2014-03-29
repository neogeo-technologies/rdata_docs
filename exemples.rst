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
        <title>Utilisation des services GrandLyon Smart Data : OpenLayers</title>
        <script src="http://openlayers.org/api/OpenLayers.js"></script>
      </head>
        <body>
          <div style="width:100%; height:100%" id="map"></div>
          <script defer="defer" type="text/javascript">
            var map = new OpenLayers.Map('map');
            var osm = new OpenLayers.Layer.OSM('Simple OSM Map', null, {
              //dynamic conversion in grey values
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
        
            var smartdata_url = "https://secure.grandlyon.webmapping.fr/wfs/smartdata?";
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
            var wfs = new OpenLayers.Layer.Vector("WFS Smartdata", {
                strategies: [new OpenLayers.Strategy.BBOX()],
                protocol: new OpenLayers.Protocol.WFS({
                    version: "1.1.0",
                    srsName: "EPSG:4326",
                    url: smartdata_url,
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
        <title>Utilisation des services GrandLyon Smart Data : Leaflet</title>
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
            var map = L.map('map').setView([45.76, 4.85], 14);

            L.tileLayer.wms("https://download.data.grandlyon.com/wms/grandlyon",{
                    layers: '1840_5175_16_CC46',
                    format: 'image/png',
                    transparent: true,    
                    opacity: 0.6       
            }).addTo(map);
            
            L.tileLayer.wms("http://openstreetmap.wms.data.grandlyon.com/default",{
                    layers: 'default',
                    format: 'image/png', 
                    transparent: true,    
                    opacity: 0.7       
            }).addTo(map);
            
            var proxy = "proxy.php?url=";
            var smartdata_url = "https://secure.grandlyon.webmapping.fr/wfs/smartdata";
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
		
            $.get(proxy + encodeURIComponent(smartdata_url + params), function(json){
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
                    //add marker to map
                    var point = L.marker(
                        [ftr.geometry.coordinates[1],ftr.geometry.coordinates[0]],
                        options
                    ).addTo(map);
                    //define popup on click
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
        <title>Utilisation des services GrandLyon Smart Data : Google API</title>
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
            
            //Add WMS layer
            var urlWMS = "https://download.data.grandlyon.com/wms/grandlyon?"
                    + "&REQUEST=GetMap&SERVICE=WMS&VERSION=1.3.0&CRS=EPSG:4171"
                    + "&LAYERS=pvo_patrimoine_voirie.pvoamenagementcyclable"
                    + "&FORMAT=image/png&TRANSPARENT=TRUE&WIDTH=256&HEIGHT=256";
                    
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
            
            //Add KML layer
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