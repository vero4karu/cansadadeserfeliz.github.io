---
layout: post
title: "Pintar polígonos en diferentes colores desde GeoJSON en Mapbox GL JS"
date: 2017-08-17 09:36:28 -0500
comments: true
published: true
featured_image: '/images/posts/mapbox-polygons-categorical-color.png'
disqus_identifier: 0704b99c-2f33-490c-aefd-58b3956d4468
excerpt_separator: <!-- more -->
categories: 
- Mapas
tags:
- Mapbox
- JavaScript
---

Tenemos un listado de de geohashes y a cada [geohash](https://en.wikipedia.org/wiki/Geohash) corresponde un valor, por ejemplo `fulfillment` en nuestro caso. Queremos mostrar polígonos en un mapa, que corresponden a cada geohash y pintados en colores diferentes, dependiendo de valor de `fulfillment`.

<!-- more -->

Primero [creamos un mapa de mapbox](https://www.mapbox.com/mapbox-gl-js/example/multiple-geometries/) (`var map;`) y asociamos un fuente de datos (url donde tenemos nuestro GeoJSON):

```javascript
map.addSource('fulfillment', {
  'type': 'geojson',
  'data': 'http://your.website.com/data.json'
});
```

Mapbox nos permite, usando el atributo `paint`, especificar qué color va a tener cada polígono (`fill-color` y `fill-opacity`):

```javascript
$(function () {
  'use strict';

  mapboxgl.accessToken = 'your-mapbox-key';

  var map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/mapbox/streets-v9',
    zoom: 12,
    center: [-74.07231699675322, 4.66336863727521]
  });

  map.on('load', function() {
    map.addSource('fulfillment', {
      'type': 'geojson',
      'data': 'http://your.website.com/data.json'
    });

    map.addLayer({
        "id": "fulfillment-polygon",
        "type": "fill",
        "source": "fulfillment",
        "paint": {
            "fill-color": "#888888",
            "fill-opacity": 0.4
        },
        "filter": ["==", "$type", "Polygon"]
    });
  });
});
```

Como resultado tenemos un mapa con polígonos pintados del mismo color gris:

![](/images/posts/mapbox-polygons-one-color.png)

Para especificar qué color exactamente para cada valor de fulfillment, vamos a usar [el atributo](https://www.mapbox.com/mapbox-gl-js/style-spec/#function-type) `fill-color`. Así podemos decir para a qué rango de valores de fulfillment qué color corresponde:

```javascript
map.addLayer({
  "id": "fulfillment-polygon",
  "type": "fill",
  "source": "fulfillment",
  "paint": {
    "fill-color": {
      property: 'fulfillment',
      type: 'categorical',
      stops: [
        [0, '#a50026'],
        [30, '#f46d43'],
        [50, '#fdae61'],
        [60, "#ffffbf"],
        [70, "#a6d96a"],
        [80, "#66bd63"],
        [90, "#006837"]
      ]
    },
    "fill-opacity": 0.4
  }
});
```

Nuestro GeoJSON de entrada:

```json
{
  "datetime": "17 ago. 2017 13:10:40", 
  "features": [
    {
      "geometry": {
        "coordinates": [
          [
            [
              -74.20166015625, 
              4.581298828125
            ], 
            [
              -74.190673828125, 
              4.581298828125
            ], 
            [
              -74.190673828125, 
              4.5758056640625
            ], 
            [
              -74.20166015625, 
              4.5758056640625
            ]
          ]
        ], 
        "type": "Polygon"
      }, 
      "properties": {
        "fulfillment": 50.0, 
        "geohash": "d2g4p9"
      }, 
      "type": "Feature"
    }, 
    {
      "geometry": {
        "coordinates": [
          [
            [
              -74.091796875, 
              4.76806640625
            ], 
            [
              -74.080810546875, 
              4.76806640625
            ], 
            [
              -74.080810546875, 
              4.7625732421875
            ], 
            [
              -74.091796875, 
              4.7625732421875
            ]
          ]
        ], 
        "type": "Polygon"
      }, 
      "properties": {
        "fulfillment": 30.0, 
        "geohash": "d2g745"
      }, 
      "type": "Feature"
    }
  ], 
  "type": "FeatureCollection"
}
```

La otra forma de hacer lo mismo, es tener el color dentro de GeoJSON de entrada. Así en lugar de `steps`, podemos usar el tipo `identity` y tener los colores en el atributo `color` que especificamos en `property` (por ejemplo, `#a50026`):

```json
map.addLayer({
  "id": "fulfillment-polygon",
  "type": "fill",
  "source": "fulfillment",
  "paint": {
    "fill-color": {
      property: 'color',
      type: 'identity'
    },
    "fill-opacity": 0.4
  }
});
```

Así se ve el resultado:

![](/images/posts/mapbox-polygons-categorical-color.png)