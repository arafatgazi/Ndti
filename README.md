# Ndti
#get the data landsat -and then accroding to the point ,get the tiff file.
// Add Sentinel 2 Image Collection and Filter
var geometry = ee.Geometry.Polygon([
  [[92.18000,20.71999],
   [92.18000,20.48000],
   [92.37000,20.71999],
   [92.37000,20.71999]
  ]
]);
var sentinelimage = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .select(['SR_B3', 'SR_B4', 'SR_B5'])
  .filterDate('2024-12-01','2024-12-30')
  .filterBounds(geometry)
  .filter(ee.Filter.lt('CLOUD_COVER', 10))
  .mean()
  .multiply(0.0001);

//NDWI Calculation

var ndwi = sentinelimage.normalizedDifference(['SR_B3', 'SR_B5']).rename("NDWI")

//create a binary raster (Set a Threshold on NDWI to Detect Water Areas)

var watermask = ndwi.gt(0).rename('watermask')

//NDTI Calculation

var ndti = sentinelimage.normalizedDifference(['SR_B4', 'SR_B3']).rename("NDTI")




//Clip NDTI from water mask

var clipndti = ndti.updateMask(watermask)
  var palette = [
 'blue','green','yellow','orange','red'
];

Map.addLayer(clipndti.clip(geometry), {
  palette:palette
  
  }, 'clipndti' , false)
  
  

var visParams = {
  min: -1,
  max: 1,
  palette: palette
};

Map.addLayer(ndti.clip(geometry), visParams, 'NDTI');

  
  
  
  // Create the legend title
var legendTitle = ui.Label({
  value: 'NDTI Legend',
  style: {fontWeight: 'bold', fontSize: '14px', margin: '0 0 4px 0', padding: '0'}
});

// Create a panel to hold the legend
var legendPanel = ui.Panel({
  style: {
    position: 'bottom-center',
    
   
    padding: '8px 55px'
  }
});
legendPanel.add(legendTitle);

// Define gradient labels (optional)
var minLabel = ui.Label('-1');
var maxLabel = ui.Label('1');
var legendLabels = ui.Panel({
  widgets: [minLabel, ui.Label({value: '', style: {stretch: 'horizontal'}}), maxLabel],
  layout: ui.Panel.Layout.flow('horizontal')
});

// Create color bar thumbnail
var colorBar = ui.Thumbnail({
  image: ee.Image.pixelLonLat().select(0).multiply((1 - (-1)) / 100).add(-1),
  params: {
    bbox: [0, 0, 100, 10],
    dimensions: '100x10',
    format: 'png',
    min: -1,
    max: 1,
    palette: palette
  },
  style: {stretch: 'horizontal', maxHeight: '20px'}
});

// Add color bar and labels to legend panel
legendPanel.add(colorBar);
legendPanel.add(legendLabels);

// Add to map
Map.add(legendPanel);
// export image to drive 
Export.image.toDrive({
  image: clipndti,

  // image: clipndti.clip(geometry),
  description: 'NDTI_Clip_Jan10_20_2024',
  folder: 'ndti',                    
  fileNamePrefix: 'ndti_clip_2024_01_10_20', 
  region: geometry,                         
  scale: 30,                               
  crs: 'EPSG:4326',                       
  maxPixels: 1e13                        
});  
