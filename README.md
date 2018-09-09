# My Local School

### *Smart school zones for connected communities*

## Story

#### [Video](https://spark.adobe.com/video/8AmtcfeB6Lygw)

<a href="https://spark.adobe.com/video/8AmtcfeB6Lygw" target="_blank"><img src="https://content.screencast.com/users/groundtruth/folders/Snagit/media/784db926-9fd8-4805-9cb9-cf188e8a8c8d/09.09.2018-10.07.png" width="600" border="0"></a>

#### [Interactive Map](https://mylocalschool.pozi.com/#/layers[existingprimaryschoolzones]/layers[primaryschools]/)

#### Problem

#### Solution

## Resources

### Data Sources

* [ABS Mesh Blocks and Statistical Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/e350fd4f-c589-4804-a4e7-a1ead4987514)
* [Local Government Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/827752c4-a75e-4f86-9540-3bb96684e856)
* [Wyndham Subdivision Stage Boundaries](https://www.data.gov.au/dataset/wyndham-city-subdivision-stage-boundaries)
* [Victorian Department of Education and Training](https://www.data.vic.gov.au/data/dataset/school-locations-2018)
* [Melbourne School Zones](http://melbourneschoolzones.com)
* [OpenStreetMap](https://overpass-api.de/index.html)

### Software

* [QGIS](https://www.qgis.org/en/site/forusers/download.html)
* QGIS OpenLayers plug-in (QGIS > Plugins > Manage and Install Plugins > OpenLayers Plugin)
* Spatialite executables
  * [spatialite-gui](http://www.gaia-gis.it/gaia-sins/windows-bin-NEXTGEN-amd64/spatialite_gui-NG-win-amd64.7z)
  * [spatialite-cli](http://www.gaia-gis.it/gaia-sins/windows-bin-NEXTGEN-amd64/spatialite-cli-NG-win-amd64.7z)
  * [spatialite_network](http://www.gaia-gis.it/gaia-sins/windows-bin-amd64/spatialite_network-4.3.0a-win-amd64.7z)
  * [spatialite_osm_net](http://www.gaia-gis.it/gaia-sins/windows-bin-amd64/spatialite_osm_net-4.3.0a-win-amd64.7z)

## Process

Install QGIS and plugin, and unzip to data and Spatialite executables to new folder  `C:\GovHack2018\`. Here's how the folder should look.

```
C:\GovHack2018
  |--Data
    |--DET
      |--dv259-allschoolslist-2018.csv
    |--Melbourne School Zones
      |--data37.csv
    |-- PSMA
      |-- 2016 ABS Mesh Blocks and Statistical Areas NOVEMBER 2017
        |--Standard
          |--VIC_MB_2016_POLYGON_shp.shp
      |--Local Government Areas AUGUST 2018
        |--Standard
          |--VIC_LGA_POLYGON_shp.shp
    |--Wyndham
      |--wyndham-city-subdivisionstageboundary.json
  spatialite.exe
  spatialite_osm_overpass.exe
  spatialite_gui.exe
  spatialite_network.exe
  spatialite_osm_net.exe
```
