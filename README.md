# PM2.5_Thailand
PM 2.5 of Thailand and neighborhood country
///the referenced code form https://www.youtube.com/watch?v=l4qzcKjqOfE
//

//Style Map
var AOD_vis = {
  min: 0.029,
  max: 1.476,
  palette: ['white', 'yellow','orange', 'red', 'purple', 'black']
};
var pm25_vis = {
  min: 5.873,
  max: 40.434,
  palette: ['white', 'yellow','orange', 'red', 'purple', 'black']
};

//------------------THAI-------------
//1. Data Management--------------
//Load Aoi Geometry
var datasetf = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017');
var geometry_T = datasetf.filter(ee.Filter.eq('country_na', 'Thailand'));//

//
Map.centerObject(geometry_T, 5)
Map.addLayer(geometry_T, {}, 'Thailand')

//Data Parameter 
var dStart = ee.Date('2024-04-01');
var dEnd = ee.Date(Date.now());

//Terra & Aqua MAIAC Land Aerosol Optical Depth Daily 1 km
var dataset = ee.ImageCollection("MODIS/061/MCD19A2_GRANULES") //
  .filterDate(dStart,dEnd)
  .filter(ee.Filter.bounds(geometry_T))
  .select('Optical_Depth_047') //
  ;
print(dataset);

//Applies scaling factors
function applyScaleFactors(image){  //
  var Depth_047Bands = image.select('Optical_Depth_047').multiply(0.001); //
  return image.addBands(Depth_047Bands, null, true);
}

//Raster Data Processing
var collection1 = dataset.map(applyScaleFactors); //
var AOD_img = collection1.mosaic().clip(geometry_T); // เ
//Map.addLayer(AOD_img, {}, 'AOD_img');


//2. Tranform Terra & Aqua MAIAC to PM 2.5-------------
// y = 23.89x + 5.18

//
function transform_to_pm25(image){
  var tpm25 = image.select('Optical_Depth_047').multiply(0.001).multiply(23.89).add(5.18); //
  return image.addBands(tpm25, null, true);
}

//Raster Data Processing
var collection2 = dataset.map(transform_to_pm25);
var PM25_img = collection2.mosaic().clip(geometry_T);


//3. TimeSeries Chart & Analysis-----------
//Display a time-series chart
var chart_reg1 = ui.Chart.image.series({
  imageCollection: collection1,
  region: geometry_T,
  reducer: ee.Reducer.mean(),
  scale: 1000   //1000 เมตร
}).setSeriesNames(['AOD 0.47']) //
.setOptions({
    lineWidth: 1,
    title: 'Thailand Land Aerosol Optical Depth[AOI]',
    interpolateNulls: true,
    vAxis: {title: 'AOD 0.47 μm'},
    hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 10}},
    trendlines: {0: {color: 'CC0000'}
    },
});
print(chart_reg1);

//PM 2.5 chart
var chart_reg2 = ui.Chart.image.series({
  imageCollection: collection2,
  region: geometry_T,
  reducer: ee.Reducer.mean(),
  scale: 1000   //1000 
}).setSeriesNames(['PM 2.5']) //
.setOptions({
    lineWidth: 1,
    title: 'ThaiLand A Satellite-Based PM 2.5 [AOI]',
    interpolateNulls: true,
    vAxis: {title: 'PM 2.5'},
    hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 10}},
    trendlines: {0: {color: '00CC00'}
    },
});
print(chart_reg2);


// //4. Create User Interface
// //Create a panel to hold our widgets.
// var panel_T = ui.Panel();
// panel_T.style().set('width', '500px');

// //Create an intro panel with labels.
// var intro = ui.Panel([
//   ui.Label({
//     value: 'Satellite-Based PM 2.5 Report Chart',
//     style: {fontSize: '20px', fontWeight: 'bold'}
//   }),
//   ui.Label('location')
//   ]);
//   panel_T.add(intro);
//   var layer = Map.layers();
  
// //panels to hold lon/lat values  
// var lon = ui.Label();
// var lat = ui.Label();
// panel_T.add(ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));

// //5. Registers a callback for onClick event 
// Map.onClick(function(coords){  //
  
// //Input inspector location
// var point = ee.Geometry.Point(coords.lon, coords.lat);

// if(layer.get(2) !== undefined){Map.remove(layer.get(2));} //

// //Zoom to location
// Map.centerObject(point,8);

// //Create point object 
// Map.addLayer(ee.FeatureCollection(point)
// .draw({color: 'black', pointRadius: 5, strokeWidth: 2}), {}, 'Observation Target');

