
## <font color="Cornflowerblue"> Occurrence Data for M. micrantha(from www.gbif.org)</font>

Import libraries


```python
#Required packages
import rpy2.robjects as ro
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import geopandas as gpd
from shapely.geometry import Point, Polygon, MultiPolygon
from fiona.crs import from_epsg
from shapely.ops import cascaded_union
```

<p style='text-align: justify;'>The occurrence data would be downloaded from the Global Biodiversity Information Facility (www.gbif.org). We basically require the longitude and latitudue of the locations where the plant was found. R provides inbuilt functions that allows the user to access the database and extract the information. We extracted data by calling R package using library <code><font color='Darkpink'>rpy2</code></font>within the Python environment. </p>

<p style='text-align: justify;'>The module <code><font color='green'>robjects</font></code> of library <code><font color='Darkpink'>rpy2</font></code> help us to use the same code as we use in R language inside the   <code><font color='green'> ro.r(' ')</font></code>. For data processing we need to install two packages from R, namely, <i>dismo</i> and <i>jsonlite</i>. We need <i>jsonlite</i> because data contains geographical coordinates. It does mapping between the R objects and JSON data.</p>


```python
ro.r('install.packages("dismo",dependencies = TRUE)')
ro.r('install.packages("jsonlite", dependencies = TRUE)')
```


```python
library = ro.r('library(dismo)') 
```


```python
downld_data = ro.r('mic_gbif_all = gbif(genus = "Mikania", geo = FALSE)') # Extracting Species with genus name Mikania
```


```python
Na_removd =ro.r('mic_gbif_all = subset(mic_gbif_all, !is.na(lon) & !is.na(lat) & !is.na(species))')# Removing the 
#missing rows for longitude , latitude
```


```python
var_name =ro.r('names(mic_gbif_all)') # variable names
```


```python
mic_gbif =ro.r('mic_gbif_all = mic_gbif_all[,c(" species "," lon "," lat ")] ')# Data contains only name , lon and lat
```


```python
mic_gbif.to_csv(r"C:\Users\Jyoti\file name\mikania_1.csv", header = True) # saving data into csv file.
```


```python
mic_gbif = pd.read_csv('mikania_1.csv') # read csv file
```


```python
unique_species=np.unique(mic_gbif.species) # array of unique name of species
unique_species[:10] # first 10 name of species in the array.
```




    array(['Kanimia strobiliferaa', 'Mikania acuminata', 'Mikania acutissima',
           'Mikania additicia', 'Mikania alba', 'Mikania allartii',
           'Mikania alvimii', 'Mikania amblyolepis', 'Mikania amorimii',
           'Mikania andrei'], dtype=object)




```python
unique = mic_gbif.nunique() # show total no.of unique species names
unique [1]
```




    426




```python
mic_gbif = mic_gbif.loc[mic_gbif['species']=="Mikania micrantha"] # subset of species
```


```python
np.unique(mic_gbif.species) # checking the species extracted are correct .
```




    array(['Mikania micrantha'], dtype=object)




```python
mic_gbif.head () # gives first five row
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>species</th>
      <th>lon</th>
      <th>lat</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Mikania micrantha</td>
      <td>111.835979</td>
      <td>2.290731</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Mikania micrantha</td>
      <td>57.431867</td>
      <td>-20.208804</td>
    </tr>
    <tr>
      <th>26</th>
      <td>27</td>
      <td>Mikania micrantha</td>
      <td>103.867744</td>
      <td>1.355379</td>
    </tr>
    <tr>
      <th>70</th>
      <td>71</td>
      <td>Mikania micrantha</td>
      <td>120.483833</td>
      <td>23.471520</td>
    </tr>
    <tr>
      <th>74</th>
      <td>75</td>
      <td>Mikania micrantha</td>
      <td>120.519792</td>
      <td>22.663100</td>
    </tr>
  </tbody>
</table>
</div>




```python
mic_gbif.lon.values
```




    array([111.835979,  57.431867, 103.867744, ..., -57.1     , -53.88    ,
           -56.73    ])



#### <font color="Cornflowerblue"> Converting Dataframe into a geological dataframe.</font>


```python
geometry =[Point(xy) for xy in zip(mic_gbif.lon.values,mic_gbif.lat.values)]
crs = {'init':'epsg :4326'}
gdf = gpd.GeoDataFrame(mic_gbif, crs=crs, geometry = geometry)
gdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>species</th>
      <th>lon</th>
      <th>lat</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Mikania micrantha</td>
      <td>111.835979</td>
      <td>2.290731</td>
      <td>POINT (111.835979 2.290731)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Mikania micrantha</td>
      <td>57.431867</td>
      <td>-20.208804</td>
      <td>POINT (57.431867 -20.208804)</td>
    </tr>
    <tr>
      <th>26</th>
      <td>27</td>
      <td>Mikania micrantha</td>
      <td>103.867744</td>
      <td>1.355379</td>
      <td>POINT (103.867744 1.355379)</td>
    </tr>
    <tr>
      <th>70</th>
      <td>71</td>
      <td>Mikania micrantha</td>
      <td>120.483833</td>
      <td>23.471520</td>
      <td>POINT (120.483833 23.47152)</td>
    </tr>
    <tr>
      <th>74</th>
      <td>75</td>
      <td>Mikania micrantha</td>
      <td>120.519792</td>
      <td>22.663100</td>
      <td>POINT (120.519792 22.6631)</td>
    </tr>
  </tbody>
</table>
</div>



World map data has been retrived from the site - https://github.com/nasa/World-Wind-Java. Data is in the shape file with extension <i>.shp</i>. The shape file contains a column of geometry as polygons, area of regions and so on.


```python
wrld_simpl = gpd.read_file('TM_WORLD_BORDERS_SIMPL-0.2.shp')
wrld_simpl.crs # gives coordinate reference system of data
```




    {'init': 'epsg:4326'}



Since the shapefile contains geometry. We can visualize the data on the world map.


```python
wrld_simpl.plot(figsize=(12,8), edgecolor ='red', color='pink')
plt.title('World Map', fontsize =12)
plt.xlabel('Longitude', size =12)
plt.ylabel('Latitude', size =12)
plt.savefig('world map')
```


![png](Page_1_files/Page_1_24_0.png)


In the above plot, edges of polygons are in red color and region inside the edges are in pink color while white region signifies the ocean in the world map.

<p style='text-align: justify;'>And our focus is only the certain region not the entire world. The regions are a continent SouthAmerica and a country India. With the view on the map you can observe the extent values of longitude and latitude of SouthAmerica and India. In next code, I have displayed the SouthAmerica region just with the view on axis of previous map and highlighting the region with transparent blue square box.</p>

<p style='text-align: justify;'>In the following, we represent the regions by transparent square. They are nothing but the approximate extent of SouthAmerica and India. The extent has been created here with visual inspection by looking into the world map. However, extent can be extracted from the data points of occurrence species located in SouthAmercia. Extent of regions will be extracted in further code with the explanations.</p>


```python
polys1 = gpd.GeoSeries([Polygon([(-120,-60),(-120,30),(-30,30),(-30,-60)])])
# creating transparent square box for representing SouthAmerica in world map.
base = wrld_simpl.plot(edgecolor ='black',color='pink', figsize =(12,8))
gdf.plot (ax=base , marker ='o', color ='red', markersize =5)
polys1.plot (ax=base , color ='blue', alpha =0.2)
plt.title('World Map', fontsize =12)
plt.xlabel('Longitude', size =12)
plt.ylabel('Latitude', size =12)
```




    Text(75.875, 0.5, 'Latitude')




![png](Page_1_files/Page_1_28_1.png)


#### <font color="Cornflowerblue">Extracting datasets of continent South America</font>

Occurrences from SouthAmerica have been extracted using the approximated extent values with visual inspection of the region SouthAmerica.


