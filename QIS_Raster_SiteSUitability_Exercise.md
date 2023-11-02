## Wind Farm Exercise QGIS:

DATA:   [https://tinyurl.com/m2p9eep]([url](https://tinyurl.com/m2p9eep))

You will be creating a model with four inputs: Federally owned land, distance to major transmission lines, wind speed, and slope. The four inputs are included in the workshop folder.  Be sure to store any new layers in this directory as well.

## Slope:

We will look for areas on relatively flat ground.  Add uselevikm to QGIS:


This is a DEM of US elevation at a 1KM resolution (actually closer to 933 meters).  This is the resolution that you will use for all the rasters we generate today.

Many of the raster tools we will be using can be found in the Raster Analysis tools.  In the Processing toolbar, open the Slope tool


Use z factor=1.  Save output as USSlope:

Here is the output:

 
## Interpolation

We will also look for areas that have high wind speeds.  Add USWind.shp to the map.  This is a vector layer of points representing US cities, attribute information average includes wind speed for each month as well as annual averages.  You can play with the visualizations to explore the data.  Here I am using the layer properties to symbolize the annual averages: 


However, we need to convert this to a raster using an interpolation strategy.  We will use Inverse Distance Weighted as the interpolation method. Open IDW from the interpolation tools:

We will use ANN as the value field and make sure to use the extant slope raster as the setting for output Extent. We will also use the same cell size – 933.154420 (which you can find in the slope layer properties. Save output as USAnnWind:
 
The output:


## Distance analysis

Next you will measure distance to transmission lines.  Add USTransLines.shp:


This is a vector dataset of lines representing major electrical transmission lines in the US.  Because the distance raster tool in QGIS requires a raster input, we will convert ths to a raster layer using the “rasterize” tool.  Burn in a value of 1 (0 fo no data), use the same extent and raster units.  Save the output as “TransitLines”:


The output is a raster layer of the lines:


Open the Proximity tool:

Use “1” as the pixel value to be a target pixel.  Save the output as TransDist:

The Output:

Finally, we will use the availability of federally owned lands. Add USlands to QGIS:

This layer is already in the right format and resolution, so you do not need to do any analysis.  It is a raster representing federally owned lands.  The data has already been “classified” for use in this analysis.  We are only interested in areas federally owned and those areas have a value of “10.”  Non owned areas have a value of “1.”  In our site analysis, higher values will represent better suitability and we will use a 1-10 scale for each input.

The inputs for our model are now ready, but there is a problem – the values for all four inputs are in different scales and units.  We need to normalize these values so they can be averaged together.  In practice this is tricky to do and requires a good knowledge of your data and their distributions. In this case, I will give you the values to use.  For this exercise, we will rescale all of the data to a 1-10 scale. 

To do this we will use the “Reclassify by table” tool


Click on the reclassify table button:

And create a table using these values (these represent an approximate quiltile classification of the data):

Save the slope as slope_reclass.

The output:

Do the same process with the interpolated wind speeds using these values.  Save the wind speed as Wind_reclass:


Finally, reclassify the distance to transit lines. Again, reverse the values so that the higher values reflect the shorter distances.  Save as Dist_reclass:


## Weighted Measures

Finally we will take a weighted measurement of all of these datasets for out final site analysis.  In QGIS we will use the Raster Calculator for this:


Add the four reclass rasters and apply weights as you see appropriate.  Here, I have doubled the weight of both Slope_reclass and Wind_reclass because I suspect that the population and transmission lines datasets will be highly correlated.  Save the output as FinalValue:

In this case, output values range from 5 to 37:




