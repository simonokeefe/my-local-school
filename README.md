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

* [ABS Mesh Blocks and Statistical Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/e350fd4f-c589-4804-a4e7-a1ead4987514) (Esri Shapefile)
* [Local Government Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/827752c4-a75e-4f86-9540-3bb96684e856) (Esri Shapefile)
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

Press Ctrl-C to return to standard command prompt.

### Populate database with DET and ABS datasetes

```
### Add Schools from Victorian Department of Education CSV ###
### (need to alter filename `dv259-allschoolslist-2018.csv` to `dv259_allschoolslist_2018.csv` to avoid issues with import)
copy ".\Data\DET\dv259-allschoolslist-2018.csv" ".\Data\DET\dv259_allschoolslist_2018.csv"
ogr2ogr MyLocalSchool.sqlite ".\Data\DET\dv259_allschoolslist_2018.csv" -dialect sqlite -sql "select *, MakePoint(cast(X as real),cast(Y as real),4326) Geometry from dv259_allschoolslist_2018" -nln det_schools -nlt POINT -t_srs EPSG:4326 -update

### Add ABS Meshblocks Shapefile ###
ogr2ogr MyLocalSchool.sqlite ".\Data\PSMA\2016 ABS Mesh Blocks and Statistical Areas NOVEMBER 2017\Standard\VIC_MB_2016_POLYGON_shp.shp" VIC_MB_2016_POLYGON_shp -nlt POLYGON -nln abs_meshblocks -update

### Create centroid layer for Meshblocks layer ###
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -sql "select mb_16pid, ST_PointOnSurface ( geometry ) as geometry from abs_meshblocks" -nlt POINT -nln abs_meshblocks_points -update

### Add PSMA LGA Shapefile ###
ogr2ogr MyLocalSchool.sqlite ".\Data\PSMA\Local Government Areas AUGUST 2018\Standard\VIC_LGA_POLYGON_shp.shp" VIC_LGA_POLYGON_shp -nlt POLYGON -nln psma_lga -update
```

### Add missing schools to DET School

```
.\spatialite MyLocalSchool.sqlite

-- Edit DET Schools table with updated information --
INSERT INTO det_schools
    (school_no, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  VALUES (100001, 'Government', 'Carranballac P-9 College Jamieson Way Campus', 'Pri/Sec', 726, 'Wyndham (C)', MakePoint(144.745775,-37.8951,4326));
INSERT INTO det_schools
    (school_no, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  VALUES (100002, 'Government', 'Laverton P-12 College Laverton Primary School', 'Pri/Sec', 311, 'Hobsons Bay (C)', MakePoint(144.7704261,-37.8653317,4326));
INSERT INTO det_schools
    (school_no, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  VALUES (100003, 'Government', 'Baden Powell P-9 College Tarneit Campus', 'Pri/Sec', 726, 'Wyndham (C)', MakePoint(144.69418,-37.84238,4326));
```

### Optimise tables for processing single LGA

```
delete from abs_meshblocks_points where not ST_Within ( geometry , ( select geometry from psma_lga where lga_pid = 'VIC221' ) )
delete from det_schools where not lga_id in ( '726' , '275' , '515' , '465' , '118' , '311' );
```

### Update and populate tables with nearest road arcs

```
CREATE VIRTUAL TABLE knn USING VirtualKNN();

-- Update Meshblocks Points with nearest road nodes (12 minutes for Wyndham LGA alone) --
ALTER TABLE abs_meshblocks_points ADD COLUMN nearest_road_node INTEGER;
UPDATE abs_meshblocks_points SET
  nearest_road_node = ( select fid from knn where f_table_name = 'road_nodes' and ref_geometry = geometry and max_items = 1 )
  WHERE ST_Within ( geometry , ( select geometry from psma_lga where lga_pid = 'VIC221' ) );
CREATE INDEX abs_meshblocks_points_mb_16pid ON abs_meshblocks_points ( mb_16pid );
CREATE INDEX abs_meshblocks_points_nearest_road_node ON abs_meshblocks_points ( nearest_road_node );
```

Press Ctrl-C to return to standard command prompt.

### Create separate layers for government primary and secondary schools

```
### Create Government Primary Schools Layer
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select * from det_schools where education_sector = 'Government' and school_type in ( 'Primary' , 'Pri/Sec' )" -nln det_gov_primary_schools -nlt POINT -t_srs EPSG:4326 -update

### Create Government Secondary Schools Layer
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select * from det_schools where education_sector = 'Government' and school_type in ( 'Secondary' , 'Pri/Sec' ) and school_name != 'Suzanne Cory High School'" -nln det_gov_secondary_schools -nlt POINT -t_srs EPSG:4326 -update
```