// //Create Online Chartvar
// var ui_chart1 = ui.Chart.image.series({
//   imageCollection: collection1,
//   region: point, //ใช้ทั้งจังหวัด
//   reducer: ee.Reducer.median(), //
//   scale: 1000
// }).setSeriesNames(['AOD 0.47']);

// ui_chart1.setOptions({
//     lineWidth: 1,
//     title: 'ThaiLand Land Aerosol Optical Depth Daily',
//     interpolateNulls: true,
//     vAxis: {title: 'AOD 0.47'},
//     hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 20}},
//     trendlines: {0: {color: 'CC0000'},
//     }
// });

// var ui_chart2 = ui.Chart.image.series({
//   imageCollection: collection2,
//   region: point,
//   reducer: ee.Reducer.median(),
//   scale: 1000
// }).setSeriesNames(['PM 2.5']);

// ui_chart2.setOptions({
//     lineWidth: 1,
//     title: 'Thailand A Satellite-Based PM 2.5 (estimation)',
//     interpolateNulls: true,
//     vAxis: {title: 'PM 2.5'},
//     hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 20}},
//     trendlines: {0: {color: '00CC00'},
//     },
// });

// //Add Chart Object to panel
// panel_T.widgets().set(2, ui_chart1);
// panel_T.widgets().set(3, ui_chart2);

// //Set Location Value
// lon.setValue('lon: ' + coords.lon.toFixed(3));
// lat.setValue('lat: ' + coords.lat.toFixed(3));
// });


// //6. Show Result with Google Map API-----------
Map.style().set('cursor', 'crosshair');
Map.centerObject(geometry_T, 6);
Map.addLayer(AOD_img, AOD_vis, 'ThaiLand Aerosol optical depth over land (0.47 μm)', false);
Map.addLayer(PM25_img, pm25_vis, 'Thailand Satellite-Based PM2.5');

// //add the panel to the ui.root
// ui.root.insert(0, panel_T);



//------------------Laos-------------
//1. Data Management--------------
//Load Aoi Geometry
var datasetf = ee.FeatureCollection('USDOS/LSIB_SIMPLE/2017');
var geometry_L = datasetf.filter(ee.Filter.eq('country_na', 'Laos'));//

//
Map.centerObject(geometry_L, 5)
Map.addLayer(geometry_L, {}, 'Laos')

//Data Parameter 
var dStart = ee.Date('2024-04-01');
var dEnd = ee.Date(Date.now());

//Terra & Aqua MAIAC Land Aerosol Optical Depth Daily 1 km
var dataset = ee.ImageCollection("MODIS/061/MCD19A2_GRANULES") //
  .filterDate(dStart,dEnd)
  .filter(ee.Filter.bounds(geometry_L))
  .select('Optical_Depth_047') //
  ;
print(dataset);

//Applies scaling factors
function applyScaleFactors(image){  //
  var Depth_047Bands = image.select('Optical_Depth_047').multiply(0.001); //
  return image.addBands(Depth_047Bands, null, true);
}

//Raster Data Processing
var collection3 = dataset.map(applyScaleFactors); //map.
var AOD_img = collection3.mosaic().clip(geometry_L); // 
//Map.addLayer(AOD_img, {}, 'AOD_img');


//2. Tranform Terra & Aqua MAIAC to PM 2.5-------------
// y = 23.89x + 5.18

//
function transform_to_pm25(image){
  var tpm25 = image.select('Optical_Depth_047').multiply(0.001).multiply(23.89).add(5.18); //
  return image.addBands(tpm25, null, true);
}

//Raster Data Processing
var collection4 = dataset.map(transform_to_pm25);
var PM25_img = collection4.mosaic().clip(geometry_L);

//3. TimeSeries Chart & Analysis-----------
//Display a time-series chart
var chart_reg3 = ui.Chart.image.series({
  imageCollection: collection3,
  region: geometry_L,
  reducer: ee.Reducer.mean(),
  scale: 1000   //1000 m
}).setSeriesNames(['AOD 0.47']) //
.setOptions({
    lineWidth: 1,
    title: 'Laos Land Aerosol Optical Depth[AOI]',
    interpolateNulls: true,
    vAxis: {title: 'AOD 0.47 μm'},
    hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 10}},
    trendlines: {0: {color: 'CC0000'}
    },
});
print(chart_reg3);

//PM 2.5 chart
var chart_reg4 = ui.Chart.image.series({
  imageCollection: collection4,
  region: geometry_L,
  reducer: ee.Reducer.mean(),
  scale: 1000   //1000 m
}).setSeriesNames(['PM 2.5']) //
.setOptions({
    lineWidth: 1,
    title: 'Laos A Satellite-Based PM 2.5 [AOI]',
    interpolateNulls: true,
    vAxis: {title: 'PM 2.5'},
    hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 10}},
    trendlines: {0: {color: '00CC00'}
    },
});
print(chart_reg4);


