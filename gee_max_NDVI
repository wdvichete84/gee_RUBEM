// Modis sensor

// should be define the geometry by the user

// script to get 
// mean mounthly NDVI

// GENERAL CONFIGURATION
// select the area of interest
var area = 'geometry'

// specify the start and end  date
var startDate = ee.Date('2008-01-01');
var endDate = ee.Date('2010-01-01');

print("Start Date: ", startDate);
print("End Date: ", endDate);

// define the numbers of months between start and end date
var diff = endDate.difference(startDate, 'month');

// external 
var batch = require('users/fitoprincipe/geetools:batch')
var palettesGeneral = require('users/gena/packages:palettes');

// center map
Map.centerObject(geometry, 5)

// add Geometry to map
Map.addLayer(geometry, {}, 'Geometry')

// NDVI
// collection of images
var dataset = ee.ImageCollection('MODIS/006/MOD13Q1')
  .filter(ee.Filter.date(startDate, endDate));

// Scaled range up to 1
var modisNDVI = dataset.select('NDVI');
var scaledNDVI = modisNDVI.map(function(image){
  return image.multiply(0.0001)
  .copyProperties(image,['system:time_start','system:time_end']);
});

var triplets = scaledNDVI.map(function(image) {
  var withStats = image.reduceRegions({
  collection: geometry,
  reducer: ee.Reducer.mean().setOutputs(['ndvi']),
  scale: 250
  }).map(function(feature) {
    return feature.set('imageId', image.id())
  })
  return withStats
}).flatten()

// NDVI visualization
var palette = palettesGeneral.colorbrewer.RdYlGn[11];
var ndviVis = {
  min: 0.0,
  max: 1.0,
  palette: palette,
};

// define min monthly NDVI 
var monthMax = ee.List.sequence(0, diff).map(function(n) {
  var start = ee.Date(startDate).advance(n, 'month');
  var end = start.advance(1, 'month');
  return ee.ImageCollection(scaledNDVI)
        .filterDate(start, end)
        .max()
        .set('system:time_start', start.millis());
});

// create image collection from monthly avg
var collection = ee.ImageCollection(monthMax);

// clip images to the polygon boundary
var clippedNdvi = collection.filter(ee.Filter.date(startDate, endDate))
  .map(function(image) {
    return ee.Image(image).clip(geometry)
  })

// Plotting chart of monthly Rainfall
var title = 'Maximum Monthly NDVI of Geometry';

var timeSeries = ui.Chart.image.seriesByRegion({
    imageCollection: clippedNdvi,
    regions: geometry,
    reducer: ee.Reducer.mean(),
    scale: 250,
    xProperty: 'system:time_start',
    seriesProperty: 'label'
  }).setChartType('ScatterChart')
    .setOptions({
      title: title,
      vAxis: {title: '[NDVI]'},
      hAxis: {title: 'Year'},
      lineWidth: 1,
      pointSize: 1,
    });

print(timeSeries);
  
// Download images for a set region
batch.Download.ImageCollection.toDrive(clippedNdvi, 'NDVI', 
  {region: clippedNdvi,
  crs: 'EPSG:4326',
  type: 'float',
  description: 'imageToDriveExample',
  scale: 250,
  fileFormat: 'GeoTIFF',
});

// add the first NDVI image to map
Map.addLayer(clippedNdvi, ndviVis, 'NDVI');

// Add bar Legend
function createColorBar(titleText, palette, min, max) {
// Legend Title
var title = ui.Label({
  value: titleText, 
  style: {fontWeight: 'bold', textAlign: 'center', stretch: 'horizontal'}});

// Colorbar
var legend = ui.Thumbnail({
  image: ee.Image.pixelLonLat().select(0),
  params: {
    bbox: [0, 0, 1, 0.1],
    dimensions: '200x20',
    format: 'png', 
    min: 0, max: 1,
    palette: palette},
  style: {stretch: 'horizontal', margin: '8px 8px', maxHeight: '40px'},
});
  
// Legend Labels
var labels = ui.Panel({
  widgets: [
    ui.Label(min, {margin: '4px 10px',textAlign: 'left', stretch: 'horizontal'}),
    ui.Label((min+max)/2, {margin: '4px 20px', textAlign: 'center', stretch: 'horizontal'}),
    ui.Label(max, {margin: '4px 10px',textAlign: 'right', stretch: 'horizontal'})],
  layout: ui.Panel.Layout.flow('horizontal')});
  
// Create a panel with all 3 widgets
var legendPanel = ui.Panel({
  widgets: [title, legend, labels],
  style: {position: 'bottom-center', padding: '8px 15px'}
})
return legendPanel
}

// Call the function to create a colorbar legend  
var colorBar = createColorBar('NDVI - First Image ', palette, 0, 1)

Map.add(colorBar)

/// William Vichete
