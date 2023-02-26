ee.ImageCollection("UCSB-CHG/CHIRPS/PENTAD")

//should be define the geometry by the user

// script to get 
// mounthly precipitation in an area
// specify the start and end  date
var startDate = ee.Date('2008-01-01');
var endDate = ee.Date('2010-01-01');//

print("Start Date: ", startDate);
print("End Date: ", endDate);

// define the numbers of months between start and end date
var diff = endDate.difference(startDate, 'month');

// external require
var batch = require('users/fitoprincipe/geetools:batch')
var palettesGeneral = require('users/gena/packages:palettes');

// center the map
Map.centerObject(geometry, 5)

// add Geometry to map
Map.addLayer(geometry, {}, 'Geometry')

// define monthly Rainfall 
var monthSum = ee.List.sequence(0, diff).map(function(n) {
  var start = ee.Date(startDate).advance(n, 'month');
  var end = start.advance(1, 'month');
  return ee.ImageCollection("UCSB-CHG/CHIRPS/DAILY")
        .filterDate(start, end)
        .sum()
        .set('system:time_start', start.millis());
});

// create image collection from monthly sum
var collection = ee.ImageCollection(monthSum);

// clip images to the polygon boundary
var clippedRain = collection.filter(ee.Filter.date(startDate, endDate))
  .map(function(image) {
    return ee.Image(image).clip(geometry)
  })

// Plotting chart of monthly Rainfall
var title = 'Mean Monthly Rainfall of Geometry';

var timeSeries = ui.Chart.image.seriesByRegion({
    imageCollection: collection,
    regions: geometry,
    reducer: ee.Reducer.mean(),
    scale: 5000,
    xProperty: 'system:time_start',
    seriesProperty: 'label'
  }).setChartType('ScatterChart')
    .setOptions({
      title: title,
      vAxis: {title: '[mm]'},
      hAxis: {title: 'Year'},
      lineWidth: 1,
      pointSize: 1,
    });

print(timeSeries);

//Precipitation visualization
var palette = palettesGeneral.misc.jet[7]
var visPrep = {
  min:0,
  max: 300,
  palette: palette
};

// Map first image of collection in the area of interest
Map.addLayer(clippedRain.first(), visPrep, 'Monthly Rainfall')

// Export image collection
batch.Download.ImageCollection.toDrive(clippedRain, 'Rainfall', 
  {region: clippedRain,
  crs: 'EPSG:4326',
  type: 'float',
  description: 'imageToDriveExample',
  scale: 5000,
  fileFormat: 'GeoTIFF',
});
// William Vichete
