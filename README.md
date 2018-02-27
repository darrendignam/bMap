# bMap - Documentation

## Tips, tricks, and information about bMap
This plugin is based on the bMap object. This object is a wrapper for the Google Maps API, and is dependent on jQuery and Google Maps. Having experience with either will help you understand bMap, but is not essential - it may also help if you know some javascript.

## Getting started
The first thing you need to do is grab a copy of bMap from the downloads link on the left (please respect my bandwidth and host your own copy), and upload it to your server. Then on the page where you want a map you need add the script tags that point to jQuery, Google Maps, and bMap there are many ways of doing this, but I like to use the google cached versions where possible. You will also need to obtain a google API key.

```
<script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.3.2/jquery.min.js"></script>
<script src="http://maps.google.com/maps?file=api&v=2&key=YOUR_GOOGLE_KEY" type="text/javascript"></script>
<script type="text/javascript" src="jQuery.bMap.1.2.1.js"></script>
```
You will then need to create a div on the page and give it an ID (that will be used later to render the map). Then create a script block in the page and write your javascript. Take a look at the examples, the HTML shown can be used as a starting point for most basic setups.

To use bMap, you need to initilise an instance of the object. There are two ways of doing this:

```javascript
var map_object = new bMap({ mapCanvas: "map" });
//or the jQuery way
$('#map').bMap();
In these instances, the element with an ID of map will have a map rendered with the bMap defaults. These are the defaults:
```

## bMap defaults
```javascript
defaults = {  
		mapCenter:  [51,0],            //Two elemet array [lat,lng]
		mapZoom:     1,                //initial zoom, 1 is fully zoomed out
		mapCanvas:   "map",            //div to render the map in
		mapSidebar:  "none",           //div to render the marker sidebar
		mapLayerbar: "none",           //div to render the layer sidebar
		mapType:     G_NORMAL_MAP,     //initial map type (uses Google map API constants)
		loadMsg: "<h2>Loading...</h2>" //content of the AJAX loading message div
}; 
```
In addition to these defaults you can also pass the constructor a set or markers to render on the map - you can assign an array of icons (more about that later).

## bMap layers
bMap makes it easy to have markers, lines and polygons on your map. These are organised as layers. To create a layer we use JSON notation to describe the layer we want. All layers share the following basic construction:

```javascript
//Common layer properties:
{
	"name":   "Layer Name", //Layer name, that will be seen in the layerBar - Defaults to the layers index in the internal layer array.
	"type":   "marker",     //This will be set by bMap when you create a layer, so its not necessary to specifically set this. (marker line or polygon)
	"visible":"true",       //This defaults to true, but if you want a layer to load hidden; set this to false
	"data":   []            //This is an array of JSON objects, with latitude and logitude values
}

//Lines and Polygons also accept the following optional properties:
//Line layer properties:
{
	"weight":  50,       //Line thickness
	"color":  "#00F",    //HTML colour for the line, defaults to blue
	"opacity": 1         //Opacity value between 0 and 1
}

//Polygon layer properties:
{
	"weight":  50,       //Line thickness of perimiter
	"color":  "#00F",    //HTML colour for the polygon, defaults to blue
	"opacity": 0.5       //Opacity value between 0 and 1. The perimiter is solid, this is the opacity of the inside fill colour
}
```

## bMap data array
All layers require the data property. This array contains all the latitude and longitude values. Marker layer data arrays can also contain extra information for the info-window/sidebar and optionally icon.

```javascript
//Polygon and Polyline data object:
{
	"lat": 53.432817,   //Latitude
	"lng": -10.075986   //Longitude
}

//Marker data object:
{
	"lat":  53.432817,                         //Latitude
	"lng":  -10.075986,                        //Longitude
	"title":"Connemara Golf Links",            //Appears in sidebar, and in info-window
	"body": "Aillebrack, Co. Galway, Ireland", //Appears in info-window
	"icon": "1"                                //Use this to reference an icon in the array of icons passed to the constructor
}
```
These data objects are combined into an array, using standard JSON techniques. There are three bMap functions that you pass layer objects too, and the information is drawn onto the map.

## bMap layer constructor functions
There are three finctions for adding layers. One for the markers, lines and polygons. You pass in a layer object, including the data array, and the information is rendered on the map.

