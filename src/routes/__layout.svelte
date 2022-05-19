<script>
  import "../app.css";
  import { geojson_store, bikeshare_transit_store, user_store, points_prompt_store, points_store, prompts_store, current_prompt_store, map_center_store, bikeshare_store, transit_store } from '$lib/stores';
  import { mapboxToken } from '$lib/conf.js'
  import { Map, Geocoder, Marker, controls } from '$lib/components.js'
  import Content from './Content.svelte';
import {afterUpdate, getContext, onMount, setContext} from 'svelte';
import { get } from 'svelte/store';
  import { variables } from '$lib/variables';
  // import turf from '@turf/helpers';
  import turfBuffer from '@turf/buffer';
  import turfBbox from '@turf/bbox';
  import turfBboxPolygon from '@turf/bbox-polygon';
  import osmtogeojson from 'osmtogeojson'
import bbox from "@turf/bbox";
import { feature } from "@turf/helpers";
import { contextKey } from '$lib/components.js'


  // let prompt = "Imagine a place in your community.  A place that hasn't changed in a very long time.  You're going there to take a picture.  Once you arrive, you drop your camera — you reach down for it, and when you stand back up, you are 50 years in the future.  What do you see?  How has this place changed?"

  const { GeolocateControl, NavigationControl } = controls


  let checked = true;

  let loading = false;

  let unique = {};

  $: center = $map_center_store;

  let marker;
  let zoom = 12;
  let mapComponent;

  let create_mode = true;

  let selected_location;

  let mode = "starting";


    // On page load
    onMount(async() => {

  // We geolocate the user's IP address, to initialize the map settings near where they are and center the map on a familiar place.
    ipToCoordinates()
    // .then(async() => {
    //   updatePointsForNewPrompt()
    //    })
        // .then(async() => {
        //   fetchAlternativesGeometery();
        // })

      // console.log($geojson_store);
            // fetchPrompts()
            // .then(async() => {
            //   updatePointsForNewPrompt();
            // })
    })

   // We take the user's IP, get coordinates from it (an approximate location — usually the data center nearest them), and update the map location to those coordinates.
   async function ipToCoordinates() {
    
    const ip = await fetch("https://serene-journey-42564.herokuapp.com/https://api.ipify.org?format=json&callback=getIP");
    
    const ip_json = await ip.json();
    console.log(ip_json);
    
    const request = await fetch(`https://serene-journey-42564.herokuapp.com/ipinfo.io/${ip_json["ip"]}/geo?token=${variables.ipInfo}`, {
        method: 'GET',
        "Content-Type": "application/json",
        "charset": "utf-8",
        "Access-Control-Allow-Headers": "X-Requested-With",
        "X-Requested-With": "XMLHttpRequest"   
    });

    const json = await request.json()
    
    console.log(json);
    
    let coordinates = json.loc.split(',');
    console.log(coordinates);
    coordinates = {"lat": coordinates[0], "lng": coordinates[1]};
    // coordinates = {"lat": 38.886503, "lng": -77.1842802};
    center = coordinates;
    $map_center_store = center;

    unique = {};
    }

function selectLocation( { detail }) {

// If create_mode is false, that means the user is just browsing the map and is not adding content.  In that case, we don't want to add a marker or update the center or anything when they click.
// So we'll only run this function if create_mode is anything but false.  This likewise means, that if a user has selected a location already in the add content workflow and are entering the content, they are still able to click around on the map and update the location.  This seems like a user-friendly behavior from testing.
if (mode == "selecting" || mode == "starting") {

  // Sometimes a click registers twice in svelte — meaning, the first click will give us (detail), and the second click will give us undefined.
  // If we tried doing the following when the click was undefined, it wouldn't work
  // So we specify to only get the location of the user's click and update the marker on the map when we actually have that information from the click (e.g. the first click, not the second undefined one that registers)
  if (detail.lat) {
  console.log(detail);
  console.log(detail.lat);
  selected_location = detail;
  console.log(selected_location);
  
  // If the map is somewhat zoomed out when the user clicks, we will zoom in on their click.
  // This is a bit jarring once you're already zoomed in close on the map, so we only have the behavior when the current zoom level is less than 16.
  // if (zoom < 16) {
  //   mapComponent.setCenter({lng: selected_location.lng, lat: selected_location.lat});
  //   zoom = 16;
  // }

  // Finally, if create_mode is not yet in the "input_content" stage yet (when the user has the content text area form available), we will update it to that stage.
  create_mode != "input_content" ? create_mode = "input_content" : null;

  mode = "selecting";
  }
}
}

