# ndvi-belo_horizonte-monoxido-carbono-2020-2023
estimativa do índice de concentração de monóxido de carbono em Belo Horizonte
// Calculate CO concentration (in mol/m² or μg/m²) for any study area  using Time series chart Analysis (Dataset: Sentinel-5P NRTI CO)

// Note:
// CO (nitrogen dioxide) column number density refers to the concentration of CO molecules present in a vertical column of the Earth's atmosphere. 
// It is typically measured in units of molecules per square meter (mol/m²) or micrograms per square meter (μg/m²).


// Coordinates for study area

//declaração de variavel

var BELO_HORIZONTE = ee.FeatureCollection ('users/gleisonarquiteto/BELO_HORIZONTE');

print('BELO_HORIZONTE',BELO_HORIZONTE);

//declaração de variaveis para data dasimagens
var ini = ('2020-01-01');
var fim = ('2020-12-30');
var nuvem= 1;

var sent5 = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017')
.filterBounds (BELO_HORIZONTE)
.filterMetadata ('CLOUD_COVER','less_than', nuvem)
.filterDate (ini,fim);

print ('sent5',sent5); 

Map.addLayer(BELO_HORIZONTE,{},'BELO_HORIZONTE');
Map.centerObject(BELO_HORIZONTE,10);

// 1. Import countries boundaries 
var countries = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017');

// 2. Import Sentinel 5P NRTI NO2
  
  var collection = ee.ImageCollection('COPERNICUS/S5P/NRTI/L3_CO')
  .select('CO_column_number_density')
  .filterDate('2020-01-01', '2020-12-30');
  
// 3. Set visualization parameters
var band_viz = {
  min: 0,
  max: 0.05,
  palette: ['black', 'blue', 'purple', 'cyan', 'green', 'yellow', 'red']
};

// 4. Display & visualize the layer
Map.addLayer(collection.mean().clip(BELO_HORIZONTE), band_viz, 'S5P CO');


// 5. Import sentinel-5P NRTI CO

//In this part, you define the start and end dates for the analysis. 
//Then, you retrieve the Sentinel-5P NRTI CO ImageCollection, filter it based on the specified geometry (study area),
//date range, and select the 'CO_column_number_density' band. 
//Additionally, a mapping function is used to set a 'month' property on each image to facilitate further processing.

var start_time = '2020-01-01'
var end_time = '2020-12-30'

var CO_collection = ee.ImageCollection('COPERNICUS/S5P/NRTI/L3_CO')
.filterBounds(geometry)
.filterDate(start_time,end_time)
.select('CO_column_number_density')
.map(function(a){
  return a.set('month', ee.Image(a).date().get('month'))
})

print(CO_collection)

// 6. Calculate the mean CO concentration for each month 

//Here, you extract the distinct months present in the collection using aggregate_array 
//and distinct. Then, you map over the months and calculate the mean CO concentration
//for each month using filterMetadata and mean. The resulting monthly mean images 
//are stored in a new ImageCollection called CO_final.

var months = ee.List(CO_collection.aggregate_array('month')).distinct()
print(months)

var CO_monthly_conc = months.map(function(x){
  return CO_collection.filterMetadata('month', 'equals', x).mean().set('month', x)
})
var CO_final = ee.ImageCollection.fromImages(CO_monthly_conc)


// 7. Create a time series chart

var chart = ui.Chart.image.series(CO_final, geometry,ee.Reducer.mean(), 5000,'month')
.setOptions({
title: 'CO Concentration',
vAxis: {title: 'Concentration(μg/m²)'},
hAxis: {title: 'Month'}
})
print(chart)

//Finally, you create a time series chart using ui.Chart.image.series, passing the 
//CO_final ImageCollection, study area (geometry), the reducer (ee.Reducer.mean()), a
//scale (5000), and the X-axis variable ('month'). Additional options like the chart title and axis 
//titles are specified using setOptions. The resulting chart is printed



//exportar NDVI
 var collection = ee.ImageCollection('COPERNICUS/S5P/NRTI/L3_CO')
  .select('CO_column_number_density')
  .filterDate('2020-01-01', '2020-12-30');
  
// 3. Set visualization parameters
var band_viz = {
  min: 0,
  max: 0.05,
  palette: ['black', 'blue', 'purple', 'cyan', 'green', 'yellow', 'red']
};

// 4. Display & visualize the layer
Map.addLayer(collection.mean().clip(BELO_HORIZONTE), band_viz, 'S5P CO');
var band_viz = {
  min: 0,
  max: 0.05,
  palette: ['black', 'blue', 'purple', 'cyan', 'green', 'yellow', 'red']
};

var media = collection.reduce(ee.Reducer.mean());
print ('sent5_media', media)




//exportar media
Export.image.toDrive ({image:media.clip(BELO_HORIZONTE),
                       description:'CO_median2020',
                       folder: 'BELO_HORIZONTE',
                       scale: 30,
                       region: BELO_HORIZONTE,
})
