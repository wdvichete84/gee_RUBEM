// import Sentinel-2 1c
var S2_SR = ee.ImageCollection('COPERNICUS/S2_SR')

// Filter cloud, date, geometry and bands (generate an image collection)
var filtered = S2_SR
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 30))
  .filter(ee.Filter.date('2022-11-01', '2022-12-25'))
  .filter(ee.Filter.bounds(geometry))
  .select('B.*')

// center map at the geometry
Map.centerObject(geometry, 16);

// compute the NDVI - generic for all image in the collection
var addNDVI = function(image) {
  var ndvi = image.normalizedDifference(['B8', 'B4']).rename('NDVI');
  return image.addBands(ndvi);
};

// add the NDVI to S2 image collection
var S2_NDVI = filtered.map(addNDVI);

// clip the image with ndvi bands to geometry region
// median could be change by the minimum and maximum
var composite = S2_NDVI.median().clip(geometry) 

// select the last image of S2_NDVI
var recent_S2 = ee.Image(S2_NDVI.sort('system:time_start', false).first()).clip(geometry);


// print console the last and composite image with NDVI
print('Last image: ', recent_S2);
print('Composite image: ', composite);

// image visualization
var ndviVis = {min: 0, max: 1, palette: ['blue', 'white', 'green']};

// add ndvi composite and last images to map
Map.addLayer(recent_S2.select('NDVI'), ndviVis, 'Recent Sentinel NDVI');
Map.addLayer(composite.select('NDVI'), ndviVis, 'Mean NDVI');

// only ndvi band in a image to export
var recentNdvi_export = recent_S2.select('NDVI');
var meanNdvi_export = composite.select('NDVI');

// export ndvi tif file
Export.image.toDrive({
  image: recentNdvi_export,
  description: 'recentNdviresults',
  folder: 'NDVI_sensor',
  fileNamePrefix: 'recent_results',
  region: geometry,
  scale: 10,
  maxPixels: 2e10
})

Export.image.toDrive({
  image: meanNdvi_export,
  description: 'meanNdviresults',
  folder: 'NDVI_sensor',
  fileNamePrefix: 'mean_results',
  region: geometry,
  scale: 10,
  maxPixels: 2e10
})

// William D. Vichete