function navigate (next) {
    page = next
  }

  function placeChanged (e) {
    const { result } = e.detail
    mapComponent.setCenter(result.center, 14)
  }
  
  function randomLng () {
    return 77 + (Math.random() - 0.5) * 30
  }

  function randomLat () {
    return 13 + (Math.random() - 0.5) * 30
  }

  function flyToRandomPlace () {
    mapComponent.flyTo({
      center: [
        randomLng(),
        randomLat()
      ],
      essential: true
    })
  }

  function recentre ({ detail }) {
    center = detail.center
  }

  function drag ({ detail }) {
    marker = detail.center
  }

  async function showNearbyBikesAndTransit() {


    loading = true;
    mode = "loading";

    console.log('test');

    console.log(selected_location);


    var point =     {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Point",
        "coordinates": [
          parseFloat(selected_location.lng),
          parseFloat(selected_location.lat)
        ]
      }
    }
    // point.geomType = "point";
    // console.log(point);
    // var buffer = turfBuffer(point, 0.25, {units:'miles'});
    // var bbox = turfBbox(buffer);

    // let geometry_xml;

    // let bboxReordered = [bbox[1], bbox[0], bbox[3], bbox[2]];

    // let bboxString = bboxReordered.toString();
    // bboxString = bboxString.replace(/,/g, '%2C');

    // let bikeshareBboxString = bboxString;
    // let transitBboxString = bboxString;


    var bikeshareBuffer = turfBuffer(point, 0.5, {units:'miles'});
    var transitBuffer = turfBuffer(point, 0.5, {units:'miles'});


    // var buffered = turfBuffer(point, 20, {units:'miles'});
    var bikeshareBbox = turfBbox(bikeshareBuffer);
    var transitBbox = turfBbox(transitBuffer);

    // let bbox_polygon = turfBboxPolygon(bbox_raw);
    // console.log(turfBboxPolygon(bbox_raw));

let geometry_xml;

// console.log(bbox_raw);

let bikeshareBboxReordered = [bikeshareBbox[1], bikeshareBbox[0], bikeshareBbox[3], bikeshareBbox[2]];

let transitBboxReordered = [transitBbox[1], transitBbox[0], transitBbox[3], transitBbox[2]];

let bikeshareBboxString = bikeshareBboxReordered.toString();
bikeshareBboxString = bikeshareBboxString.replace(/,/g, '%2C');

let transitBboxString = transitBboxReordered.toString();
transitBboxString = transitBboxString.replace(/,/g, '%2C');

let overpass_bikeshare_transit_query = `%2F*%0AThis+query+looks+for+nodes%2C+ways+and+relations+%0Awith+the+given+key%2Fvalue+combination.%0AChoose+your+region+and+hit+the+Run+button+above!%0A*%2F%0A%5Bout%3Ajson%5D%5Btimeout%3A25%5D%3B%0A%2F%2F+gather+results%0A(%0A++%2F%2F+query+part+for%3A+%E2%80%9Camenity%3Dbicycle_rental%E2%80%9D%0A++relation%5B%22route%22%3D%22bicycle%22%5D(${bikeshareBboxString})%3B%0A++node%5B%22amenity%22%3D%22bicycle_rental%22%5D(${bikeshareBboxString})%3B%0A++way%5B%22amenity%22%3D%22bicycle_rental%22%5D(${bikeshareBboxString})%3B%0A++relation%5B%22amenity%22%3D%22bicycle_rental%22%5D(${bikeshareBboxString})%3B%0A++node%5B%22shop%22%3D%22bicycle%22%5D(${bikeshareBboxString})%3B%0A++way%5B%22shop%22%3D%22bicycle%22%5D(${bikeshareBboxString})%3B%0A++relation%5B%22shop%22%3D%22bicycle%22%5D(${bikeshareBboxString})%3B%0A++node%5B%22public_transport%22%5D(${transitBboxString})%3B%0A++way%5B%22public_transport%22%5D(${transitBboxString})%3B%0A++relation%5B%22public_transport%22%5D(${transitBboxString})%3B%0A++node%5B%22route%22%3D%22tram%22%5D(${transitBboxString})%3B%0A++way%5B%22route%22%3D%22tram%22%5D(${transitBboxString})%3B%0A++relation%5B%22route%22%3D%22tram%22%5D(${transitBboxString})%3B%0A++node%5B%22route%22%3D%22bus%22%5D(${transitBboxString})%3B%0A++way%5B%22route%22%3D%22bus%22%5D(${transitBboxString})%3B%0A++relation%5B%22route%22%3D%22bus%22%5D(${transitBboxString})%3B%0A++node%5B%22route%22%3D%22train%22%5D(${transitBboxString})%3B%0A++way%5B%22route%22%3D%22train%22%5D(${transitBboxString})%3B%0A++relation%5B%22route%22%3D%22train%22%5D(${transitBboxString})%3B%0A++node%5B%22route%22%3D%22subway%22%5D(${transitBboxString})%3B%0A++way%5B%22route%22%3D%22subway%22%5D(${transitBboxString})%3B%0A++relation%5B%22route%22%3D%22subway%22%5D(${transitBboxString})%3B%0A++node%5B%22route%22%3D%22monorail%22%5D(${transitBboxString})%3B%0A++way%5B%22route%22%3D%22monorail%22%5D(${transitBboxString})%3B%0A++relation%5B%22route%22%3D%22monorail%22%5D(${transitBboxString})%3B%0A++node%5B%22route%22%3D%22trolleybus%22%5D(${transitBboxString})%3B%0A++way%5B%22route%22%3D%22trolleybus%22%5D(${transitBboxString})%3B%0A++relation%5B%22route%22%3D%22trolleybus%22%5D(${transitBboxString})%3B%0A)%3B%0A%2F%2F+print+results%0Aout+body%3B%0A%3E%3B%0Aout+skel+qt%3B`;