```python
mic_gbif_america=gdf[(gdf.lon<-34)&(gdf.lon>-130)]
mic_gbif_america.head() # subset data of South America
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>species</th>
      <th>lon</th>
      <th>lat</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>157</th>
      <td>158</td>
      <td>Mikania micrantha</td>
      <td>-40.609514</td>
      <td>-19.990989</td>
      <td>POINT (-40.609514 -19.990989)</td>
    </tr>
    <tr>
      <th>414</th>
      <td>415</td>
      <td>Mikania micrantha</td>
      <td>-99.995244</td>
      <td>21.904365</td>
      <td>POINT (-99.995244 21.904365)</td>
    </tr>
    <tr>
      <th>460</th>
      <td>461</td>
      <td>Mikania micrantha</td>
      <td>-52.485556</td>
      <td>-30.324722</td>
      <td>POINT (-52.485556 -30.324722)</td>
    </tr>
    <tr>
      <th>484</th>
      <td>485</td>
      <td>Mikania micrantha</td>
      <td>-53.030722</td>
      <td>-24.068778</td>
      <td>POINT (-53.030722 -24.068778)</td>
    </tr>
    <tr>
      <th>500</th>
      <td>501</td>
      <td>Mikania micrantha</td>
      <td>-48.239353</td>
      <td>-2.672933</td>
      <td>POINT (-48.239353 -2.672933)</td>
    </tr>
  </tbody>
</table>
</div>




```python
base=wrld_simpl.plot(color='pink', edgecolor='black', figsize=(9,5))
mic_gbif_america.plot(ax=base)
base.set_xlim(-150,-30)
base.set_ylim(-60,50)
plt.title('South America',fontsize =12)
plt.xlabel('Longitude',size =12)
plt.ylabel('Latitude',size =12)

```




    Text(48.875, 0.5, 'Latitude')




![png](Page_1_files/Page_1_32_1.png)


The extracted data is saved into shape file by specifying the crs(coordinate reference system) to the geological dataframe for the future use.


```python
mic_gbif_america.crs = from_epsg(4326) 
mic_gbif_america.crs # displays coordinate reference system
```




    {'init': 'epsg:4326', 'no_defs': True}




```python
outfp=r"C:\Users\Prajapati Vivek\Desktop\spatial project\mikania\html file\mic_gbif_america.shp"
mic_gbif_america.to_file(outfp)
```


```python
mic_gbif_america=gpd.read_file('mic_gbif_america.shp')
mic_gbif_america.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0xe124023a20>




![png](Page_1_files/Page_1_36_1.png)


#### <font color="Cornflowerblue"> Same way we go for India</font>


```python
polys2 = gpd.GeoSeries([Polygon([(65,5),(65,45),(100,45),(100,5)])])
base=wrld_simpl.plot(color='pink', edgecolor='black', figsize=(9,5))
gdf.plot(ax=base, marker='o', color='red', markersize=5)
polys2.plot(ax=base, color='blue', alpha=0.2)
plt.title('India Map', fontsize =12)
plt.xlabel('Longitude', size =12)
plt.ylabel('Latitude', size =12)
```




    Text(48.875, 0.5, 'Latitude')




![png](Page_1_files/Page_1_38_1.png)



