# **Pansharpened Landsat 8 Mosaic**

This script processes Landsat 8 satellite images to create pansharpened mosaics for a specific region and time period. It utilizes **Google Earth Engine (GEE)** functionalities to filter, mask, and enhance satellite images for visualization and export. The script is designed to maintain natural colors for visualization and analysis.

## **Features**

- **Region of Interest:**
  - Defines a 3 km x 3 km rectangle around a specified center point.
- **Cloud Masking:**
  - Applies a cloud mask using the `QA_PIXEL` band to ensure minimal cloud cover.
- **Pan-Sharpening:**
  - Enhances RGB bands using the panchromatic band via HSV transformation.
  - Maintains natural color appearance (e.g., vegetation appears green).
- **Annual Mosaic Generation:**
  - Creates annual mosaics for a list of specified years.
- **Export Functionality:**
  - Exports pansharpened mosaics as high-resolution images to Google Drive.

## **Study Case**

**Lumpur Lapindo** refers to the devastating mudflow disaster that began in May 2006 in **Sidoarjo, Indonesia**. Caused by a combination of drilling activities and geological factors, the mudflow has displaced thousands of residents and created an ongoing environmental and social crisis.

In this code, the Lumpur Lapindo study case in Sidoarjo, Indonesia, is analyzed from **2020 to 2024**, with annual pansharpened mosaics generated for each year.

## **Script Overview**

### **1. Define Region of Interest**
The center point and rectangle for the region of interest are defined:
```javascript
var center = ee.Geometry.Point([112.7059683, -7.5244837]);
var rectangle = center.buffer(3000).bounds();
```

### **2. Cloud Masking**
Clouds are masked using the `QA_PIXEL` band:
```javascript
var maskL8 = function(image) {
  var qa = image.select('QA_PIXEL');
  var mask = qa.bitwiseAnd(1 << 3).eq(0); // Bit 3 for cloud
  return image.updateMask(mask);
};
```

### **3. Pan-Sharpening**
RGB bands are enhanced using the panchromatic band:
```javascript
var panSharpenL8 = function(image) {
  var rgb = image.select('B4', 'B3', 'B2');
  var pan = image.select('B8');
  var huesat = rgb.rgbToHsv().select('hue', 'saturation');
  var pansharpenedRgb = ee.Image.cat(huesat, pan).hsvToRgb();
  return image.addBands(pansharpenedRgb.rename(['B4_ps', 'B3_ps', 'B2_ps']), null, true);
};
```

### **4. Mosaic Generation**
Filters and processes Landsat 8 images to create annual mosaics:
```javascript
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
```

### **5. Visualization and Export**
Processes multiple years, visualizes the results on the map, and exports them to Google Drive:
```javascript
var years = [2020, 2021, 2022, 2023, 2024];
years.forEach(function(year) {
  var mosaic = getMosaic(year);
  Map.addLayer(mosaic, {bands: ['B4_ps', 'B3_ps', 'B2_ps'], min: 0.02, max: 0.25, gamma: 1.2}, 'Pansharpened Natural Mosaic ' + year);

  Export.image.toDrive({
    image: mosaic.visualize({bands: ['B4_ps', 'B3_ps', 'B2_ps'], min: 0.02, max: 0.25, gamma: 1.2}),
    description: 'Natural_Pansharpened_Landsat8_Mosaic_' + year,
    folder: 'Lapindo',
    fileNamePrefix: 'Natural_Pansharpened_Landsat8_Mosaic_' + year,
    region: rectangle,
    scale: 15,
    crs: 'EPSG:4326'
  });
});
```

### **6. Map Centering and Boundary Display**
Centers the map and highlights the rectangle:
```javascript
Map.centerObject(rectangle, 13);
Map.addLayer(rectangle, {color: 'red'}, 'Rectangle');
```

## **Usage**
1. Open the script in the **Google Earth Engine Code Editor**.
2. Modify the center coordinates and years as needed.
3. Run the script to generate and export pansharpened mosaics to your Google Drive.

## **Requirements**
- **Google Earth Engine account.**
- **Access to the Landsat 8 TOA image collection** (`LANDSAT/LC08/C02/T1_TOA`).

## **Output**
- High-resolution pansharpened mosaics exported to a folder named `Lapindo` in Google Drive.
- Visualizations of annual mosaics in the Google Earth Engine Map interface.

## **Notes**
- The script uses a **15-meter resolution** for pansharpened exports.
- Ensure the Google Drive folder `Lapindo` exists, or adjust the folder name in the script.

## **License**
This script is provided for **educational and research purposes**. Use at your own discretion.

