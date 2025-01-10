**Pansharpened Landsat 8 Mosaic Generator**

This script processes Landsat 8 satellite images to create pansharpened mosaics for a specific region and time period. It utilizes Google Earth Engine (GEE) functionalities to filter, mask, and enhance satellite images for visualization and export.
Features

- Region of Interest:
Defines a 3 km x 3 km rectangle around a specified center point.

- Cloud Masking:
Applies a cloud mask using the QA_PIXEL band to ensure minimal cloud cover.

- Pan-Sharpening:
Enhances RGB bands using the panchromatic band via HSV transformation.

- Annual Mosaic Generation:
Creates annual mosaics for a list of specified years.

- Export Functionality:
Exports pansharpened mosaics as high-resolution images to Google Drive.

**Study Case**

**Lumpur Lapindo or Lapindo Mud** refers to the devastating mudflow disaster that began in May 2006 in Sidoarjo, Indonesia. Caused by a combination of drilling activities and geological factors, the mudflow has displaced thousands of residents and created an ongoing environmental and social crisis.
In this code, I use the Lumpur Lapindo study case in Sidoarjo, Indonesia, from 2020 to 2023 and download each year's imagery.

**Script Overview**
**1. Define Region of Interest**
The center point and rectangle for the region of interest are defined:
var center = ee.Geometry.Point([112.7059683, -7.5244837]);
var rectangle = center.buffer(3000).bounds();

**2. Cloud Masking**
Clouds are masked using the QA_PIXEL band:
var maskL8 = function(image) {
  var qa = image.select('QA_PIXEL');
  var mask = qa.bitwiseAnd(1 << 3).eq(0); // Bit 3 for cloud
  return image.updateMask(mask);
};

**3. Pan-Sharpening**
RGB bands are enhanced using the panchromatic band:
var panSharpenL8 = function(image) {
  var rgb = image.select('B4', 'B3', 'B2');
  var pan = image.select('B8');
  var huesat = rgb.rgbToHsv().select('hue', 'saturation');
  var upres = ee.Image.cat(huesat, pan).hsvToRgb();
  return image.addBands(upres.rename(['B4', 'B3', 'B2']), null, true);
};

**4. Mosaic**
Filters and processes Landsat 8 images to create annual mosaics:
function getMosaic(year) {
  var startDate = ee.Date.fromYMD(year, 1, 1);
  var endDate = ee.Date.fromYMD(year, 12, 31);

  var landsatCollection = ee.ImageCollection('LANDSAT/LC08/C02/T1_TOA')
    .filterBounds(rectangle)
    .filterDate(startDate, endDate)
    .filter(ee.Filter.lt('CLOUD_COVER', 20))
    .map(maskL8)
    .map(panSharpenL8);

  var mosaic = landsatCollection.median().clip(rectangle);
  return mosaic;
}

**5. Visualization and Export**
Processes multiple years, visualizes the results on the map, and exports them to Google Drive:
var years = [2020, 2021, 2022, 2023];
years.forEach(function(year) {
  var mosaic = getMosaic(year);
  Map.addLayer(mosaic, {bands: ['B4', 'B3', 'B2'], min: 0, max: 0.3, gamma: 1.4}, 'Pansharpened Mosaic ' + year);

  Export.image.toDrive({
    image: mosaic.visualize({bands: ['B4', 'B3', 'B2'], min: 0, max: 0.3, gamma: 1.4}),
    description: 'Pansharpened_Landsat8_Mosaic_' + year,
    folder: 'Lapindo',
    fileNamePrefix: 'Pansharpened_Landsat8_Mosaic_' + year,
    region: rectangle,
    scale: 15,
    crs: 'EPSG:4326'
  });
});

**6. Map Centering and Boundary Display**
Centers the map and highlights the rectangle:
Map.centerObject(rectangle, 13);
Map.addLayer(rectangle, {color: 'red'}, 'Rectangle');

**Usage**
- Open the script in the Google Earth Engine Code Editor.
- Modify the center coordinates and years as needed.
- Run the script to generate and export pansharpened mosaics to your Google Drive.

**Requirements**
- Google Earth Engine account.
- Access to the Landsat 8 TOA image collection (LANDSAT/LC08/C02/T1_TOA).
  *notes:I use TOA (Top of Atmosphere) instead of Surface Reflectance because I only use for visualization. If you will use it for further analysis like vegetation loss, please use Surface Reflectance product. https://developers.google.com/earth-engine/datasets/catalog/landsat-8*

**Output**
- High-resolution pansharpened mosaics exported to a folder named Lapindo in Google Drive.
- Visualizations of annual mosaics in the Google Earth Engine Map interface.

**Notes**
- The script uses a 15-meter resolution for pansharpened exports.
- Ensure the Google Drive folder Lapindo exists, or adjust the folder name in the script.