```python
mic_gbif_India=gdf[(gdf.lon<96) & (gdf.lon>58)]
mic_gbif_India.head() # subset data of India
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>species</th>
      <th>lon</th>
      <th>lat</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>492</th>
      <td>493</td>
      <td>Mikania micrantha</td>
      <td>90.532148</td>
      <td>27.189478</td>
      <td>POINT (90.53214799999999 27.189478)</td>
    </tr>
    <tr>
      <th>1089</th>
      <td>1090</td>
      <td>Mikania micrantha</td>
      <td>92.390000</td>
      <td>23.440000</td>
      <td>POINT (92.39 23.44)</td>
    </tr>
    <tr>
      <th>1339</th>
      <td>1340</td>
      <td>Mikania micrantha</td>
      <td>92.390000</td>
      <td>23.440000</td>
      <td>POINT (92.39 23.44)</td>
    </tr>
    <tr>
      <th>1342</th>
      <td>1343</td>
      <td>Mikania micrantha</td>
      <td>92.390000</td>
      <td>23.440000</td>
      <td>POINT (92.39 23.44)</td>
    </tr>
    <tr>
      <th>6677</th>
      <td>6678</td>
      <td>Mikania micrantha</td>
      <td>84.350833</td>
      <td>27.550556</td>
      <td>POINT (84.35083299999999 27.550556)</td>
    </tr>
  </tbody>
</table>
</div>




```python
mic_gbif_India.shape # dimension
```




    (6, 5)




```python
base=wrld_simpl.plot(color='pink', edgecolor='black', figsize=(9,5))
mic_gbif_India.plot(ax=base)
base.set_xlim(60,100)
base.set_ylim(1,40)
plt.title('India Map', fontsize =12)
plt.xlabel('Longitude', size =12)
plt.ylabel('Latitude', size =12)
```




    Text(48.875, 0.5, 'Latitude')




![png](Page_1_files/Page_1_41_1.png)



```python
mic_gbif_India.crs = wrld_simpl.crs
mic_gbif_India.crs
```




    {'init': 'epsg:4326'}




```python
outfp=r"C:\Users\Prajapati Vivek\Desktop\spatial project\mikania\html file\mic_gbif_India.shp"
mic_gbif_India.to_file(outfp)
```

### <font color='Cornflowerblue'> Extent using inbuilt functions</font>

<p style='text-align: justify;'>We can extract extent based on polygons or points as geometry. Each point in column <code>geometry</code> has their own extent. Similarly, each polygon in column <code>geometry</code> has thier own extent. But SouthAmerica is continent, it contains 37 countries in it. That means individual polygons as geometry contained in the <i>TM_WORLD_BORDERS_SIMPL-0.2.shp</i> shapefile represent each polygons of the country present in the World map respectively. And Points as geoemtry contained in <code><font color='green'>GeoDataFrame</font></code> with name <code>mic_gbif_america</code>.</p>

<p style='text-align: justify;'>In the following code, we extract the extent from individual Points of occurrence species, extent from Polygons of countries in world map and extent from Polygon of SouthAmerica.</p>

<font color='Cornflowerblue'><b>Extent from Points of occurrence species</b></font>

In the following code, extent of all points as geometry has been extracted using the attribute <code><font color='red'>.bounds</font></code>. You can see below in the table, each point is represented with its extent. 


```python
Extent_points = mic_gbif_america['geometry'].bounds
Extent_points[:10]# first 10 rows 
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>minx</th>
      <th>miny</th>
      <th>maxx</th>
      <th>maxy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-40.609514</td>
      <td>-19.990989</td>
      <td>-40.609514</td>
      <td>-19.990989</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-99.995244</td>
      <td>21.904365</td>
      <td>-99.995244</td>
      <td>21.904365</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-52.485556</td>
      <td>-30.324722</td>
      <td>-52.485556</td>
      <td>-30.324722</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-53.030722</td>
      <td>-24.068778</td>
      <td>-53.030722</td>
      <td>-24.068778</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-48.239353</td>
      <td>-2.672933</td>
      <td>-48.239353</td>
      <td>-2.672933</td>
    </tr>
    <tr>
      <th>5</th>
      <td>-52.124278</td>
      <td>-24.759639</td>
      <td>-52.124278</td>
      <td>-24.759639</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-53.753694</td>
      <td>-25.108444</td>
      <td>-53.753694</td>
      <td>-25.108444</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-55.705278</td>
      <td>-11.667778</td>
      <td>-55.705278</td>
      <td>-11.667778</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-95.750455</td>
      <td>18.593192</td>
      <td>-95.750455</td>
      <td>18.593192</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-51.255000</td>
      <td>-26.865000</td>
      <td>-51.255000</td>
      <td>-26.865000</td>
    </tr>
  </tbody>
</table>
</div>



<font color='Cornflowerblue'><b>Extent from Polygons of Country in World Map</b></font>

Similarly, extent of all polygons as geoemtry has been extracted.


