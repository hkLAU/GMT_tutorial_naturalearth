# convert SQLite vector package into .gmt vectors with GDAL and GRASS GIS

This will be a short demonstration of the code I use for converting Natural Earth SQLite package into .gmt format for plotting in GMT. The version of GRASS GIS that use in this demonstration is grass78. 

**Introduction:** 
[Natural Earth](https://www.naturalearthdata.com) is a public domain map dataset provides a wide range of physical, cultural and raster features for users to create well-crafted maps with cartography and GIS tools. Physical vector features, such as coastline, land, islands, reefs, rivers, lakes, and cultural vector features, such as countries, boundaries, roads, urban areas, parks and protected lands are available at 1:10m, 1:50m and 1:110m scales in the format of SHP, SQLite, or GeoPackage. 

[GDAL](https://gdal.org) is a translator library for raster and vector geospatial data. [GRASS GIS](https://grass.osgeo.org) is a free and open source software for geographic resources analysis. Documentation and installation guide can be found in [the GRASS Wiki page](https://grasswiki.osgeo.org/wiki/GRASS-Wiki). 

## 1.	Convert discrete and simple geographic features (e.g. rivers, roads, coastlines, boundaries, lakes) into GMT format vectors with GDAL

Assumed that we have downloaded a SQLite package from the Natural earth data [download page](https://www.naturalearthdata.com/downloads/ ), we can view the data info by typing: 
```
ogrinfo packages/natural_earth_vector.sqlite 
```
To convert simple features data for example rivers and lakes between file formats, we can use the command *ogr2ogr*. 
```
ogr2ogr -f OGR_GMT ne_110m_rivers_lake_centerlines.gmt packages/natural_earth_vector.sqlite ne_110m_rivers_lake_centerlines
```
Where OGR_GMT is the desire format, ne_110m_rivers_lake_centerlines.gmt is the outfile name, packages/natural_earth_vector.sqlite is the infile name and ne_110m_rivers_lake_centerlines is the  layer name. 

## 2.	Convert continues and complex geographic features (e.g. ice sheets, ocean) into GMT format vectors with GRASS GIS

Open GRASS GIS.
```
grass78
```
Import vector data into a GRASS vector map with the command *v.in.ogr*. 
```
v.in.ogr input=packages/natural_earth_vector.sqlite layer=ne_50m_ocean output=ne_50m_ocean
```
Set region of interest. You can view region info with *g.region -p*. 
```
g.region e=180e w=180w n=90n s=90s
```
Creates a vector map of a user-defined grid. We would like to create a 6x12 grid (30 arc-min or 0.5 degree resulotion) and name it *grid30min*. For details of making grid, see [manual](https://grass.osgeo.org/grass80/manuals/v.mkgrid.html). 
```
v.mkgrid map=grid30min grid=6,12
```
View the vector map you have just made. 
```
g.list type=vect
```
Overlays two vector maps and clip the ocean ploygon within the 30 arc-min grid vector with the ***and*** operator. 
```
v.overlay ainput=ne_50m_ocean binput=grid30deg operator=and output=ne_50m_ocean_blocks
```
Export the output vector in GMT format. 
```
v.out.ogr input=ne_50m_ocean_blocks output=ne_50m_ocean_blocks.gmt format=OGR_GMT
```
Exit programm.

From my expereience, if GMT can not plot the polygon properly, it may because it doesn't like the points in "90" degree (the real reason is unknown to me). Try to open .gmt file in text editor and replace "90.0" to "89.999" or avioding plotting "90" in your GMT script. 