### Update schools layers with nearest road nodes (2 minutes for schools in Wyndham and surrounding LGAs only)

*Note: the schools must be processed separately for the knn process to associate the fid with the features in the corresponding table.*

```
.\spatialite MyLocalSchool.sqlite

-- Update Primary Schools
ALTER TABLE det_gov_primary_schools ADD COLUMN nearest_road_node INTEGER;
UPDATE det_schools SET
  nearest_road_node = ( select fid from knn where f_table_name = 'road_nodes' and ref_geometry = geometry and max_items = 1 )
  WHERE lga_id in ( '726' , '275' , '515' , '465' , '118' , '311' );

-- Update Secondary Schools
ALTER TABLE det_gov_secondary_schools ADD COLUMN nearest_road_node INTEGER;
UPDATE det_schools SET
  nearest_road_node = ( select fid from knn where f_table_name = 'road_nodes' and ref_geometry = geometry and max_items = 1 )
  WHERE lga_id in ( '726' , '275' , '515' , '465' , '118' , '311' );
```

### Create and populate master look-up table

```
.\spatialite MyLocalSchool.sqlite

-- Create look-up-table of meshblock points and their nearest schools (up to 28 minutes for whole of Victoria, 1-2 minutes for Wyndham) --
CREATE TABLE mls_lut AS
SELECT p.mb_16pid, p.nearest_road_node as start_node, k.*
FROM knn k, abs_meshblocks_points p
WHERE f_table_name in ( 'det_gov_primary_schools' , 'det_gov_secondary_schools' )
AND k.ref_geometry = p.geometry
AND k.max_items = 5;

-- Add school name to the LUT. May not be necessary for final output, but useful for debugging --;
ALTER TABLE mls_lut ADD COLUMN school_name TEXT;
UPDATE mls_lut SET
  school_name = ( SELECT school_name FROM det_gov_primary_schools d WHERE d.ogc_fid = fid)
  WHERE f_table_name = 'det_gov_primary_schools';
UPDATE mls_lut SET
  school_name = ( SELECT school_name FROM det_gov_secondary_schools d WHERE d.ogc_fid = fid)
  WHERE f_table_name = 'det_gov_secondary_schools';

-- Add the school's nearest road node to the LUT --;
ALTER TABLE mls_lut ADD COLUMN end_node INTEGER;
UPDATE mls_lut SET
  end_node = ( SELECT nearest_road_node FROM det_gov_primary_schools d WHERE d.ogc_fid = fid)
  WHERE f_table_name = 'det_gov_primary_schools';
UPDATE mls_lut SET
  end_node = ( SELECT nearest_road_node FROM det_gov_secondary_schools d WHERE d.ogc_fid = fid)
  WHERE f_table_name = 'det_gov_secondary_schools';

CREATE INDEX mls_lut_start_node ON mls_lut ( start_node );
CREATE INDEX mls_lut_end_node ON mls_lut ( end_node );

ALTER TABLE mls_lut ADD COLUMN travel_distance REAL;
UPDATE mls_lut SET
  travel_distance = ( SELECT max ( cost ) FROM roads_net WHERE NodeFrom = start_node AND NodeTo = end_node );

-- Test results for Sassafras Close Point Cook --;
select mls_lut.*
from mls_lut
where mb_16pid = 'MB1620633179000';

--- Test navigating from Sassafras Close to Point Cook P-9; returns null because school's nearest road node is on disconnected path --;
SELECT * FROM roads_net WHERE NodeFrom = 2120747207 AND NodeTo = 2147483647;
```

### Generate DET School Zones layer (Voronoi polygons)

#### Government Primary Schools

QGIS > Vector > Geometry Tools > Voronoi Polygons > 

* Input layer: `det_gov_primary_schools`
* Buffer region: 10

QGIS > select layer `Voronoi polygons`: Layer > Save as >

* Format: Spatialite
* File name: `C:\MyLocalSchool\MyLocalSchool.sqlite`
* Layer name: `det_gov_primary_school_zones`

#### Government Secondary Schools

QGIS > Vector > Geometry Tools > Voronoi Polygons > 

* Input layer: `det_gov_secondary_schools`
* Buffer region: 10

QGIS > select layer `Voronoi polygons`: Layer > Save as >

* Format: Spatialite
* File name: `C:\MyLocalSchool\MyLocalSchool.sqlite`
* Layer name: `det_gov_secondary_school_zones`