```javascript
//layer constructor functions
insertMarkers();
insertLine();
insertPolygon();

//example usage - bMap object:
var map_object = new bMap({ mapCanvas: "map" });
map_object.insertLine({
	"color":"#F0F",
	"data":[
		{
			"lat":51.49757618329838,
			"lng":-0.1746654510498047
		},{
			"lat":51.47769451182406,
			"lng":-0.0009441375732421875
		}	
	]
});

//example usage - jQuery object:
$('#map').bMap();
$('#map').data('bMap').insertLine({
	"color":"#F0F",
	"data":[
		{
			"lat":51.49757618329838,
			"lng":-0.1746654510498047
		},{
			"lat":51.47769451182406,
			"lng":-0.0009441375732421875
		}	
	]
});
```

The bMap constructor accepts a marker object as a parameter, and this is processed with the insertMarker() function. The three AJAX functions also use the layer functions internally to process the JSON data they recieve.

## bMap AJAX functions
For a larger project with lots of data, or when data is stored in a database - it may be better to use the AJAX functions. There are three functions, one for each type of layer, and they use the same object structures. The functions, and their defaults are shown below:

```javascript
AJAXMarkers({
	serviceURL: "mapService.php", //Address of the page to request data
	action: "getMarkers",         //POST var action, with value getMarkers
	vars: []                      //An optional array of aditional values to POST to the target page
});

AJAXLine({
	serviceURL: "mapService.php",
	action: "getLine",
	vars: []
});

AJAXPolygon({
	serviceURL: "mapService.php",
	action: "getPolygon",
	vars: []
});
```
Because you can send your own POST variables to the service page, it should be possible to manage any kind of data. Your target page needs to respond with JSON that would be acceptable to the layer functions above. To help here are some ASP and PHP loops that can generate JSON for the AJAXMarkers function:

```visualbasic
'ASP loop to generate JSON for bMap
'This example assumes you sanatise the POST vars, and make a database connection...
serviceAction = Request("action")
if serviceAction = "getMarkers" then
	mySQL = "SELECT * FROM markerTable"
	Set objRS = oConn.Execute(mySQL)

	JSONreply = "{ ""name"":""Markers"", ""type"":""marker"", ""data"" : [ "
	if NOT objRS.EOF then
		do while NOT objRS.EOF
			JSONreply = JSONreply + "{""lat"":"&objRS;("Lat")&",""lng"":"&objRS;("Lng")&", ""title"":"""&objRS;("title")&""", ""icon"":"""&objRS;("icon")&""", ""body"":"""&objRS;("body")&"""},"
			objRS.movenext
		loop
		'remove trailing comma "," that got added to the last entry above
		JSONreply = Mid(JSONreply,1,Len(JSONreply)-1)
	End IF
	'finish off string, and send to the client
	JSONreply = JSONreply + " ] }"
	response.write(JSONreply)
		
	'close the ADO objects
	objRS.Close
	Set objRS = Nothing
End If
```

```php
//PHP loop to generate JSON for bMap
//Again: This example assumes you sanatise the POST vars, and make a database connection...
$SQLquery = "SELECT * FROM markerTable";
$result = mysql_query($SQLquery);
$counter = 0;
//Send layer properties
echo "{ ""name"":""Markers"", ""type"":""marker"", ""data"" : [ ";
//Loop markers, sending to client
while($row = mysql_fetch_array($result)){
	if ($counter != 0) { echo "," ; }   //Dealing with commas
	echo '{"lat":"' . $row['lat'] . '","lng":"' . $row['lng']. '","title":"' . $row['title']. '","icon":"' . $row['icon']. '","body":"' . $row['body']. '"}';
	$counter++;
}
echo " ] }";   //finishing up, and sending to client
```
These are just examples, in your website you will want to protect your database from SQL injection - but that is outside the scope of this document. Please use these as a starting point. You might want to send the AJAX service page values that you can then use in your SELECT statement for instance. And you will have to open and close your own database connections.

## bMap geocoder
There is a wrapper function for the google geocoder - centerAtAddress - so you can center the map at an address. You can also use the map objects setCenter function to change the map center.
```javascript
$('#map').bMap();
$('#map').data('bMap').centerAtAddress('London, UK');
$('#map').data('bMap').map.setCenter( new GLatLng(51.31714872961282,-1.0788917541503906);
```

## This readme was generated from the old internet archive of the old project website:
Plenty of actual examples and user discussion there:
https://web.archive.org/web/20131213055413/http://www.blocsoft.com:80/bmap/docs.asp
