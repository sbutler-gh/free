<script>
  import { getContext } from 'svelte'
  import { contextKey } from '$lib/components.js'
  import MiniScroller from './MiniScroller.svelte'
  import { bikeshare_transit_store, points_prompt_store, bikeshare_store, transit_store } from "$lib/stores.js"

  const { getMap, getMapbox } = getContext(contextKey)
  const map = getMap()
  const mapbox = getMapbox()

  let load = false;

  // Add a new source from our GeoJSON data and set the
  // 'cluster' option to true. GL-JS will add the point_count property to your source data.

  // We check to see if the mapsource has already been drawn, in which case we just want to update it via the map.getSource('content').setData() function
  // This conditional is filled once the page has already loaded, and we destroy the <Content> component to update the data, and then refresh it
  // When it refreshes, map.getSource('content') returns true and this conditional continues

  if ($bikeshare_store) {

    if (map.getLayer('bikeshare-point')) {

      console.log('removing');
      map.removeLayer('bikeshare-point'); 
      map.removeSource('bikeshare-points');
    }

  map.addSource('bikeshare-points', {
    type: 'geojson',
    data: {
        "type": "FeatureCollection",
        "features": $bikeshare_store
    }
  })

//   map.loadImage(
// './bikeshare.png',
// (error, image) => {
// if (error) throw error;
 
// // Add the image to the map style.
// map.addImage('bikeshare', image);
// })

//   map.addLayer({
//   'id': 'bikeshare-point',
//   'type': 'symbol',
//   'source': 'bikeshare-points',
//   'layout': {
// 'icon-image': 'bikeshare', // reference the image
// 'icon-size': 0.05
// }
// });

    map.addLayer({
  'id': 'bikeshare-point',
  'type': 'circle',
  'source': 'bikeshare-points',
      'paint': {
        'circle-color': 'red',
        'circle-radius': 6,
        'circle-stroke-width': 1.5,
        'circle-stroke-color': 'WHITE'
    },
  });


// 'layout': {
// 'text-field': ['🚲'],
// 'text-variable-anchor': ['top', 'bottom', 'left', 'right'],
// 'text-radial-offset': 0.5,
// 'text-justify': 'auto',
// }



  // 'layout': {
  //   'icon-image': [
  //     'match',
  //     [ 'get', 'type' ], // type corresponds to the field name you are keying off of
  //     [ 'bike parking' ],
  //     'custom-marker',
  //     [ 'pedestrian' ],
  //     'custom-marker-2',
  //     'custom-marker' // fallback icon
  //   ],
  //   // get the title name from the source's "title" property
  //   'text-field': [ 'get', 'title' ],
  //   'text-font': [
  //     'Open Sans Semibold',
  //     'Arial Unicode MS Bold'
  //   ],
  //   'text-offset': [ 0, 1.25 ],
  //   'text-anchor': 'top'
  // }

}

  //   map.addLayer({
  // 'id': 'bikeshare-point',
  // 'type': 'circle',
  // 'source': 'bikeshare-points',
  //     'paint': {
  //       'circle-color': 'red',
  //       'circle-radius': 6,
  //       'circle-stroke-width': 1.5,
  //       'circle-stroke-color': 'black'
  //   },
  // });

else {
  console.log('none yet')
}

if ($transit_store) {

  if (map.getLayer('transit-line')) {

    console.log('removing;')
    map.removeLayer('transit-line'); 
    map.removeLayer('transit-point'); 
    map.removeSource('transit-data');
  }

map.addSource('transit-data', {
  type: 'geojson',
  data: {
        "type": "FeatureCollection",
        "features": $transit_store
    }
})

map.addLayer({
    'id': 'transit-line',
    'generateid': true,
    'type': 'line',
    'source': 'transit-data',
    'layout': {
    'line-join': 'round',
    'line-cap': 'round'
    },
    'paint': {
    'line-color': [
      'case',
      ['boolean', ['feature-state', 'hover'], false],
      'yellow',
      '#BF40BF'
    ],
    'line-width': 8
    }
    });

  map.addLayer({
'id': 'transit-point',
'type': 'circle',
'source': 'transit-data',
    'paint': {
      'circle-color': 'royalblue',
      'circle-radius': 6,
      'circle-stroke-width': 1.5,
      'circle-stroke-color': 'black'
  },
});

}
else {
console.log('none yet')
}