// //4. Create User Interface --------------
// //Create a panel to hold our widgets.
// var panel_L = ui.Panel();
// panel_L.style().set('width', '500px');

// //Create an intro panel with labels.
// var intro = ui.Panel([
//   ui.Label({
//     value: 'Satellite-Based PM 2.5 Report Chart',
//     style: {fontSize: '20px', fontWeight: 'bold'}
//   }),
//   ui.Label('location')
//   ]);
//   panel_L.add(intro);
//   var layer = Map.layers();
  
// //panels to hold lon/lat values  
// var lon = ui.Label();
// var lat = ui.Label();
// panel_L.add(ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));

// //5. Registers a callback for onClick event 
// Map.onClick(function(coords){  //
  
// //Input inspector location
// var point = ee.Geometry.Point(coords.lon, coords.lat);

// if(layer.get(2) !== undefined){Map.remove(layer.get(2));} //

// //Zoom to location
// Map.centerObject(point,8);

// //Create point object 
// Map.addLayer(ee.FeatureCollection(point)
// .draw({color: 'black', pointRadius: 5, strokeWidth: 2}), {}, 'Observation Target');


// //Create Online Chartvar
// var ui_chart3 = ui.Chart.image.series({
//   imageCollection: collection3,
//   region: point, //
//   reducer: ee.Reducer.median(), //
//   scale: 1000
// }).setSeriesNames(['AOD 0.47']);

// ui_chart3.setOptions({
//     lineWidth: 1,
//     title: 'Laos Land Aerosol Optical Depth Daily',
//     interpolateNulls: true,
//     vAxis: {title: 'AOD 0.47'},
//     hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 20}},
//     trendlines: {0: {color: 'CC0000'},
//     }
// });

// var ui_chart4 = ui.Chart.image.series({
//   imageCollection: collection4,
//   region: point,
//   reducer: ee.Reducer.median(),
//   scale: 1000
// }).setSeriesNames(['PM 2.5']);

// ui_chart4.setOptions({
//     lineWidth: 1,
//     title: 'Laos A Satellite-Based PM 2.5 (estimation)',
//     interpolateNulls: true,
//     vAxis: {title: 'PM 2.5'},
//     hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 20}},
//     trendlines: {0: {color: '00CC00'},
//     },
// });

// //Add Chart Object to panel
// panel_L.widgets().set(2, ui_chart3);
// panel_L.widgets().set(3, ui_chart4);

// //Set Location Value
// lon.setValue('lon: ' + coords.lon.toFixed(3));
// lat.setValue('lat: ' + coords.lat.toFixed(3));
// });


//6. Show Result with Google Map API-----------
Map.style().set('cursor', 'crosshair');
Map.centerObject(geometry_L, 6);
Map.addLayer(AOD_img, AOD_vis, 'Laos Aerosol optical depth over land (0.47 μm)', false);
Map.addLayer(PM25_img, pm25_vis, 'Laos Satellite-Based PM2.5');

// //add the panel to the ui.root
// ui.root.insert(0, panel_L);

//------------------Myanmar-------------
//1. Data Management--------------
//Load Aoi Geometry
var datasetf = ee.FeatureCollection("FAO/GAUL_SIMPLIFIED_500m/2015/level0");
var geometry_M = datasetf.filter(ee.Filter.eq('ADM0_NAME', 'Myanmar'));//

//
Map.centerObject(geometry_M, 5)
Map.addLayer(geometry_M, {}, 'Myanmar')

//Data Parameter
var dStart = ee.Date('2024-04-01');
var dEnd = ee.Date(Date.now());

//Terra & Aqua MAIAC Land Aerosol Optical Depth Daily 1 km
var dataset = ee.ImageCollection("MODIS/061/MCD19A2_GRANULES") //
  .filterDate(dStart,dEnd)
  .filter(ee.Filter.bounds(geometry_M))
  .select('Optical_Depth_047') //
  ;
print(dataset);

//Applies scaling factors
function applyScaleFactors(image){  //
  var Depth_047Bands = image.select('Optical_Depth_047').multiply(0.001); //
  return image.addBands(Depth_047Bands, null, true);
}

//Raster Data Processing
var collection5 = dataset.map(applyScaleFactors); //map
var AOD_img = collection5.mosaic().clip(geometry_M); // 
//Map.addLayer(AOD_img, {}, 'AOD_img');


//2. Tranform Terra & Aqua MAIAC to PM 2.5-------------
// y = 23.89x + 5.18

