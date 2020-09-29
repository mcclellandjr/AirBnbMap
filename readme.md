<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Buncombe County Airbnbs (2020)</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.0/dist/leaflet.css"/>
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.14.0/css/all.css">
    <link href="https://fonts.googleapis.com/css?family=Titillium+Web" rel="stylesheet">
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; background: #fff; }
        .legend {
    line-height: 40px;
    font-family: 'Titillium Web', sans-serif;
    font-size: 16px;
    width: 190px;
    color: #333333;
    padding: 6px 8px;
    background: white;
    background: rgba(255,255,255,0.5);
    box-shadow: 0 0 15px rgba(0,0,0,0.2);
    border-radius: 5px;
}

.legend i {
    width: 20px;
    height: 20px;
    float: left;
    margin-right: 8px;
    opacity: 0.9;
}

.legend img {
    width: 16px;
    height: 16px;
    margin-right: 3px;
    float: left;
}

.legend p {
    font-size: 14px;
    line-height: 20px;
    margin: 0;
}

    </style>
    <script src="https://unpkg.com/leaflet@1.7.0/dist/leaflet.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet-ajax/2.1.0/leaflet.ajax.min.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chroma-js/1.3.4/chroma.min.js"></script>
</head>
<body>
<!-- Our web map and content will go here -->
<div id="map"></div>
<script>
    // 1. Create a map object.
      var mymap = L.map('map', {
      center: [35.5946,-82.5540], //note that we've centered the map to downtown AVL
      zoom: 11, //this line adjusts the starting zoom level of the map
      maxZoom: 18,//this line sets the maximum zoom level
      minZoom: 11,//this line sets the minimum zoom level
      detectRetina: true // detect whether the sceen is high resolution or not.
    });
    // 2. Add a base map.
    L.tileLayer('http://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png').addTo(mymap);
    // 3. Add Airbnb GeoJSON Data
    // Null variable that will hold Airbnb data
    var airbnb_listings = null;
    // 4. build up a set of colors from colorbrewer's dark2 category
    var colors = chroma.scale('Dark2').mode('lch').colors(3);

    // 5. dynamically append style classes to this page. The style classes will be used to shade the markers.
    // We can use a for loop to do this.
    for (i = 0; i < 3; i++) {
    $('head').append($("<style> .marker-color-" + (i + 1).toString() + " { color: " + colors[i] + "; font-size: 15px; text-shadow: 0 0 3px #ffffff;} </style>"));
}
    // Get GeoJSON and put it on the map when it loads
    // Make sure you have the correct directory path below
    // You can see we're also adding attribution information for our data sources
    airbnb_listings = L.geoJson.ajax("assets/airbnb_listings.geojson",{

pointToLayer: function(feature, latlng) {
    var id = 0;
    if (feature.properties.property_t == "Entire house") { id = 0; }
    else if (feature.properties.property_t == "Private room in house")  { id = 1; }
    else { id = 2;} // All other property types from attribute table
    return L.marker(latlng, {icon: L.divIcon({className: 'fab fa-airbnb marker-color-' + (id + 1).toString() })});
  },
    attribution: 'Airbnb Listings &copy; Inside Airbnb | Asheville Zoning Districts &copy; City of Asheville Open Data | Base Map &copy; CartoDB | Map Author: '
});
// Add the Airbnbs to the map.
airbnb_listings.addTo(mymap);
// create the zoning district layer
// 6. Set function for color ramp
colors = chroma.scale('Purples').colors(5); //we'll use 5 classes of purples

// this function manually defines your choropleth classification system
//so you'll need to figure out which break points you'd like to use
//based on the data distribution
//this equal interval classification with 5 classes, takes the range of the
//data (133) and divides it by 5, to show there are intervals of 27 per class
//so...
function setColor(density) {
    var id = 0;
    if (density > 106) { id = 4; } //133-27=106:highest fifth class
    else if (density > 79 && density <= 106) { id = 3; } //106-27=79:4thclass
    else if (density > 52 && density <= 79) { id = 2; } //79-27=52:3rdclass
    else if (density > 25 &&  density <= 52) { id = 1; } //52-27=25:2ndclass
    else  { id = 0; }
    return colors[id];
}
// 7. Set style function that sets fill color property equal to total Airbnbs
function style(feature) {
    return {
        fillColor: setColor(feature.properties.total_bnbs),
        fillOpacity: 0.4,
        weight: 2,
        opacity: 1,
        color: '#b4b4b4',
        dashArray: '4'
    };
}
// 8. Add county polygons
L.geoJson.ajax("assets/zoning_districts.geojson", {
    style: style
}).addTo(mymap);

// 9. Create Leaflet Control Object for Legend
var legend = L.control({position: 'topright'});

// 10. Function that runs when legend is added to map
legend.onAdd = function () {

    // Create Div Element and Populate it with HTML
    var div = L.DomUtil.create('div', 'legend');
    //this line creates a title for the choropleth part of the legend
    div.innerHTML += '<b>Airbnbs per District</b><br />';
    //notice the class breaks entered at the end of the next 5 lines
    //the colors specify the shade of purple that we used to do the polygon shading
    div.innerHTML += '<i style="background: ' + colors[4] + '; opacity: 0.5"></i><p>107+</p>';
    div.innerHTML += '<i style="background: ' + colors[3] + '; opacity: 0.5"></i><p>80-106</p>';
    div.innerHTML += '<i style="background: ' + colors[2] + '; opacity: 0.5"></i><p>53-79</p>';
    div.innerHTML += '<i style="background: ' + colors[1] + '; opacity: 0.5"></i><p>26-52</p>';
    div.innerHTML += '<i style="background: ' + colors[0] + '; opacity: 0.5"></i><p> 0-25</p>';
    //this line provides the legend title for the airbnb colored symbols
    div.innerHTML += '<hr><b>Property Type<b><br />';
    //the next 3 lines call the airbnb icon along with its proper color
    //notice the names of the Airbnb property types listed within the <p> tags at the end of the lines
    div.innerHTML += '<i class="fab fa-airbnb marker-color-1"></i><p>Entire house</p>';
    div.innerHTML += '<i class="fab fa-airbnb marker-color-2"></i><p>Private room in house</p>';
    div.innerHTML += '<i class="fab fa-airbnb marker-color-3"></i><p>Other</p>';
    // Return the Legend div containing the HTML content
    return div;
};

// 11. Add a legend to map
legend.addTo(mymap);
// 12. Add a scale bar to map
L.control.scale({position: 'bottomleft'}).addTo(mymap);
</script>
</body>
</html>