//   let geojson_array = $geojson_store.features;

//   console.log(geojson_array);

//   console.log(geojson_array.filter(feature => feature.geometry.type == "MultiLineString"));
//   console.log(geojson_array.filter(feature => feature.geometry.type == "Point"));

//   map.addSource('transit-line', {
//     type: 'geojson',
//     data: $geojson_store
//   })

//   // map.addSource('transit-line', {
//   //   type: 'geojson',
//   //   data: {
//   //       "type": "FeatureCollection",
//   //       "features": geojson_array.filter(feature => feature.geometry.type == "MultiLineString")
//   //   }
//   // })

//   // map.addSource('transit-point', {
//   //   type: 'geojson',
//   //   data: {
//   //       "type": "FeatureCollection",
//   //       "features": geojson_array.filter(feature => feature.geometry.type == "Point")
//   //   }
//   // })

//   map.addLayer({
//     'id': 'transit-line',
//     'type': 'line',
//     'source': 'transit-line',
//     'layout': {
//     'line-join': 'round',
//     'line-cap': 'round'
//     },
//     'paint': {
//     'line-color': 'purple',
//     'line-width': 8
//     }
//     });


// map.addLayer({
// 'id': 'transit-point',
// 'type': 'circle',
// 'source': 'transit-line',
//     'paint': {
//       'circle-color': '#11b4da',
//       'circle-radius': 6,
//       'circle-stroke-width': 1.5,
//       'circle-stroke-color': 'black'
//   },
// });


//   if (map.getSource('content')) {
//     console.log('exists');
//     let data = {
//         "type": "FeatureCollection",
//         "crs": { "type": "name", "properties": { "name": "urn:ogc:def:crs:OGC:1.3:CRS84" } },
//         "features": $points_prompt_store,
//     }
//     map.getSource('content').setData(data);
//   }

//   // If the map source hasn't yet been drawn, then we do everything from scratch — this happens on initial page load
//   else {

//   map.addSource('content', {
//     type: 'geojson',
    
//     // Point to GeoJSON data. This example visualizes all points for the selected prompt
//     // data:  {
//     //     "type": "FeatureCollection",
//     //     "crs": { "type": "name", "properties": { "name": "urn:ogc:def:crs:OGC:1.3:CRS84" } },
//     //     "features": $points_prompt_store,
//     // },
//     cluster: true,
//     clusterMaxZoom: 14, // Max zoom to cluster points on
//     clusterRadius: 50 // Radius of each cluster when clustering points (defaults to 50)
//   })

//   map.addLayer({
//     id: 'clusters',
//     type: 'circle',
//     source: 'content',
//     filter: [ 'has', 'point_count' ],
//     paint: {
//       // Use step expressions (https://docs.mapbox.com/mapbox-gl-js/style-spec/#expressions-step)
//       // with three steps to implement three types of circles:
//       //   * Blue, 20px circles when point count is less than 100
//       //   * Yellow, 30px circles when point count is between 100 and 750
//       //   * Pink, 40px circles when point count is greater than or equal to 750
//       'circle-color': [
//         'step',
//         [ 'get', 'point_count' ],
//         '#51bbd6',
//         100,
//         '#f1f075',
//         750,
//         '#f28cb1'
//       ],
//       'circle-radius': [
//         'step',
//         [ 'get', 'point_count' ],
//         20,
//         100,
//         30,
//         750,
//         40
//       ]
//     }
//   })

//   map.addLayer({
//     id: 'cluster-count',
//     type: 'symbol',
//     source: 'content',
//     filter: [ 'has', 'point_count' ],
//     layout: {
//       'text-field': '{point_count_abbreviated}',
//       'text-font': [ 'DIN Offc Pro Medium', 'Arial Unicode MS Bold' ],
//       'text-size': 12
//     }
//   })

//   map.addLayer({
//     id: 'unclustered-point',
//     type: 'circle',
//     source: 'content',
//     filter: [ '!', [ 'has', 'point_count' ] ],
//     paint: {
//       'circle-color': '#11b4da',
//       'circle-radius': 6,
//       'circle-stroke-width': 1.5,
//       'circle-stroke-color': 'black'
//     }
//   })

