# Sound - sonification

* We used Two Tone to create sonification [link](https://app.twotone.io/)

* I wanted to use a COVID-19 dataset. First, I tried this dataset on Statistics Canada, for all cases in Canada. It has about 71K cases [link](https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1310078101)

* I downloaded only the occurence week for each case. So I got a CSV with each row representing a COVID-19 case, and the week it occured (week 0 is the first week of 2020, and onwards). Using a Pivot table, I generated number of cases by week.

* Put this through sonification. It was very short, because there weren't many weeks in the data. I could hear how the the cases peaked, and then declined before another spike at the end (which is interesting) but the weekly data doesn't really show all the ups and downs in the data. The situation is also vastly different in different parts of the country, so I don't know if it is accurate to look at the Canadian data as a whole.

* I downloaded the COVID-19 case dataset from the Ontario open data portal [link](https://data.ontario.ca/dataset/status-of-covid-19-cases-in-ontario)

* Again, every row represented one case (about 31K cases in this dataset). I used Pivot tables to get the number of cases by date.

* This was a more interesting sonification, we could see how the pandemic had a slow start in Ontario, perhaps giving public health authorities to prepare, then had a steady rise to the peak, and then an uneven 'flattening' of the work where cases slowly start coming down, with a few temporary spikes.


# Leaflet for tiled maps

* We are making a Leaflet map - this is Javascript code for a tiled map

* Created a new foldef "web-map" then opened the terminal from it.

* Start a new text file index.html, saved in the same folder.

* Defining the headers

```
<head>

	<title>A Demo WebMap</title>

	<meta charset="utf-8" />
	<meta name="viewport" content="width=device-width, initial-scale=1.0">

	<link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css" />
	<script src="https://unpkg.com/leaflet@1.6.0/dist/leaflet.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
    <script type="text/javascript"></script>
    </script>

</head>
```

* Save the leaflet.css and leaflet.js in the same directory.

* Then the base map: (the centre coordinates are for Ottawa)

```
<body>
<div id="mapid" style="width: 600px; height: 400px;"></div>
<script type="text/javascript">
var myMap = L.map('mapid').setView([45.4192857, -75.6973237],13);
L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner/{z}/{x}/{y}.png',
	{maxZoom: 19
}).addTo(myMap);
```

* We are using geojson format for the data on the map. This tool can be used to convert geographic data in a csv to geojson [link](http://www.convertcsv.com/csv-to-geojson.htm)

* Make a new file that will be saved as point-data.geojson . This geojson code contains 2 points:

```
{
   "type": "FeatureCollection",
   "features": [
  {
    "type": "Feature",
    "geometry": {
       "type": "Point",
       "coordinates":  [-75.6973237, 45.4192857]
    },
    "properties": {
    "Label":"Centre of our map!"
    }
  },
  {
    "type": "Feature",
    "geometry": {
       "type": "Point",
       "coordinates":  [ -75.699222, 45.4275 ]
    },
    "properties": {
    "Label":"Not the centre of the map "
    }
  }
]
}
```

* Then add this code to index.html pointing to the geojson file

```
// load a GeoJSON from external file
$.getJSON("point-data.geojson", function(data) {
  // add GeoJSON layer to the map once the file is loaded
  var datalayer = L.geoJson(data ,{
  onEachFeature: function(feature, featureLayer) {
  featureLayer.bindPopup(feature.properties.Label);
  }
}).addTo(myMap);
});

</script>
</body>
</html>
```

* In the terminal: `python -m http.server`

* The webserver will start. Take the http address and paste it into a web browser. I got http://0.0.0.0:8000/

* The map will load with two points

* Adding layers - Map Warper can be used to overlay historical map images onto tile maps. We used a map of Ottawa [link](https://mapwarper.net/maps/29435)

* In the index.html file, we add this code after .addTo(myMap); (where we specified the basemap)

```
L.tileLayer('http://mapwarper.net/maps/tile/29435/{z}/{x}/{y}.png',
  {maxZoom: 19}
).addTo(myMap);
```

* Refresh the browser page and the map image should overlay the map now, with the two points shown.

# Georectifying your own map images

* I grabbed the map in the exercise, an 1878 map of downtown Ottawa [link](https://collectionscanada.gc.ca/pam_archives/index.php?fuseaction=genitem.displayItem&rec_nbr=3824226&lang=eng)

* Went to upload to Mapwarper [link](http://mapwarper.net/maps/new)

* After filling the metadata and uploading the image, in the next page hit 'rectify.' Here there are two maps shown, the jpg and a current map that you can zoom to Ottawa. We have to add control points - 3 minimum, the more the better, so it can match up the two maps.

* After adding the control points, hit warp image. The map on the right will have the map image overlaid on top, hopefully correctly.

* In the Export tab, we will see the Tiles (Googe/OSM scheme) URL. I got http://mapwarper.net/maps/tile/48018/{z}/{x}/{y}.png

* Added that URL into index.html at the right place.

# Adding the layer toggle in my map

* Followed the instructions to add the layer options, so we can toggle between layers. Add these layers:

```
var toner = L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner/{z}/{x}/{y}.png'),
  watercolor = L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/watercolor/{z}/{x}/{y}.png'),
  historical1897 = L.tileLayer('https://mapwarper.net/maps/tile/29435/{z}/{x}/{y}.png'),
  lumberdistrict = L.tileLayer('http://mapwarper.net/maps/tile/48018/{z}/{x}/{y}.png')
```

* Then we add code to group the maps - base and Mapwarper overlays

```
var baseMaps = {
  "Toner": toner,
  "Watercolor": watercolor
};

var overlayMaps = {
  "1897": historical1897,
  "Lumber District": lumberdistrict
}
```

* Then we add the centre of the map and zoom
```
var myMap = L.map('mapid', {
  center: [45.4192857, -75.6973237],
  zoom: 13,
  layers:[toner]
});
```

* Finally the control box itself

```
L.control.layers(baseMaps, overlayMaps).addTo(myMap);
```

* Save as index2.html, then view as `http://localhost:8000/index2.html`

# Animation

* We use the Animated Marker plugin [link](https://github.com/openplans/Leaflet.AnimatedMarker)

* it animates points on the map

* This is the code we need to save as Leaflet.AnimatedMarker.js [link](Leaflet.AnimatedMarker.js)

* Make a new copy of the index file - index3.html

* Add in the <head> 
`<script src="Leaflet.AnimatedMarker.js"></script>`

* Add the points - we can add any lat/longs. Added this line after L.control.layers:

```
var line = L.polyline([[45.4245941,-75.6953823],[45.423299,-75.6927912],[45.4174419,-75.6907598]]),
    animatedMarker = L.animatedMarker(line.getLatLngs());
```

* Then add the animation as a layer
`myMap.addLayer(animatedMarker);`

* Open it in the browser at http://localhost:8000/index2.html
The point will move!

* The last two are Mapwarper maps

# Poster design in Inkscape

* Using Inkscape to edit a graphic in a PDF 

* Downloaded the sample graph - it's a simple bar graph, seems to be generated in R Studio. Import the PDF into Inkscape

* Object > Ungroup makes all the elements of the graph separate, so they can be edited separately

* I had to Ungroup twice - to get to the right level of selection

# Creating my own poster

* I tried to adapt some details of a paper I'm writing for another summer class, the History of Oil, into the poster template. The paper is about an ad series by British Petroleum that promoted its Persian oil operations in the early 20th century.