// console.log(overpass_turbo_query);

// let data_key_value = `data=${overpass_turbo_query}`;

// let data_key_value = `data=${overpass_bikeshare_query}`;

const fetch_bikeshare_transit = await fetch(`https://serene-journey-42564.herokuapp.com/https://overpass.kumi.systems/api/interpreter?data=${overpass_bikeshare_transit_query}`, {

  method: "GET",
  
  // Adding headers to the request
  headers: {
    "User-agent": "development project; sam@ysli.be",
    "X-Requested-With": "XmlHttpRequest",
    "Access-Control-Allow-Methods": "POST",
    "Access-Control-Allow-Origin": "*",
  }
  })

const bikeshare_transit = await fetch_bikeshare_transit.json();

// console.log(json);


let bikeshare_transit_geojson = osmtogeojson(bikeshare_transit);

$geojson_store = bikeshare_transit_geojson;

// console.log(bikeshare_transit_geojson);

// $bikeshare_transit_store = bikeshare_transit_geojson;

// $bikeshare_store = bikeshare_transit_geojson.features.filter(feature => feature.properties?.amenity == "bicycle_rental" || feature.properties?.shop == "bicycle" || feature.properties?.route == "bicycle");


// $transit_store = bikeshare_transit_geojson.features.filter(feature => feature.properties?.amenity != "bicycle_rental" && feature.properties?.shop != "bicycle" && feature.properties?.route != "bicycle");

// console.log(get(bikeshare_store));
// console.log(get(transit_store));

unique = {};

loading = false;


mode = "browsing";

console.log(mode);

  }

  function toggleMapLayer(e) {

    const { getMap, getMapbox } = getContext(contextKey)
  const map = getMap()

    console.log(e.target.name);
    
    console.log(e);

    console.log(e.target.checked);

    e.target.checked == false ? map.setLayoutProperty(`${e.target.name}`, 'visibility', 'none') : map.setLayoutProperty(`${e.target.name}`, 'visibility', 'visible');

    // let element = e;

    // console.log(element.target.checked);

    // console.log(e.target.checked);

    // element.target.checked ? e.target.checked = false : e.target.checked = true;

    // console.log(e.target.checked);

    // return;

    // setTimeout(() => {

    //   console.log(e);

    // console.log(e.target.name);

    // console.log(e.srcElement.checked);

    // e.target.checked = "false";

    // console.log(e.srcElement.checked);

    // // checked = false;

    // }, 50)

    // e.target.checked ? checked = false : null;

    // e.target.name
  }

</script>

<svelte:head>

  <title>Find Bikes and Transit Near You</title>

  
  <meta name="description" content="Find Bikes and Transit Near You">
  
  <!-- Facebook Meta Tags -->
  <meta property="og:url" content="https://freego.netlify.app">
  <meta property="og:type" content="website">
  <meta property="og:title" content="Find Bikes and Transit Near You">
  <meta property="og:description" content="And start living car free!">
  <meta property="og:image" content="https://images.squarespace-cdn.com/content/v1/58a1c69f5016e176238a2ae8/1605195330050-OVDXMMS41AZ5J1NJH9KS/GSD-G2-clubhousefort.jpg">
  
  <!-- Twitter Meta Tags -->
  <meta name="twitter:card" content="summary_large_image">
  <meta property="twitter:domain" content="freego.netlify.app">
  <meta property="twitter:url" content="https://freego.netlify.app">
  <meta name="twitter:title" content="Find Bikes and Transit Near You">
  <meta name="twitter:description" content="And start living car free!">
  <meta name="twitter:image" content="https://images.squarespace-cdn.com/content/v1/58a1c69f5016e176238a2ae8/1605195330050-OVDXMMS41AZ5J1NJH9KS/GSD-G2-clubhousefort.jpg">
</svelte:head>

<!-- This script runs first, as soon as the page is loaded, before anything in the script tag above -->
<!-- The very first thing we do is load data from the backend, so we can populate the map with it -->
<!-- <script context="module">

  let original = true;

  // ipToCoordinates();

  export const load = async ({ fetch, url }) => {

       // We take the user's IP, get coordinates from it (an approximate location — usually the data center nearest them), and update the map location to those coordinates.
    
    const ip = await fetch("https://serene-journey-42564.herokuapp.com/https://api.ipify.org?format=json&callback=getIP");
    
    const ip_json = await ip.json();
    console.log(ip_json);
    
    const request = await fetch(`https://serene-journey-42564.herokuapp.com/ipinfo.io/${ip_json["ip"]}/geo?token=${variables.ipInfo}`, {
        method: 'GET',
        "Content-Type": "application/json",
        "charset": "utf-8",
        "Access-Control-Allow-Headers": "X-Requested-With",
        "X-Requested-With": "XMLHttpRequest"   
    });

    if (request.ok) {

    const json = await request.json()
    
    console.log(json);
    
    let coordinates = json.loc.split(',');
    coordinates = {"lat": 38.886503, "lng": -77.1842802};
    center = coordinates;
    map_center_store.set(center);

			return {
				props: { 
          center: center
        }
			};
		}

    else {
      console.log(request);
    }

		const { message } = await response.json();

		return {
			error: new Error(message)
		};

  }
  
</script> -->

{#if center?.lng}
<div class="section-txt" id="map">
<div class="map-wrap">
  <Map
    bind:this={mapComponent}
    accessToken={mapboxToken}
    on:recentre={recentre}
    on:drag={drag}
    on:click={selectLocation}
    {center}
    bind:zoom
  >
  {#if selected_location}
  <Marker lat={selected_location.lat} lng={selected_location.lng}></Marker>
  {/if}
  {#key unique}
  <slot></slot>
  {/key}

  {#if mode == "selecting"}
  <button class="mode_button" style="position: absolute; bottom: 42%; margin: auto; left: 0; display: block; left: 50%;
  -webkit-transform: translateX(-50%);
  -moz-transform: translateX(-50%);
  transform: translateX(-50%);" on:click|preventDefault={showNearbyBikesAndTransit} type="button">
  Show Nearby Bikes And Transit
</button>
{:else if mode == "loading"}
<span class="mode_button" style="background: white; color: black; border: solid 2px black; padding: 5px; position: absolute; bottom: 42%; margin: auto; left: 0; display: block; left: 50%;
-webkit-transform: translateX(-50%);
-moz-transform: translateX(-50%);
transform: translateX(-50%);">
Loading results ...
</span>
{:else if mode == "starting"}
<span class="mode_button" style="background: white; color: black; border: solid 2px black; padding: 5px; position: absolute; bottom: 42%; margin: auto; left: 0; display: block; left: 50%;
-webkit-transform: translateX(-50%);
-moz-transform: translateX(-50%);
transform: translateX(-50%);">
Click a point to see nearby bikesharing and transit.
</span>
{:else if mode == "browsing"}
<button class="mode_button" style="position: absolute; bottom: 42%; margin: auto; left: 0; display: block; left: 50%;
-webkit-transform: translateX(-50%);
-moz-transform: translateX(-50%);
transform: translateX(-50%);" on:click|preventDefault={function() {mode = "selecting"}} type="button">
Select a new location
</button>
{/if}

  </Map>
</div>
</div>
{/if}

<style>
  
  :global(#logo svg) {
    fill: white;
    height: 60px;
  }

  .slogan {
    margin-top: 14px;
  }

  .map-wrap {
    width: 100%;
    height: 100vh;
    /* height: 300px; */
  }

  .action-buttons {
    display: flex;
    justify-content: space-between;
  }

  #fly-to,
  #change-zoom {
    display: block;
    position: relative;
    margin: 0px auto;
    height: 40px;
    padding: 10px;
    border: none;
    border-radius: 3px;
    font-size: 12px;
    text-align: center;
    color: #fff;
    background: #ee8a65;
  }

  /* For responsive formatting, this covers screens wider than mobile */
  @media only screen and (min-width: 601px) {
  #content_textarea {
    height: 100px !important;
  }

  #prompt {
    max-width: 600px;
    line-height: 22px;
    /* text-align: left !important; */
  }

  .mode_button {
    bottom: 25% !important;
    font-size: 20px;
  }
}

@media only screen and (max-width: 600px) {

  .mode_button {
    bottom: 10% !important;
    font-size: 20px;
  }
}

</style>