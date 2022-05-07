<script>
  import { getContext } from 'svelte'
  import { contextKey } from '$lib/components.js'
  import MiniScroller from './MiniScroller.svelte'
  import { bikeshare_transit_store, points_prompt_store, bikeshare_store, transit_store, geojson_store } from "$lib/stores.js"
  import { get } from 'svelte/store';

  const { getMap, getMapbox } = getContext(contextKey)
  const map = getMap()
  const mapbox = getMapbox()

  let load = false;

  // Add a new source from our GeoJSON data and set the
  // 'cluster' option to true. GL-JS will add the point_count property to your source data.

  // We check to see if the mapsource has already been drawn, in which case we just want to update it via the map.getSource('content').setData() function
  // This conditional is filled once the page has already loaded, and we destroy the <Content> component to update the data, and then refresh it
  // When it refreshes, map.getSource('content') returns true and this conditional continues

  if ($geojson_store) {

    let geojson_array = $geojson_store.features;

    if (map.getLayer('transit-line')) {

console.log('removing;')
map.removeLayer('transit-line'); 
map.removeLayer('transit-point'); 
map.removeSource('transit-points');
map.removeSource('transit-lines');
}

    if (map.getLayer('bicycle-point')) {

      console.log('removing');
      map.removeLayer('bicycle-point'); 
      map.removeLayer('bicycle-line'); 
      map.removeSource('bicycle-points');
      map.removeSource('bicycle-lines');
    }

    let transit_points_data = {
        "type": "FeatureCollection",
        "features": geojson_array.filter(feature => feature.properties?.type != "route" && feature.properties?.amenity != "bicycle_rental" && feature.properties?.shop != "bicycle")
    }

    let transit_lines_data = {
        "type": "FeatureCollection",
        "features": geojson_array.filter(feature => feature.properties?.type == "route" && feature.properties?.route != "bicycle")
    }

    let bike_points_data = {
        "type": "FeatureCollection",
        "features": geojson_array.filter(feature => feature.properties?.type != "route" && (feature.properties?.amenity == "bicycle_rental" || feature.properties?.shop == "bicycle") )
    }

    let bike_lines_data = {
        "type": "FeatureCollection",
        "features": geojson_array.filter(feature => feature.properties?.type == "route" && feature.properties?.route == "bicycle")
    }

    map.addSource('transit-points', {
    type: 'geojson',
    data: transit_points_data
  })

  map.addSource('transit-lines', {
    type: 'geojson',
    data: transit_lines_data
  })

  map.addSource('bicycle-points', {
    type: 'geojson',
    data: bike_points_data
  })

  map.addSource('bicycle-lines', {
    type: 'geojson',
    data: bike_lines_data
  })

map.addLayer({
    'id': 'transit-line',
    'generateid': true,
    'type': 'line',
    'source': 'transit-lines',
    'layout': {
    'line-join': 'round',
    'line-cap': 'round'
    },
    'paint': {
    'line-color': [
      'case',
      ['boolean', ['feature-state', 'hover'], false],
      'yellow',
      'indigo'
    ],
    'line-width': 8
    }
    });

  map.addLayer({
'id': 'transit-point',
'type': 'circle',
'source': 'transit-points',
    'paint': {
      'circle-color': '#0096FF',
      'circle-radius': 6,
      'circle-stroke-width': 1.5,
      'circle-stroke-color': 'black'
  },
});

map.addLayer({
    'id': 'bicycle-line',
    'generateid': true,
    'type': 'line',
    'source': 'bicycle-lines',
    'layout': {
    'line-join': 'round',
    'line-cap': 'round'
    },
    'paint': {
    'line-color': [
      'case',
      ['boolean', ['feature-state', 'hover'], false],
      'yellow',
      '#DE3163'
    ],
    'line-gap-width': 1.2,
    'line-width': 5
    }
    });

    map.addLayer({
  'id': 'bicycle-point',
  'type': 'circle',
  'source': 'bicycle-points',
      'paint': {
        'circle-color': 'red',
        'circle-radius': 6,
        'circle-stroke-width': 1.5,
        'circle-stroke-color': 'WHITE'
    },
  });

}
else {
console.log('none yet')
}


  map.on('mouseenter', 'bicycle-point', function () {
    map.getCanvas().style.cursor = 'pointer'
  })

  map.on('mouseleave', 'bicycle-point', function () {
    map.getCanvas().style.cursor = ''
  })

  map.on('mouseenter', 'bicycle-line', function () {
    map.getCanvas().style.cursor = 'pointer'
  })

  map.on('mouseleave', 'bicycle-line', function () {
    map.getCanvas().style.cursor = ''
  })

  map.on('mouseenter', 'transit-line', function () {
    map.getCanvas().style.cursor = 'pointer'
  })

  map.on('mouseleave', 'transit-line', function () {
    map.getCanvas().style.cursor = ''
  })

  map.on('mouseenter', 'transit-point', function () {
    map.getCanvas().style.cursor = 'pointer'
  })

  map.on('mouseleave', 'transit-point', function () {
    map.getCanvas().style.cursor = ''
  })

  map.on('click', 'bicycle-point', function (e) {

console.log(e);
const coordinates = e.lngLat;
// const coordinates = e.features[0].geometry.coordinates.slice();

// Ensure that if the map is zoomed out such that multiple
// copies of the feature are visible, the popup appears
// over the copy being pointed to.
// while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
//   coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360
// }

// should be centered, but need to figure out the map dimensions first
console.log(coordinates);
let updated_center;
screen.width < 600 ? updated_center = [ coordinates[0], coordinates[1] - 0.025 ] : updated_center = [ coordinates[0], coordinates[1] - 0.015 ];
// let updated_center = [ coordinates[0], coordinates[1] - 0.015 ]
// console.log(updated_center);
// map.setCenter(updated_center)

// There is a bug that occurs, when you have clicked on one point and are viewing the popup.  If you click on another point, while the first popup is visible, then the popup for the other point will incorrectly display.
// So this conditional checks to see if a popup already exists on the page, and removes it if it does, to bypass that bug.  Not sure about the behavior causing the root issue — this was simpler for now.
// if (document.getElementById('mini-scroller')) {
//   console.log('present');
//   document.getElementById('mini-scroller').remove();
// }

addPopup(e, coordinates);

// // The miniscroller component is what appears in the popup, and we instantiate it with props from the selected point (most importantly, content) so we can show the content from that point in the popup.
// new MiniScroller({ 
//   target: document.getElementById('mini-scroller'), 
//   props: { layer:e.features[0].layer.id, properties:e.features[0]._vectorTileFeature.properties, id:e.features[0]._vectorTileFeature.properties.id, shop:e.features[0]._vectorTileFeature.properties?.shop, network:e.features[0]._vectorTileFeature.properties.network, name:e.features[0]._vectorTileFeature.properties.name } 
//  }) // eslint-disable-line no-new
})


  map.on('click', 'transit-point', function (e) {
    console.log(e);

    const coordinates = e.lngLat;

    // const coordinates = e.features[0].geometry.coordinates.slice()
  
    // // Ensure that if the map is zoomed out such that multiple
    // // copies of the feature are visible, the popup appears
    // // over the copy being pointed to.
    // while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
    //   coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360
    // }

    console.log(e.features[0]);
  
    // should be centered, but need to figure out the map dimensions first
    console.log(coordinates);

    let updated_center;
    screen.width < 600 ? updated_center = [ coordinates[0], coordinates[1] - 0.025 ] : updated_center = [ coordinates[0], coordinates[1] - 0.015 ];
    // let updated_center = [ coordinates[0], coordinates[1] - 0.015 ]
    // console.log(updated_center);
    // map.setCenter(updated_center)

    addPopup(e, coordinates);
  })

  map.on('click', 'transit-line', function (e) {
    console.log(e);

    const coordinates = e.lngLat;

    // const coordinates = e.features[0].geometry.coordinates.slice()
  
    // // Ensure that if the map is zoomed out such that multiple
    // // copies of the feature are visible, the popup appears
    // // over the copy being pointed to.
    // while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
    //   coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360
    // }

    console.log(e.features[0]);
  
    // should be centered, but need to figure out the map dimensions first
    console.log(coordinates);

    let updated_center;
    screen.width < 600 ? updated_center = [ coordinates[0], coordinates[1] - 0.025 ] : updated_center = [ coordinates[0], coordinates[1] - 0.015 ];
    // let updated_center = [ coordinates[0], coordinates[1] - 0.015 ]
    // console.log(updated_center);
    // map.setCenter(updated_center)

    console.log('test');

    addPopup(e, coordinates);

  })

  map.on('click', 'bicycle-line', function (e) {
    console.log(e);

    const coordinates = e.lngLat;

    // const coordinates = e.features[0].geometry.coordinates.slice()
  
    // // Ensure that if the map is zoomed out such that multiple
    // // copies of the feature are visible, the popup appears
    // // over the copy being pointed to.
    // while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
    //   coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360
    // }

    console.log(e.features[0]);
  
    // should be centered, but need to figure out the map dimensions first
    console.log(coordinates);

    let updated_center;
    screen.width < 600 ? updated_center = [ coordinates[0], coordinates[1] - 0.025 ] : updated_center = [ coordinates[0], coordinates[1] - 0.015 ];
    // let updated_center = [ coordinates[0], coordinates[1] - 0.015 ]
    // console.log(updated_center);
    // map.setCenter(updated_center)

    console.log('test');
    
    addPopup(e, coordinates);


  })

  let lineID = null;
  
  map.on('mouseenter', 'transit-line', function (e) {

    console.log(lineID);

    map.getCanvas().style.cursor = 'pointer'

    console.log(e);

    const features = map.queryRenderedFeatures(e.point, {
      layers: [ 'transit-line' ]
    });

    console.log(features);

    if (lineID) {
    map.removeFeatureState({
      source: 'transit-lines',
      id: lineID
    });
  }

  lineID = e.features[0].properties.id;

  map.setFeatureState(
    {
      source: 'transit-lines',
      id: lineID
    },
    {
      hover: true
    }
  );

  console.log(lineID);


  })

  function toggleMapLayer(e) {

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

function addPopup(e, coordinates) {
          // There is a bug that occurs, when you have clicked on one point and are viewing the popup.  If you click on another point, while the first popup is visible, then the popup for the other point will incorrectly display.
    // So this conditional checks to see if a popup already exists on the page, and removes it if it does, to bypass that bug.  Not sure about the behavior causing the root issue — this was simpler for now.
    if (document.getElementById('mini-scroller')) {
      console.log('present');
      document.getElementById('mini-scroller').remove();
    }

    const popups = document.getElementsByClassName('mapboxgl-popup-anchor-top');
    
    while(popups.length > 0){
        popups[0].parentNode.removeChild(popups[0]);
    }

    // let popupHTML;

    // if (e.features[0]._vectorTileFeature?.properties?.network) {
    //   popupHTML = e.features[0]._vectorTileFeature?.properties?.network
    // }
    // else if (e.features[0]._vectorTileFeature.properties?.name) {
    //   popupHTML = e.features[0]._vectorTileFeature?.properties?.name
    // }
    // else {
    //   popupHTML = "Transit Station"
    // }
    

    new mapbox.Popup({
      closeOnClick: true,
    })
      .setLngLat(coordinates)
      .setHTML(`<div id="mini-scroller"></div>`)
      .addTo(map)

          // The miniscroller component is what appears in the popup, and we instantiate it with props from the selected point (most importantly, content) so we can show the content from that point in the popup.
    new MiniScroller({ 
      target: document.getElementById('mini-scroller'), 
      props: { layer:e.features[0].layer.id, properties:e.features[0]._vectorTileFeature.properties, id:e.features[0]._vectorTileFeature.properties.id, network:e.features[0]._vectorTileFeature.properties.network, name:e.features[0]._vectorTileFeature.properties.name } 
     }) // eslint-disable-line no-new
}
</script>

{#if $geojson_store}

<div style="position: absolute; background: white; top: 5%; right: 5%; display: block; padding: 10px;">

  <div on:input|preventDefault={toggleMapLayer}>
    {#if $geojson_store}
    <input type="checkbox" checked name="bicycle-line">
    {/if}
    <label for="bicycle-lines">
      <svg style="vertical-align: text-top" width="23" height="15" viewBox="0 0 33 20" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M0 10H33" stroke="#DE3163" stroke-width="20"/>
      <line y1="10" x2="33" y2="10" stroke="white" stroke-width="2"/>
      </svg>
      Bike routes</label>
    </div>

    <div on:input|preventDefault={toggleMapLayer}>
      {#if $geojson_store}
      <input type="checkbox" name="transit-line" checked> 
      {/if}
      <svg style="vertical-align: text-top" width="23" height="15" viewBox="0 0 33 20" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M0 10H33" stroke="indigo" stroke-width="20"/>
        </svg>        
      <label for="transit-lines">Transit lines</label>
      </div>

    <div on:input|preventDefault={toggleMapLayer}>
      {#if $geojson_store}
  <input type="checkbox" name="bicycle-point" checked>
  {/if}
  <label for="bicycle-points">
    <svg style="vertical-align: top" width="17" height="17" viewBox="0 0 17 17" fill="none" xmlns="http://www.w3.org/2000/svg">
      <circle cx="8.5" cy="8.5" r="6.5" fill="#FF0000" stroke="white" stroke-width="2"/>
      <circle cx="8.5" cy="8.5" r="8.4" stroke="black" stroke-width="0.2"/>
      </svg>
      Bikeshares + Bike shops</label>
  </div>

  <div on:input|preventDefault={toggleMapLayer}>
    {#if $geojson_store}
    <input type="checkbox" name="transit-point" checked>
    {/if}
    <label for="bicycle-points">
      <svg style="vertical-align: top" width="15" height="15" viewBox="0 0 15 15" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M7.5 14.25C11.2279 14.25 14.25 11.2279 14.25 7.5C14.25 3.77208 11.2279 0.75 7.5 0.75C3.77208 0.75 0.75 3.77208 0.75 7.5C0.75 11.2279 3.77208 14.25 7.5 14.25Z" fill="#0096FF" stroke="black" stroke-width="1.5"/>
        </svg>        
       Transit Stations</label>
    </div>
</div>

{/if}