==============================================
FoCIS (Forecasting to Combat Invasive Species)
==============================================
Date Created: August 8, 2019
==============================================
Authors: New York Ecological Forecasting II
	 NASA DEVELOP Summer 2019
	 Rya Inman (Project Lead),
	 Abigail Barenblitt
	 Ryan Hammock
	 Niharika 'Chitra' Kokkirala
=================================================================================================
Eastern Hemlock Distribution Model:
Four codes scripted to run in Google Earth Engine to compute predicted habitat distribution of 
Eastern Hemlock in (1) Adirondack Park and (2) New York State.
=================================================================================================
Primary satellite data used - only one per model:
(1) Landsat 8 OLI
(2) Sentinel-2 MSI 
=================
Ancillary data:
NDVI (from Landsat 8 OLI or Sentinel-2 MSI), 
NLCD (for Deciduous, Evergreen, and Mixed forest cover),
New York Linear Hydrography (as euclidean distance to streams layer), 
Elevation (from SRTM),
Aspect - north and northwest (from SRTM), 
Soil moisture profile (SMAP), 
Soil Acidity (USDA SSURGO), 
Ground-truthed training data from Adirondack Park Invasive Plant Program (APIPP) and New York Natural Heritage Program (NHP) -- total 286 points. 
==================
This code is scripted to produce a classified map of hemlock dominance in 4 categories: 
0 = No Hemlock, 1 = Hemlock Sub Dominant, 2 = Hemlock Dominant, 3 = Pure Hemlock
=================================================================================================

Recommended Packages:
* "palettes": 'users/gena/packages:palettes' *Used for visualization only*
=================================================================================================

Parameters for hemlock distribution model:
------------------------------------------
This code imports least-cloudy mosaicked imagery from Landsat 8 OLI or Sentinel-2 MSI, 
plus NLCD (forest cover), New York Linear Hydrography (stream distance), elevation and aspect from SRTM DEM, SMAP soil moisture profile, SSURGO soil acidity, plus
ground-truthed training data of hemlock presence from Adirondack Park Invasive Plant Program and New York Natural Heritage Program, 
and the boundary shapefile of either (1) Adirondack Park or (2) New York State shoreline.

Imported from Google Earth Engine data catalog: 
	// l8_SR: USGS Landsat 8 Surface Reflectance Tier 1 (LANDSAT/LC08/C01/T1_SR) 
	   (or S2: ImageCollection "Sentinel-2 MSI: MultiSpectral Imstrument, Level-1C")
	// SRTM: SRTM Digital Elevation Data 30m (USGS/SRTMGL1_003)
	// SMAP: NASA-USDA SMAP Global Soil Moisture Data (NASA_USDA/HSL/SMAP_soil_moisture)
	// NLCD: USGS National Land Cover Database (USGS/NLCD)

Outside Datasets:
	// acidSoils: Soil pH levels as derived from NRCS SSURGO data
	// distStream: Euclidean distance to stream at 30 m resolution from New York State linear hydrography
	// apipp: Boundary of Adirondack Park/APIPP
	// nys: New York State shoreline (from NYS GIS Clearinghouse)

Datasets from APIPP and NY NHP:
	// allTrains: Point data used to train the Random Forest Model. These points indicate ground-truthed 
		locations of hemlock in Adirondack Park in 4 categories:
		0 = No Hemlock, 1 = Hemlock Sub Dominant, 2 = Hemlock Dominant, 3 = Pure Hemlock
	// allValids: Point data used to validate the Random Forest Model. These points indicate ground-truthed 
		locations of hemlock in Adirondack Park in 4 categories:
		0 = No Hemlock, 1 = Hemlock Sub Dominant, 2 = Hemlock Dominant, 3 = Pure Hemlock

Model Steps:
------------
1) Import all parameters listed above -- shapefiles as "Table" assets and GEE datasets from API search bar and import feature
2) Acquire leaf off and leaf on least cloudy mosaics
	2a) Filter dates of interest for leaf off and leaf on periods
	2b) Merge all years of leaf off and leaf on periods
	2c) Mask out cloudy images
3) Add additional bands of interest to leaf on mosaic
	3a) Calculate NDVI from Red and NIR bands and create NDVI band
	3b) Create Distance to Stream band from "distStream" image
	3c) Add both bands to leaf on mosaic
4) Begin constructing Random Forest model
	4a) Add all bands of interest to a single "predictors" image -- blue, green, red, NIR satellite bands plus distance to streams band
	4b) Create training points by sampling "predictors" at points included in "allTrains"
5) Create masks for parameters of interest
	5a) Clip/ mask all parameters of interest to restrictions determined based on the habitat requirements
		of eastern hemlock
6) Train Random Forest model 
	6a) Create list of bands to include in classifier
	6b) Specify constraints of Random Forest model and train using training data
	6c) Classify the Random Forest model to determine predicted hemlock distribution
7) Get a confusion matrix representing resubstitution accuracy
	7a) Resubstitute training data into Random Forest model to determine fit of model to original training
		points
8) Mask for parameters of interest
	7a) Add masks to Random Forest model results to explude areas without the habitat requirments for hemlock
9) Visualize and export results

*End of hemlock distribution modeling component*
================================================================================================================
Forecasting for HWA Spread and Eastern Hemlock Infestation/Die Off Through 2049:
Two codes scripted with methods for predicting HWA spread and subsequently the remaining unaffected hemlock
into future years. We chose 2049 to allow for multiple cycles of HWA spread and hemlock die off, but the model
can be extended into the future indefinitely (currently NASA temperature projections extend to 2099).
----------------------------------------------------------------------------------------------------------------

Random Forest Method:
=====================
Primary satellite data used:
(1) Landsat 8 OLI
* can also be run using Sentinel-2 MSI
======================================
Ancillary data:
NEX-DCP30 NASA downscaled climate projections - 2016-2019 to train, and 2046-2049 to classify
NDVI (from Landsat 8 OLI or Sentinel-2 MSI), 
NLCD (for Deciduous, Evergreen, and Mixed forest cover),
New York Linear Hydrography (as euclidean distance to streams layer), 
Elevation (from SRTM),
Aspect - north and northwest (from SRTM), 
Soil moisture profile (SMAP), 
Soil Acidity (USDA SSURGO), 
Ground-truthed training data from Adirondack Park Invasive Plant Program (APIPP) and New York Natural Heritage Program (NHP) -- total 286 points. 
==================================
Model Steps:
------------
1) Start with L8_NYS RF model script, and add NEX-DCP30 temperature data from 2016-2019 into layered image
   on which the RF classifier is trained, using HWA presence data instead of hemlock presence data.
2) Add NEX-DCP30 data from 2046-2049 into new layered image on which this trained classifier is run. Output is 
   predicted HWA extent in 2049 given these future projected temperature conditions.

================================================================================================================

Overlay Method:
===============
Primary datasets used:
======================
(1) Hemlock distribution from present day
(2) NEX-DCP30 temperature projections to 2049
(3) HWA presence data from iMap and BISON, raster layer of buffer for spread to 2049
====================================================================================
Model Steps:
1) Load in these three data sets, plus NY state boundary shapefile
2) Mask HWA spread layer for temperature constraints, as minimum temp. across time period, for -20 deg. C
3) Additionally, mask this layer with hemlock distribution from present day
Output: final map of where HWA can spread to by 2049, given climate projections to that time

*End of Forecasting Code*
================================================================================================================
---------
 Contact
---------
Name: Rya Inman
E-mail: ryainman16@gmail.com


***
***