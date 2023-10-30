## Exercise

In this exercise you will use QGIS to create a map showing the change in housing units in New 
York City neighborhoods since 2010. 

The housing data is available publically from the NYC Data Clearinghouse [NYCOpenData ](https://opendata.cityofnewyork.us)

![NYCopendata]

Specifically, this data is provided by the NYC Department of City Planning from their housing 
database. You can access the data at the census tract level from [this link](https://data.cityofnewyork.us/Housing-Development/Housing-Database-by-2020-Census-Tract/nahe-je7c)

![housingdb]

The table from this page provides the net number of housing units annually for every census tract for every year from 210 to 2022. We can use QGIS to manipulate and map this data for the exercise. However, there are a few problems with this table. First, there is no functional geography for the tracts – the boundaries are not defined. You will have to find those geographies elsewhere (we will come back to this shortly). Second, the data download (as of the writing of this exercise) is not formatted and cleaned appropriately. Much of the needed cleaning is out of the scope of this exercise, so we have provided you with a modified and cleaned version of the data in the exercise files. It is the table named HousingUnitsTracts2010_22.csv:

![unitscsv]

The data is composed of the Census tract FIPS code, which is a unique number identifying the tract, in the first column and the net totals of housing units in that trach for each year from 2010 to 2022. If you sum the totals of these 13 columns you will get the net total housing unit change for the tome period (you will do this later in QGIS).
The final problem with the table is that it is not at the level of geography that we want for this exercise. The data is provided at the census tract level. Census tracts are a neighborhood definition created by the US Census department. Census tracts typically have around 3000-5000 people in them and in densely populated areas they can be quite small. For example, here is a map of the tracts in the Columbia University area:

![tracts]

For this exercise we will use a neighborhood definition called “Neighborhood Tabulation Areas” (NTAs). NTAs are created by the NYC Dept. Of Planning and are aggregations of census tracts that are meant to approximate neighborhoods as they are popularly thought of for city residents. NTAs typically have around 40,000-50,000 residents. For example, here is the “Morningside Heights” NTA that is composed of 8 census tracts:

![NTA]

You will use QGIS to create NTA geographic definitions and housing unit totals for the entire city.
As previously stated, census tracts are determined by the Census Department and they produce authoritative boundaries for them on a regular basis. There are also a variety of sources of derivative boundary definitions for these tracts. You will use a modified tract file for! NYC that can be found at the Geodata@columbia home page and download at [this link](https://geodata.library.columbia.edu/catalog/sde-columbia-cul_nyc_tracts_2020)):

![geodatatracts]

If you haven’t already, create a directory to store all the files for this exercise. Store the csv table and the geography download in that directory. Unzip the track geography files from Geodata. This file is in an open source format called “geopackage” and has a gpkg extension. Geopackages are a fast and convenient geospatial format that come in a single file. A more common file format for vector based GIS data is the “shapefile.” If you are interested in comparing them, you can download a shapefile version of this layer from NYC Planning [here](https://www.nyc.gov/site/planning/data-maps/open-data/census-download-metadata.page).

With all the data ready, open QGIS, and create an empty map document.
One way to add the geopackage to QGIS is through the browser panel. The browser panel is likely to be docked to the left side of QGIS. From here you can (among other things) browse your computers file directory. If you expand the directory structure to your working directory, you can simply click on the geopackage and drag it into the map window. If you do that the tracts will appear in the map window:

![geopackage]

Note that the tracts appear in a random color. You can change this, but we will do that later.

Next inspect the layer attributes by right clicking on the layer name in the layer panel and choosing “open attribute table”:

![openAtt]

Inspect it:

![AttributeTable]

There is a variety of information for each tract. For our purposes the three important columns are GEOID, NTA2020 and NTAName. GEOID is the 11 digit “FIPS” identifier that uniquely identifies the tract. NTA2020 is the unique identifier that represents the NTA that the tract is in – note that these are NOT unique as NTAs are comprised of multiple tracts. NTAName is the vernacular name that identifies the NTA.

Next, you will add the housing unit data.

CSV files may not add properly from the browser panel so instead, use the data source manager ![DSMButton] and choose the “Delimited text” option. Specify that the table is a CSV and choose “no geography.” The dialog will provide a preview of the data at the bottom:

![dsmWIndow]

One change needs to be made here. QGIS guesses the data type for each variable. In this case it will likely guess “integer” since the table only includes numeric characters with no decimals. However, this is not appropriate for the first field “centract2020,” which is the same FIPS code as used in the GEOID field in the geopackage. While this is a numeric identifier, it is not a quantitative variable. Instead, it is more akin to a name. In the geopackage it is encoded as a text or string variable, which is appropriate, and it will prove necessary in the next step for both tables to use the same datatype for this variable. Thus change that variable type to “Text (string)” through the drop down menu:

![string]

Click “add” and inspect the attribute table to make sure it looks correct:

![csvattributes]

Since the centract2020 and GEOID fields are unique identifiers, we can join the csv data to our geopackage using a table join.

Open the layer properties for the tract geographies by right clicking in the layers panel:

![properties]

In the properties dialog select the Joins tab:

![join]

Click the add join button ![addjoin]. Choose the csv as the join layer, select the appropriate join field and make sure the “custom field name prefix” field is blank:

![addjoinbox]

Click OK. Open the geopackage attribute table again and see that the data joined into the table:

![AttributeTablejoin]

Since the table join is stored in memory, it is recommended that you make the join permanent by exporting the geographies to a new layer. Right click on the track layer entry and choose export>Save features as:

![exportlayer]

Save the file as a geopackage to your working directory with the name TractsWIthHousingData.gpkg make sure the CRS is EPSG:2263:

![exportdialog]

The new layer should appear in the map window. At this point, we no longer need the original geopackage or csv. You can remove them by right-clicking in the layers panel and choosing “Remove Layer”
The next step is to generate the NTA boundaries by using information in the attribute table. You can construct the NTAs by merging the geographies of all the tracts for the same NTA. The GIS term for this transformation is “Dissolve”.
There are several dissolve functions available in QGIS. The one you will use is called “Aggregate.” To find it open the toolbox under the processing menu:

![openprocessing]

The processing panel will open, search dissolve in the search bar. You should get several results, choose “aggregate”:

![Searchdissolve]

The aggregate tool will allow you to make arithmetic calculations on the variables for the merged tracts. Important as we will want to sum the housing unit fields. In the Aggregate dialog, choose NTA2020 as the group field. In the aggregate controls choose “first” for the NTA2020 fields and the NTAName fields. More importantly, choose “sum” for all of the housing unit total fields (comp20**). You don’t need to
keep the other fields and they may be deleted using the delete field button ![deletefields]. Save the output file to the working directory with an appropriate file name:

![aggregate]

The output layer should appear in the map window. The boundaries will now represent the NTAs :

![NTAs]

And the attribute table will have the net unit change for each year:

![attrinuteNTAs]

Next, you will calculate the total net housing unit change for the 12-year period. You can do this using the Field calculator tool. Click the field calculator button ![fieldcalculator] in the attribute table. In the dialog you will want to “create a new field” and make it an integer field named “TotalUnits”. You can use the expression window to populate the new field with the sum of the 12 annual columns. You can access the fields by expanding the “fields and values” menu in the center window. Sum them all together using the + operator as seen here:

![calculation]

Click OK. Inspect the results in the attribute table:

![totalunits]

When using the field calculator QGIS automatically starts an editing session for the layer. End the editing session by depressing the edit button ![stopEdits].

Now you can make a “choropleth” map of the housing unit totals.
Remove the layer for the tracts.
Open the layer properties for the NTA layers. Choose the symbology properties. Choose a “graduated” color scheme and select TotalUnits as the value:

![symbology]

Now you can select the classification scheme and color palette for the map:

![symbologyclass]

Choose “classify” and OK to inspect the results:

![map]

Save the project with the name “NTA_Housingchange” in the directory when you are satisfied with your output.

![savemap]

Click on “New Print Layout” ![layout] to create a map. Enter a Title:

![housingUnitCHange]

Click on OK and the layout window will pop out.
Click on “add a new map” ![newlayout] and draw a frame filling the page.  And the image will appear in the frame you draw.

![Composer]

Insert a title, north arrow, scale bar, and legend.

![map2]

Finally, add a text box to the lower portion of your map indicating your data sources, your name and date.

Export your map to pdf by clicking “export as pdf” ![pdf]



[NYCopendata]: IntroPics/NYCopendata.png
[housingdb]: IntroPics/housingdb.png
[unitscsv]: IntroPics/unitscsv.png
[tracts]: IntroPics/tract[csvattributes]s.png
[NTA]: IntroPics/NTA.png
[geodatatracts]: IntroPics/geodatatracts.png
[geopackage]: IntroPics/geopackage.png
[openAtt]: IntroPics/openAtt.png
[AttributeTable]: IntroPics/AttributeTable.png
[DSMButton]: IntroPics/DSMButton.png
[dsmWIndow]: IntroPics/dsmWIndow.png
[string]: IntroPics/string.png
[csvattributes]: IntroPics/csvattributes.png
[properties]: IntroPics/properties.png
[join]: IntroPics/join.png
[addjoin]: IntroPics/addjoin.png
[addjoinbox]: IntroPics/addjoinbox.png
[AttributeTablejoin]: IntroPics/AttributeTablejoin.png
[exportlayer]: IntroPics/exportlayer.png
[exportdialog]: IntroPics/exportdialog.png
[openprocessing]: IntroPics/openprocessing.png
[aggregate]: IntroPics/aggregate.png
[deletefields]: IntroPics/deletefields.png
[Searchdissolve]: IntroPics/Searchdissolve.png
[NTAs]: IntroPics/NTAs.png
[attrinuteNTAs]: IntroPics/attrinuteNTAs.png
[fieldcalculator]: IntroPics/fieldcalculator.png
[calculation]: IntroPics/calculation.png
[totalunits]: IntroPics/totalunits.png
[stopEdits]: IntroPics/stopEdits.png
[symbology]: IntroPics/symbology.png
[symbologyclass]: IntroPics/symbologyclass.png
[map]: IntroPics/map.png
[savemap]: IntroPics/savemap.jpg
[layout]: IntroPics/layout.jpg
[housingUnitCHange]: IntroPics/housingUnitCHange.png
[newlayout]: IntroPics/newlayout.jpg
[Composer]: IntroPics/Composer.png
[map2]: IntroPics/map2.png
[pdf]: IntroPics/pdf.jpg