```python
Extent_polygons = wrld_simpl['geometry'].bounds
Extent_polygons[:10] #first 10 rows
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>minx</th>
      <th>miny</th>
      <th>maxx</th>
      <th>maxy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-61.887222</td>
      <td>17.024441</td>
      <td>-61.686668</td>
      <td>17.703888</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-8.667223</td>
      <td>18.976387</td>
      <td>11.986475</td>
      <td>37.086388</td>
    </tr>
    <tr>
      <th>2</th>
      <td>44.778864</td>
      <td>38.442408</td>
      <td>50.374994</td>
      <td>41.897058</td>
    </tr>
    <tr>
      <th>3</th>
      <td>19.288609</td>
      <td>39.691200</td>
      <td>20.983490</td>
      <td>42.618050</td>
    </tr>
    <tr>
      <th>4</th>
      <td>43.460772</td>
      <td>38.841150</td>
      <td>46.541384</td>
      <td>41.297052</td>
    </tr>
    <tr>
      <th>5</th>
      <td>11.693609</td>
      <td>-18.016392</td>
      <td>24.020555</td>
      <td>-4.388990</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-170.826111</td>
      <td>-14.325003</td>
      <td>-169.444489</td>
      <td>-14.167501</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-73.583618</td>
      <td>-55.051674</td>
      <td>-53.807785</td>
      <td>-21.780521</td>
    </tr>
    <tr>
      <th>8</th>
      <td>112.951105</td>
      <td>-54.749725</td>
      <td>159.101900</td>
      <td>-10.051666</td>
    </tr>
    <tr>
      <th>9</th>
      <td>50.461662</td>
      <td>25.595276</td>
      <td>50.821388</td>
      <td>26.288887</td>
    </tr>
  </tbody>
</table>
</div>



<font color='Cornflowerblue'><b>Extent from Continent SouthAmerica</b></font>

<p style='text-align: justify;'>For Continent SouthAmerica, SouthAmerica contains 37 countries that means there are 37 polygons inside the boundary of SouthAmerica. We need to create single polygon by taking the union of all polygons inside the boundary of SouthAmerica. This is done with the method <code><font color='orange'>cascaded_union</font></code>. The method will bring out the union of all countries reside inside the boundary of SouthAmerica. In this way polygon of SouthAmerica has been created. And with the attribute <code><font color='red'>.bounds</font></code>, extent of SouthAmerica is extracted.</p>


```python
Extent_SouthAmerica = cascaded_union(mic_gbif_america['geometry']).bounds
Extent_SouthAmerica
```




    (-108.766667, -34.99, -34.950278000000004, 28.0)




```python
Extent_India = wrld_simpl[wrld_simpl['NAME']=='India']['geometry'].bounds
Extent_India
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>minx</th>
      <th>miny</th>
      <th>maxx</th>
      <th>maxy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>82</th>
      <td>68.197802</td>
      <td>6.745554</td>
      <td>97.348879</td>
      <td>35.501329</td>
    </tr>
  </tbody>
</table>
</div>



#### <font color="Cornflowerblue"> Working On Duplicacy</font>


```python
## duplicated rows of lon-lat columns for South America
dups=mic_gbif_america.duplicated(subset=('lon','lat')) 
```


```python
sum(dups) # total no. of duplicates
```




    559




```python
# dropping duplicated rows
mic_gbif_america=mic_gbif_america.drop_duplicates(subset=('lon','lat'),inplace=False)
mic_gbif_america.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed_ 0</th>
      <th>species</th>
      <th>lon</th>
      <th>lat</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>158</td>
      <td>Mikania micrantha</td>
      <td>-40.609514</td>
      <td>-19.990989</td>
      <td>POINT (-40.609514 -19.990989)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>415</td>
      <td>Mikania micrantha</td>
      <td>-99.995244</td>
      <td>21.904365</td>
      <td>POINT (-99.995244 21.904365)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>461</td>
      <td>Mikania micrantha</td>
      <td>-52.485556</td>
      <td>-30.324722</td>
      <td>POINT (-52.485556 -30.324722)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>485</td>
      <td>Mikania micrantha</td>
      <td>-53.030722</td>
      <td>-24.068778</td>
      <td>POINT (-53.030722 -24.068778)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>501</td>
      <td>Mikania micrantha</td>
      <td>-48.239353</td>
      <td>-2.672933</td>
      <td>POINT (-48.239353 -2.672933)</td>
    </tr>
  </tbody>
</table>
</div>



Why do we need to remove duplicated rows?

## [Previous Page](http://localhost:8888/notebooks/Desktop/spatial%20project/mikania/html%20file/Introduction.ipynb) <code>&nbsp;</code><code>&nbsp;</code><code>&nbsp;</code><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>  [Next Page](http://localhost:8888/notebooks/Desktop/spatial%20project/mikania/html%20file/Page_2.ipynb)
