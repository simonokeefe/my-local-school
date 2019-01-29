# My Local School

### *Smart school zones for connected communities*

## Story

*'If I could fly it'd be perfect': The glitch with Melbourne's school zones*, [The Age, 18 Apr 2018](https://www.theage.com.au/national/victoria/if-i-could-fly-it-d-be-perfect-the-glitch-with-melbourne-s-school-zones-20180416-p4z9uw.html)

Victoria's current school zones make no sense. They increase travel distances, and add more cars to our roads at the busiest time of day. They fail because they don't take into account the paths and roads we use to travel.

**My Local School** uses open data from the Australian Bureau of Statistics, PSMA and Wyndham City Council to generate alternative school intake zones. These geographic datasets represent neighbourhood units that, combined with routing analysis, form cohesive community regions for fairer school intake zones. Households within these zones have the certainty that their assigned school is the shortest travel distance for their neighbourhood.

My Local School started as a project for [GovHack 2018](https://hackerspace.govhack.org/projects/my_local_school_280), and received the runner-up award for the [Growing Wyndham challenge](https://hackerspace.govhack.org/challenges/growing_wyndham).

#### [Video](https://spark.adobe.com/video/8AmtcfeB6Lygw)

<a href="https://spark.adobe.com/video/WfP0wesXohBmw" target="_blank"><img src="https://content.screencast.com/users/groundtruth/folders/Snagit/media/784db926-9fd8-4805-9cb9-cf188e8a8c8d/09.09.2018-10.07.png" width="600" border="0"></a>

#### [Interactive Map](https://mylocalschool.pozi.com/#/layers[existingprimaryschoolzones]/layers[primaryschools]/)

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

Install QGIS and plugin, and unzip data and Spatialite executables to new folder `C:\MyLocalSchool\`. Here's how the folder should look.

```
C:\MyLocalSchool
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

### Build Spatialite Database, starting with OpenStreetMap roads

Open command line at `C:\MyLocalSchool`

```
### Create Database using OpenStreetMap Roads, bounded by LGA coords ###
.\spatialite_osm_overpass --help
.\spatialite_osm_overpass -d MyLocalSchool.sqlite -minx 144.44 -maxx 144.84 -miny -38.02 -maxy -37.78 -mode ROAD
```

### Populate database with datasetes

```
### Add Wyndham Subdivsion Stages ###
ogr2ogr MyLocalSchool.sqlite ".\Data\Wyndham\wyndham-city-subdivisionstageboundary.json" wyndham-city-subdivisionstageboundary -nlt POLYGON -nln wcc_subdivision_stages -dim 2 -update

### Add Schools from Victorian Department of Education CSV ###
### (need to alter filename `dv259-allschoolslist-2018.csv` to `dv259_allschoolslist_2018.csv` to avoid issues with import)
copy ".\Data\DET\dv259-allschoolslist-2018.csv" ".\Data\DET\dv259_allschoolslist_2018.csv"
ogr2ogr MyLocalSchool.sqlite ".\Data\DET\dv259_allschoolslist_2018.csv" -dialect sqlite -sql "select *, MakePoint(cast(X as real),cast(Y as real),4326) Geometry from dv259_allschoolslist_2018" -nln det_schools -nlt POINT -t_srs EPSG:4326 -update

### Add ABS Meshblocks Shapefile ###
ogr2ogr MyLocalSchool.sqlite ".\Data\PSMA\2016 ABS Mesh Blocks and Statistical Areas NOVEMBER 2017\Standard\VIC_MB_2016_POLYGON_shp.shp" VIC_MB_2016_POLYGON_shp -nlt POLYGON -nln abs_meshblocks -update

### Add PSMA LGA Shapefile ###
ogr2ogr MyLocalSchool.sqlite ".\Data\PSMA\Local Government Areas AUGUST 2018\Standard\VIC_LGA_POLYGON_shp.shp" VIC_LGA_POLYGON_shp -nlt POLYGON -nln psma_lga -update

### Add Schools from Melbourne School Zones CSV ###
ogr2ogr MyLocalSchool.sqlite ".\Data\Melbourne School Zones\data37.csv" -dialect sqlite -sql "select *, MakePoint(cast(lng as real),cast(lat as real),4326) Geometry from data37" -nln msz_schools -nlt POINT -t_srs EPSG:4326 -update
```

### Generate derived layers

```
### Create centroid layer for Subdivision Stage layer ###
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -sql "select id1, ST_PointOnSurface ( geometry ) as geometry from wcc_subdivision_stages" -nlt POINT -nln wcc_subdivision_stages_points -update

### Create centroid layer for Meshblocks layer for Wyndham LGA ###
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -sql "select mb_16pid, ST_PointOnSurface ( geometry ) as geometry from abs_meshblocks" -nlt POINT -nln abs_meshblocks_points -update
```

### Update and populate tables with nearest road arcs

```
.\spatialite MyLocalSchool.sqlite

-- Edit DET Schools table with updated information --
INSERT INTO det_schools
    (ogc_fid, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  VALUES (100001, 'Government', 'Carranballac P-9 College Jamieson Way Campus', 'Pri/Sec', 726, 'Wyndham (C)', MakePoint(144.745775,-37.8951,4326));
INSERT INTO det_schools
    (ogc_fid, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  VALUES (100002, 'Government', 'Laverton P-12 College Laverton Primary School', 'Pri/Sec', 311, 'Hobsons Bay (C)', MakePoint(144.7704261,-37.8653317,4326));
INSERT INTO det_schools
    (ogc_fid, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  VALUES (100003, 'Government', 'Baden Powell P-9 College Tarneit Campus', 'Pri/Sec', 726, 'Wyndham (C)', MakePoint(144.69418,-37.84238,4326));

-- Update tables with nearest road arcs --
ALTER TABLE abs_meshblocks_points ADD COLUMN nearest_road_node INTEGER;
ALTER TABLE wcc_subdivision_stages_points ADD COLUMN nearest_road_node INTEGER;
CREATE VIRTUAL TABLE knn USING VirtualKNN();
UPDATE wcc_subdivision_stages_points SET
  nearest_road_node = ( select fid from knn where f_table_name = 'road_nodes' and ref_geometry = geometry and max_items = 1 );
UPDATE abs_meshblocks_points SET
  nearest_road_node = ( select fid from knn where f_table_name = 'road_nodes' and ref_geometry = geometry and max_items = 1 )
  WHERE ST_Within ( geometry , ( select geometry from psma_lga where lga_pid = 'VIC221' ) );

```

### Clean road data

```
.\spatialite .\MyLocalSchool.sqlite

-- Clean Roads Data --;
DELETE FROM road_arcs WHERE node_from = node_to;
DELETE FROM road_arcs WHERE ST_Length(geometry) = 0;
DELETE FROM road_arcs WHERE type IN ('Proposed');
UPDATE road_arcs SET name = '' WHERE name IS NULL;
```

Press Ctrl-C to return to standard command prompt.

### Generate road network

```
.\spatialite_network --help
.\spatialite_network -d MyLocalSchool.sqlite -T road_arcs -f node_from -t node_to -g geometry --oneway-fromto oneway_ft --oneway-tofrom oneway_tf -n name -o roads_data -v roads_net --overwrite-output
```

Test the route between two random nodes

```
.\spatialite MyLocalSchool.sqlite
SELECT * FROM roads_net WHERE NodeFrom = 347257370 AND NodeTo = 347748405;
```

### Generate DET School Zones layer (Voronoi polygons)

In QGIS, set up layer filter on `det_schools` layer to exclude where `school_name = 'Suzanne Cory High School' or school_type = 'Special'`

QGIS > select layer: Layer > Save as > GeoJSON, `C:\MyLocalSchool\Data\DET\det_primary_schools.json`

QGIS > Vector > Geometry Tools > Voronoi Polygons > Run
QGIS > select layer: Layer > Save as > GeoJSON, `C:\MyLocalSchool\Data\DET\det_primary_school_zones.json`, Coordinate Precision: 6

```
### Add DET School Zones layer to database ###
ogr2ogr MyLocalSchool.sqlite ".\Data\DET\det_primary_school_zones.json" det_primary_school_zones -nlt POLYGON -nln det_primary_school_zones -update
```

### Prepare school data for routing

```
#### Add column in Schools table for closest road node ####
ALTER TABLE det_schools ADD COLUMN closest_road_node;
```

