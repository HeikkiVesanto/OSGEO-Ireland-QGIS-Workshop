QGIS Workshop OSGeo Ireland 2018

These instructions are for QGIS 2.14. But any higher version of QGIS should work.

All steps will be run through during the workshop as a group.

Part 1.

Create a print composer:
Project> New Print Composer

Add in two maps one on the left and another on the right.

Adjust the left map, so that is shows all of Ireland. This will be an overview map.

Set a preset for the left map. Using the eye. And set the scalt to 2250000.

This right one will be zoomed automatically so its positioning is not as important.

Go back to the QGIS window.

Turn off the background mapping layers by unticking them in the layers panel, and turn on the Admin Counties layer.

Return back to the print composer.

Lets turn on the atlas generator.

From the "Atlas generation" tab tick the "Generate an atlas" tick box.

Select the Admin Counties layer from the drop down list.

Select the map on the right.

Choose the Item properties tab.

Tick the "Controlled by atlas" check box.

Turn on the atlas preview, from Atlas> Preview.

Success, but not much success yet.

Lets add in a copyright statement:
"Contains Ordnance Survey Ireland data © OSi 2012"

We have a few things we can do to improve the map. For example actually having some data.

Let's add some in.

From the folder:
\Data\CSO_Census_2016

Add drag and drop the "Small Areas" file that end in .shp into the QGIS window.

Also add in the "Vacant Housing" csv file.

One is the geometries for the census small areas, and one is the vacant property counts for the small areas. 

We will join these two together.

Open up the properties for the Small Areas layer. Right click on it in the Layers Panel and choose properties.

Choose the Joins tab.

Click on the green plus at the bottom left.

Choose:
Join layer: Vacant_Housing_Data_2016
Join field: SA_2017_GUID
Target field: GUID

Hit Ok. We can check if the join worked.

Right click on the "Small Areas" layer in the layer panel and choose "Open Attribute Table". If you scroll to the right, we can see a bunch of data.

This is data from the Census that shows how many houses are vacant in each small area. You can find a description of the columns in the VHD_Glossary_2016, found in the same folder as the original file.

Let's symbolise the joined data.

Open up the properties for the Small Areas layer by right clicking on it in the layers panel and choosing "Properties".

Choose the "Style" tab,

From the drop-down list in the top left choose "Graduated".

Now we can create a symbology from any of the columns in the data. And there is one one for total vacant properties. However a better option would be to look at the number vacant properties normalised with the number of households.

We can do this using an expression.

Click on the "E" button to the right of the Column drop down list. Enter the following expression:

( "Vacant_Housing_Data_2016_VT1_TOT_TOT" / "Total_HH" ) * 100

Vacant_Housing_Data_2016_VT1_TOT_TOT is the total count of vacant properties.
Total_HH is the total number of housholds.

From Mode choose "Quantile (Equal Count)"
Choose a Color ramp.
Change the number of classes to "10".
Click Ok.

Make sure the Admin Counties is visible on top of the Small Areas layer. If not you can drag it up in the layers panel.

Lets return to the Print Composer. It is probably still open behind the regular QGIS window.

If not you can re-open it from: Project> Print Composers> Composer 1

You can scroll through the admin councils using the arrow buttons.

Looking better.

However we have a couple more things we could do.

For example an overview on the map of Ireland, and a title with the area being highlighted.

Lets start with the overview.

Select the map on the left.

In the "Item properties" tab, scroll down to "Overviews".

Click the small triangle next to "Overviews"

Click on the geen plus button.

From the Map frame selector, choose "Map 1".

Lets add a title as well.

Add in a text box, with the following text:

Vacant properties in [% "ENGLISH" %]

The ENGLISH column conatins the English name of the local authority.

Cycle through a couple of maps to check if it is working.

Looking great, although it would be a bit clearer if only the council in question was visible.

The easiest way to do this is to create a new symbology that masks out the other councils.

Return to the main QGIS window.

Right click on the Admin Counties layer and choose "Properties".

From the drop down list in the top left choose "Inverted Polygons".

From the Sub renderer drop down choose "Rule-based".

Add a new rule by clicking the green plus sign in the bottom left.

Set the "Filter" to:
$id = @atlas_featureid 

Change the symbology so that the fill color is white and the border is a darker color.

Done

If you have time there are a couple of additions that make the map a bit cleaner.

Add a legend for the vacant properties map.
Change the case case of the council name in the title to title case.
Add a new layer to the backgroud map preset. This should be the "Admin Counties" dataset which shows a red outline in the overview map for the county shown in the atlas generator.

Part 2.

Add in the "counties" ShapeFile from the Data\Townlands_ie folder. Drag and drop the "counties.shp" file into QGIS.

Add in the AllIrelandSeniorFootballChampions.csv. Drag and drop the file from Data\Wikipedia into QGIS. 

In the last part we joined non-spatial data to spatial data. The other way around is a little bit trickier, but luckily QGIS 2.14 add a very hand feature, the Virtual Layer. It adds the ability to do SQL joins between any layers.

From:
Layer> Add Layer> Add Virtual Layer...

Click the "Import" button. 

Choose (you can select more than one by holding control while selecting a second layer, or do it in two Imports):
counties
and
AllIrelandSeniorFootballChampions

The query should be:

select win.Winner, win.Year, counties.geometry, win.Year_Date, win.Total_Wins, win.Centroid_x, win.Centroid_y
from AllIrelandSeniorFootballChampions win
left join counties on counties.NAME_TAG = win.Winner

Ok to add.

Lets symbolise the layer.

We want one symbology for the winner being shown that year and one for previous winners.

From the drop down list in the top left choose "Rule-based".

The first rule should be:
"Year" = attribute(  @atlas_feature , 'Year')

This will be the symbology for the currrent winner. Set a suitable symbology.

Add a new rule with the green plus sign at the bottom left, with the rule:
to_int( "Year" ) <   to_int( attribute(  @atlas_feature , 'Year'))

Create a new print composer. Through Project> New Print Composer 

From the Composition tab set the Orientation to "Portrait"

Add in a map and set the scale to 1800000 and align the map.

From the Atlas Generation tab set the "Coverage layer" to "AllIrelandSeniorFootballChampions".

Add in a copyright statement:
"© OpenStreetMap contributors"

Turn on the atlas preview, from Atlas> Preview.

Check to see if it is working by cycling through a few years.

We can also add a few more features.

Like a title:
All Ireland Senior Football Champion

And a lable naming the winners.
Total wins: [% "Total_Wins" %]
[% "Winner" %] - [% "Year" %]

Export out as png images.

Creating the gif.

sudo apt-get update
sudo apt-get install imagemagick

convert *.png -layers OptimizePlus -loop 0 -set delay 80 gaa_winners.gif

If you have time:

Centroid of winners:
New rule:
"Year" = attribute(  @atlas_feature , 'Year')

Geometry generator, point output:
make_point( "Centroid_x" , "Centroid_y")

Centre of champions legend.

Label of number of wins:
Add a point geometry generator to existing symbology:

case when
 num_geometries( $geometry) = 1
 then  point_on_surface( $geometry)
else centroid( $geometry)
end

Marker - Font marker
Total_wins as symbol.

Texture in background:

Composition, Page settings, Page background, Raster image fill.
\Created\noise.png
