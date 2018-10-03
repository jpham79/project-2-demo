# project-2-demo

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/sRl6yWzZy8M/0.jpg)](http://www.youtube.com/watch?v=sRl6yWzZy8M)

## Optimized Route

This demo will show you how to:
Create a map
Plot points
Use Mapbox’s API

Quickstart:

Sign up for Mapbox

Include Javascript and CSS files in the ```<head>``` of you HTML file.

```
<script src='https://api.tiles.mapbox.com/mapbox-gl-js/v0.49.0/mapbox-gl.js'></script>
<link href='https://api.tiles.mapbox.com/mapbox-gl-js/v0.49.0/mapbox-gl.css' rel='stylesheet' />
```
Include the following in the ```<body>``` of your HTML file.
```
<div id='map' style='width: 400px; height: 300px;'></div>
<script>
mapboxgl.accessToken = 'YOUR ACCESS TOKEN HERE';
var map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/mapbox/streets-v9'
});
</script>
```

More complex demo:
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset='utf-8' />
    <title>Delivery App</title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <script src='https://npmcdn.com/@turf/turf/turf.min.js'></script>
    <script src='https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js'></script>
    <script src='https://api.tiles.mapbox.com/mapbox-gl-js/v0.49.0/mapbox-gl.js'></script>
    <link href='https://api.tiles.mapbox.com/mapbox-gl-js/v0.49.0/mapbox-gl.css' rel='stylesheet' />
    <style>
      
      body {
        margin: 0;
        padding: 0;
      }

      #map {
        position: absolute;
        top: 0;
        bottom: 0;
        right: 0;
        left: 0;
      }
    </style>
  </head>
  <body>
    <div id='map' class='contain'></div>
    <script>
      var truckLocation = [-83.093, 42.376];
      var warehouseLocation = [-83.083, 42.363];
      var lastQueryTime = 0;
      var lastAtRestaurant = 0;
      var keepTrack = [];
      var currentSchedule = [];
      var currentRoute = null;
      var pointHopper = {};
      var pause = true;
      var speedFactor = 50;

      // Add your access token
      mapboxgl.accessToken = 'pk.eyJ1Ijoia2lyazYiLCJhIjoiY2ptczlzbTM4MDRtbTN2czhyeTVua2V5bCJ9._s7C27cvwfR1-haPFPSe3Q';

      // Initialize a map
      var map = new mapboxgl.Map({
        container: 'map', // container id
        style: 'mapbox://styles/mapbox/light-v9', // stylesheet location
        center: truckLocation, // starting position
        zoom: 12 // starting zoom
      });
    </script>
  </body>
</html>
```
![alt text](https://github.com/kirkwilson/project-3-demo/blob/master/optimization-step-one.png)
### Adding a truck
Between ```<style>``` tags:
```
.truck {
  margin: -10px -10px;
  width: 20px;
  height: 20px;
  border: 2px solid #fff;
  border-radius: 50%;
  background: #3887be;
  pointer-events: none;
}
```
When the map loads:
```
map.on('load', function() {
  var marker = document.createElement('div');
  marker.classList = 'truck';

  // Create a new marker
  truckMarker = new mapboxgl.Marker(marker)
    .setLngLat(truckLocation)
    .addTo(map);
});
```
### Adding a warehouse location

```
// Create a GeoJSON feature collection for the warehouse
var warehouse = turf.featureCollection([turf.point(warehouseLocation)]);
```
In ```map.on(‘load, function() {``` function:
```
// Create a circle layer
map.addLayer({
  id: 'warehouse',
  type: 'circle',
  source: {
    data: warehouse,
    type: 'geojson'
  },
  paint: {
    'circle-radius': 20,
    'circle-color': 'white',
    'circle-stroke-color': '#3887be',
    'circle-stroke-width': 3
  }
});

// Create a symbol layer on top of circle layer
map.addLayer({
  id: 'warehouse-symbol',
  type: 'symbol',
  source: {
    data: warehouse,
    type: 'geojson'
  },
  layout: {
    'icon-image': 'grocery-15',
    'icon-size': 1
  },
  paint: {
    'text-color': '#3887be'
  }
});
```
![alt text](https://github.com/kirkwilson/project-3-demo/blob/master/optimization-step-two.png)
### Drop off locations
```
// Create an empty GeoJSON feature collection for drop-off locations
var dropoffs = turf.featureCollection([]);
```

```
map.addLayer({
  id: 'dropoffs-symbol',
  type: 'symbol',
  source: {
    data: dropoffs,
    type: 'geojson'
  },
  layout: {
    'icon-allow-overlap': true,
    'icon-ignore-placement': true,
    'icon-image': 'marker-15',
  }
});
```
Add click listener:
```
// Listen for a click on the map
map.on('click', function(e) {
  // When the map is clicked, add a new drop-off point
  // and update the `dropoffs-symbol` layer
  newDropoff(map.unproject(e.point));
  updateDropoffs(dropoffs);
});
```
Outside of ```map.on('load', function()```
```
function newDropoff(coords) {
  // Store the clicked point as a new GeoJSON feature with
  // two properties: `orderTime` and `key`
  var pt = turf.point(
    [coords.lng, coords.lat],
    {
      orderTime: Date.now(),
      key: Math.random()
    }
  );
  dropoffs.features.push(pt);
}

function updateDropoffs(geojson) {
  map.getSource('dropoffs-symbol')
    .setData(geojson);
}
```
![alt text](https://github.com/kirkwilson/project-3-demo/blob/master/optimization-step-three.gif)
### Build routes

Same as last step:

Empty JSON for the routes
```
// Create an empty GeoJSON feature collection, which will be used as the data source for the route before users add any new data
var nothing = turf.featureCollection([]);
```
```map.addSource('route', {
  type: 'geojson',
  data: nothing
});

map.addLayer({
  id: 'routeline-active',
  type: 'line',
  source: 'route',
  layout: {
    'line-join': 'round',
    'line-cap': 'round'
  },
  paint: {
    'line-color': '#3887be',
    'line-width': {
      base: 1,
      stops: [[12, 3], [22, 12]]
    }
  }
}, 'waterway-label');
```
### Using the API
Build a request then make a request.
```
// Here you'll specify all the parameters necessary for requesting a response from the Optimization API
function assembleQueryURL() {

  // Store the location of the truck in a variable called coordinates
  var coordinates = [truckLocation];
  var distributions = [];
  keepTrack = [truckLocation];

  // Create an array of GeoJSON feature collections for each point
  var restJobs = objectToArray(pointHopper);

  // If there are actually orders from this restaurant
  if (restJobs.length > 0) {

    // Check to see if the request was made after visiting the restaurant
    var needToPickUp = restJobs.filter(function(d, i) {
      return d.properties.orderTime > lastAtRestaurant;
    }).length > 0;

    // If the request was made after picking up from the restaurant,
    // Add the restaurant as an additional stop
    if (needToPickUp) {
      var restaurantIndex = coordinates.length;
      // Add the restaurant as a coordinate
      coordinates.push(warehouseLocation);
      // push the restaurant itself into the array
      keepTrack.push(pointHopper.warehouse);
    }

    restJobs.forEach(function(d, i) {
      // Add dropoff to list
      keepTrack.push(d);
      coordinates.push(d.geometry.coordinates);
      // if order not yet picked up, add a reroute
      if (needToPickUp && d.properties.orderTime > lastAtRestaurant) {
        distributions.push(restaurantIndex + ',' + (coordinates.length - 1));
      }
    });
  }

  // Set the profile to `driving`
  // Coordinates will include the current location of the truck,
  return 'https://api.mapbox.com/optimized-trips/v1/mapbox/driving/' + coordinates.join(';') + '?distributions=' + distributions.join(';') + '&overview=full&steps=true&geometries=geojson&source=first&access_token=' + mapboxgl.accessToken;
}

function objectToArray(obj) {
  var keys = Object.keys(obj);
  var routeGeoJSON = keys.map(function(key) {
    return obj[key];
  });
  return routeGeoJSON;
}
```
Make the request:
```
function newDropoff(coords) {
  // Store the clicked point as a new GeoJSON feature with
  // two properties: `orderTime` and `key`
  var pt = turf.point(
    [coords.lng, coords.lat],
    {
      orderTime: Date.now(),
      key: Math.random()
    }
  );
  dropoffs.features.push(pt);
  pointHopper[pt.properties.key] = pt;

  // Make a request to the Optimization API
  $.ajax({
    method: 'GET',
    url: assembleQueryURL(),
  }).done(function(data) {
    // Create a GeoJSON feature collection
    var routeGeoJSON = turf.featureCollection([turf.feature(data.trips[0].geometry)]);

    // If there is no route provided, reset
    if (!data.trips[0]) {
      routeGeoJSON = nothing;
    } else {
      // Update the `route` source by getting the route source
      // and setting the data equal to routeGeoJSON
      map.getSource('route')
        .setData(routeGeoJSON);
    }

    if (data.waypoints.length === 12) {
      window.alert('Maximum number of points reached. Read more at mapbox.com/api-documentation/#optimization.');
    }
  });
}
```
### Add directionally to the routes:
```
map.addLayer({
  id: 'routearrows',
  type: 'symbol',
  source: 'route',
  layout: {
    'symbol-placement': 'line',
    'text-field': '▶',
    'text-size': {
      base: 1,
      stops: [[12, 24], [22, 60]]
    },
    'symbol-spacing': {
      base: 1,
      stops: [[12, 30], [22, 160]]
    },
    'text-keep-upright': false
  },
  paint: {
    'text-color': '#3887be',
    'text-halo-color': 'hsl(55, 11%, 96%)',
    'text-halo-width': 3
  }
}, 'waterway-label');
```

<https://kirkwilson.github.io/project-3-demo/index.html>

## Georeference Imagery

Go to:
<https://www.mapbox.com/studio/>

Add a tileset (must be a collection of raster or vector data).

Create a new style and add tileset.

Click:

Share & use
![alt text](https://github.com/kirkwilson/project-3-demo/blob/master/share-use.PNG)

```
var style1 = 'xxx';
var style2 = 'xxx';

// add the map
mapboxgl.accessToken = 'xxx';

var streetsMap = new mapboxgl.Map({
  container: 'streets',
  style: 'mapbox://styles/examples/' + style1,
  center: [-122.58862880207116,45.53387784078623],
  zoom: 12.6,
});

var historicMap = new mapboxgl.Map({
  container: 'historic',
  style: 'mapbox://styles/examples/' + style2,
});
```

```
var map = new mapboxgl.Compare(streetsMap, historicMap)
```

<https://kirkwilson.github.io/project-3-demo/index2.html>


