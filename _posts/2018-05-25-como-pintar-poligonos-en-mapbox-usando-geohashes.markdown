---
layout: post
title: "Cómo pintar polígonos en Mapbox usando geohashes"
date: 2018-05-25 20:42:27 -0500
comments: true
featured_image: '/images/tech/mapbox_polygons.png'
disqus_identifier: 096db109-bde5-43e8-8625-f6050d23186e
excerpt_separator: <!-- more -->
categories:
- Desarrollo
tags:
- Mapas
- Mapbox
---

En nuestro ejemplo tenemos información acerca de velocidades (*km/h*) 
para cada sector de nuestra ciudad representado por un geohash:

<!-- more -->

`geohashes.json`:

```json 
{
    "d2g62s": 18.0,
    "d2g68c": 36.33,
    "d2g74k": 21.32
    // etc.
}
```

Lo primero que vamos a hacer, es obtener las coordenadas de los sectores a partir de la cadena de geohash. Para esto vamos a usar una librería de Python llamanda [python-geohash](https://github.com/hkwi/python-geohash). Usando el método `bbox` de ésta librería, podemos potener las coordenadas de los puntos extremos:

```python
> import geohash
> gh = geohash.bbox('d2g628')
{'e': -74.146728515625,
 'n': 4.6197509765625,
 's': 4.6142578125,
 'w': -74.15771484375}
```

donde `n` es el norte, `s` - el sur,  `w` - el oeste y `e` es el este. Entonces los puntos serán `(-74.15771484375, 4.6197509765625)`, `(-74.146728515625, 4.6197509765625)`, `(-74.14672851562, 4.6142578125)` y `(-74.15771484375, 4.6142578125)`.

Ahora construimos una vista (en éste ejemplo usamos Flask) para devolver un [GeoJSON](http://leafletjs.com/examples/geojson/), que vamos a consumir desde el código JavaScript.

`views.py`:

```python
import geohash

@main.route('/maps/speeds-map-data')
def speeds_map_data():
    features = []
    speeds_hash = get_speeds_json()
    for hash, speed in speeds_hash.iteritems():
        gh = geohash.bbox(hash)
        features.append({
            'type': 'Feature',
            'properties': {
                'popupContent': '{} km/h'.format(speed),
                'style': {
                    'weight': 2,
                    'color': "#999",
                    'opacity': 1,
                    'fillColor': color,
                    'fillOpacity': 0.7
                },
            },
            'geometry': {
                'type': 'Polygon',
                'coordinates': [[
                    [gh['w'], gh['n']], [gh['e'], gh['n']],
                    [gh['e'], gh['s']], [gh['w'], gh['s']],
                ]]
            }
        })
    response = {
        'type': 'FeatureCollection',
        'features': features,
    }
    return jsonify(response)
```

Ahora sólo nos falta crear la plantilla HTML

`map.html`:

```html
<div id="map"></div>
```

y el archivo JavaScript para crear el mapa y obtener los datos de nuestra vista:

`map.js`:

```javascript
$(function () {
  'use strict';

  L.mapbox.accessToken = 'YOUR_MAPBOX_ACCESS_TOKEN';

  // Crear un mapa, especificando el centro y zoom
  var map = L.mapbox.map('map', 'mapbox.streets').setView([4.66336863727521, -74.07231699675322], 12)

  // Obtener el GeoJSON de nuestra vista
  $.get('/maps/speeds-map-data', function (data) {
    L.geoJson(data, {
      style: function(feature) {
        return feature.properties.style;
      },
      onEachFeature: function (feature, layer) {
        // Mostrar un pop-up con la velocidad mara cada sector
        var popupContent = '';
        if (feature.properties && feature.properties.popupContent) {
          popupContent += feature.properties.popupContent;
        }
        layer.bindPopup(popupContent);
      }
    }).addTo(map);
  });

});
```

Y voilà: podemos mostrar un mapa de calor a nuestros usuarios:

![](/images/tech/mapbox_polygons.png)

#### Referentes

* [Using GeoJSON with Leaflet](http://leafletjs.com/examples/geojson/)
* [The GeoJSON Format Specification](http://geojson.org/geojson-spec.html)
