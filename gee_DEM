ee.ImageCollection("COPERNICUS/S2")

//should be define the geometry by the user.

// script to get 
// selected area DEM (SRTM 30m resolution)

// GENERAL CONFIGURATION
// select the area of interest
var area = 'geometry'

// specify the start and end  date
var start = '2008-01-01';
var end = '2010-01-01';

print("Start Date: ", start);
print("End Date: ", end);

// external 
var batch = require('users/fitoprincipe/geetools:batch')
var palettesGeneral = require('users/gena/packages:palettes');

////////////////////////////////////////////////////////
// DIGITAL ELEVATION MODEL
// get the DEM of the area
// change the max elevation according the region (visDem: max: elevation to change)
var dataset = ee.Image('USGS/SRTMGL1_003');
var elevation = dataset.select('elevation');
var clippedDem = elevation.clip(geometry);
var palette = palettesGeneral.kovesi.diverging_rainbow_bgymr_45_85_c67[7];
var visDem = {
  min:0,
  max: 2000,
  palette: palette
};
// center the map
Map.centerObject(geometry, 15)

// add Geometry to map
Map.addLayer(geometry, {}, 'Geometry')

// export DEM to drive
var exportDem = clippedDem.select('elevation')
Export.image.toDrive({
    image: exportDem,
    description: 'demFile',
    folder: 'Gee_project',
    fileNamePrefix: 'demFile',
    region: geometry,
    scale: 30,
    maxPixels: 2e10
})
// add DEM to map
Map.addLayer(clippedDem, visDem, 'DEM');

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
    bbox: [0, 0, 2000, 500],
    dimensions: '200x20',
    format: 'png', 
    min: 0, max: 2000,
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
var colorBar = createColorBar('Digital Elevation Model (m)', palette, 0, 2000)

Map.add(colorBar)

// William Vichete