//
function transform_to_pm25(image){
  var tpm25 = image.select('Optical_Depth_047').multiply(0.001).multiply(23.89).add(5.18); //
  return image.addBands(tpm25, null, true);
}

//Raster Data Processing
var collection6 = dataset.map(transform_to_pm25);
var PM25_img = collection6.mosaic().clip(geometry_M);

//3. TimeSeries Chart & Analysis-----------
//Display a time-series chart
var chart_reg5 = ui.Chart.image.series({
  imageCollection: collection5,
  region: geometry_M,
  reducer: ee.Reducer.mean(),
  scale: 1000   //1000 m
}).setSeriesNames(['AOD 0.47']) //
.setOptions({
    lineWidth: 1,
    title: 'Myanmar Land Aerosol Optical Depth[AOI]',
    interpolateNulls: true,
    vAxis: {title: 'AOD 0.47 μm'},
    hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 10}},
    trendlines: {0: {color: 'CC0000'}
    },
});
print(chart_reg5);

//PM 2.5 chart
var chart_reg6 = ui.Chart.image.series({
  imageCollection: collection6,
  region: geometry_M,
  reducer: ee.Reducer.mean(),
  scale: 1000   //1000 m
}).setSeriesNames(['PM 2.5']) //
.setOptions({
    lineWidth: 1,
    title: 'Myanmar A Satellite-Based PM 2.5 [AOI]',
    interpolateNulls: true,
    vAxis: {title: 'PM 2.5'},
    hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 10}},
    trendlines: {0: {color: '00CC00'}
    },
});
print(chart_reg6);

// 4. Create User Interface 
// //Create a panel to hold our widgets.
// var panel_M = ui.Panel();
// panel_M.style().set('width', '500px');

// //Create an intro panel with labels.
// var intro = ui.Panel([
//   ui.Label({
//     value: 'Satellite-Based PM 2.5 Report Chart',
//     style: {fontSize: '20px', fontWeight: 'bold'}
//   }),
//   ui.Label('location')
//   ]);
//   panel_M.add(intro);
//   var layer = Map.layers();
  
// //panels to hold lon/lat values  
// var lon = ui.Label();
// var lat = ui.Label();
// panel_M.add(ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));

// //5. Registers a callback for onClick event 
// Map.onClick(function(coords){  //
  
// //Input inspector location
// var point = ee.Geometry.Point(coords.lon, coords.lat);

// if(layer.get(2) !== undefined){Map.remove(layer.get(2));} //

// //Zoom to location
// Map.centerObject(point,8);

// //Create point object
// Map.addLayer(ee.FeatureCollection(point)
// .draw({color: 'black', pointRadius: 5, strokeWidth: 2}), {}, 'Observation Target');


// //Create Online Chartvar
// var ui_chart5 = ui.Chart.image.series({
//   imageCollection: collection5,
//   region: point, //
//   reducer: ee.Reducer.median(), //
//   scale: 1000
// }).setSeriesNames(['AOD 0.47']);

// ui_chart5.setOptions({
//     lineWidth: 1,
//     title: 'Myanmar Land Aerosol Optical Depth Daily',
//     interpolateNulls: true,
//     vAxis: {title: 'AOD 0.47'},
//     hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 20}},
//     trendlines: {0: {color: 'CC0000'},
//     }
// });

// var ui_chart6 = ui.Chart.image.series({
//   imageCollection: collection6,
//   region: point,
//   reducer: ee.Reducer.median(),
//   scale: 1000
// }).setSeriesNames(['PM 2.5']);

// ui_chart6.setOptions({
//     lineWidth: 1,
//     title: 'Myanmar A Satellite-Based PM 2.5 (estimation)',
//     interpolateNulls: true,
//     vAxis: {title: 'PM 2.5'},
//     hAxis: {title: 'Date', format: 'DD-MM-YY', gridlines: {count: 20}},
//     trendlines: {0: {color: '00CC00'},
//     },
// });

// //Add Chart Object to panel
// panel_L.widgets().set(2, ui_chart5);
// panel_L.widgets().set(3, ui_chart6);

// //Set Location Value
// lon.setValue('lon: ' + coords.lon.toFixed(3));
// lat.setValue('lat: ' + coords.lat.toFixed(3));
// });


//6. Show Result with Google Map API-----------
Map.style().set('cursor', 'crosshair');
Map.centerObject(geometry_M, 6);
Map.addLayer(AOD_img, AOD_vis, 'Myanmar Aerosol optical depth over land (0.47 μm)', false);
Map.addLayer(PM25_img, pm25_vis, 'Myanmar Satellite-Based PM2.5');

// //add the panel to the ui.root
// ui.root.insert(0, panel_M);
//--------------------------------------------//