// }

  // map.on('click', 'transit', function (e) {
  //   const features = map.queryRenderedFeatures(e.point, {
  //     layers: [ 'transit' ]
  //   })
  //   const clusterId = features[0].properties.cluster_id
  //   map.getSource('content').getClusterExpansionZoom(clusterId, function (err, zoom) {
  //     if (err) { return }

  //     map.easeTo({
  //       center: features[0].geometry.coordinates,
  //       zoom: zoom
  //     })
  //   })
  // })

  map.on('click', 'bikeshare-point', function (e) {

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
    if (document.getElementById('mini-scroller')) {
      console.log('present');
      document.getElementById('mini-scroller').remove();
    }

    new mapbox.Popup({})
      .setLngLat(coordinates)
      .setHTML('<div id="mini-scroller"></div>')
      .addTo(map)

    // The miniscroller component is what appears in the popup, and we instantiate it with props from the selected point (most importantly, content) so we can show the content from that point in the popup.
    new MiniScroller({ 
      target: document.getElementById('mini-scroller'), 
      props: { layer:e.features[0].layer.id, properties:e.features[0]._vectorTileFeature.properties, id:e.features[0]._vectorTileFeature.properties.id, shop:e.features[0]._vectorTileFeature.properties?.shop, network:e.features[0]._vectorTileFeature.properties.network, name:e.features[0]._vectorTileFeature.properties.name } 
     }) // eslint-disable-line no-new
  })

  map.on('mouseenter', 'bikeshare-point', function () {
    map.getCanvas().style.cursor = 'pointer'
  })

  map.on('mouseleave', 'bikeshare-point', function () {
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

    console.log('test');

    // There is a bug that occurs, when you have clicked on one point and are viewing the popup.  If you click on another point, while the first popup is visible, then the popup for the other point will incorrectly display.
    // So this conditional checks to see if a popup already exists on the page, and removes it if it does, to bypass that bug.  Not sure about the behavior causing the root issue — this was simpler for now.
    if (document.getElementById('mini-scroller')) {
      console.log('present');
      document.getElementById('mini-scroller').remove();
    }

    new mapbox.Popup({})
      .setLngLat(coordinates)
      .setHTML('<div id="mini-scroller"></div>')
      .addTo(map)

    // The miniscroller component is what appears in the popup, and we instantiate it with props from the selected point (most importantly, content) so we can show the content from that point in the popup.
    new MiniScroller({ 
      target: document.getElementById('mini-scroller'), 
      props: { layer:e.features[0].layer.id, properties:e.features[0]._vectorTileFeature.properties, id:e.features[0]._vectorTileFeature.properties.id, network:e.features[0]._vectorTileFeature.properties.network, name:e.features[0]._vectorTileFeature.properties.name } 
     }) // eslint-disable-line no-new
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

    // There is a bug that occurs, when you have clicked on one point and are viewing the popup.  If you click on another point, while the first popup is visible, then the popup for the other point will incorrectly display.
    // So this conditional checks to see if a popup already exists on the page, and removes it if it does, to bypass that bug.  Not sure about the behavior causing the root issue — this was simpler for now.
    if (document.getElementById('mini-scroller')) {
      console.log('present');
      document.getElementById('mini-scroller').remove();
    }

    new mapbox.Popup({})
      .setLngLat(coordinates)
      .setHTML('<div id="mini-scroller"></div>')
      .addTo(map)

    // The miniscroller component is what appears in the popup, and we instantiate it with props from the selected point (most importantly, content) so we can show the content from that point in the popup.
    new MiniScroller({ 
      target: document.getElementById('mini-scroller'), 
      props: { layer:e.features[0].layer.id, properties:e.features[0]._vectorTileFeature.properties, id:e.features[0]._vectorTileFeature.properties.id, network:e.features[0]._vectorTileFeature.properties.network, name:e.features[0]._vectorTileFeature.properties.name } 
     }) // eslint-disable-line no-new
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
      source: 'transit-data',
      id: lineID
    });
  }

  lineID = e.features[0].properties.id;

  map.setFeatureState(
    {
      source: 'transit-data',
      id: lineID
    },
    {
      hover: true
    }
  );

  console.log(lineID);


  })
</script>