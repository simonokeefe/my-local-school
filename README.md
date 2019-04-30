# My Local School

### *Smart school zones for connected communities*

## Story

*'If I could fly it'd be perfect': The glitch with Melbourne's school zones*, [The Age, 18 Apr 2018](https://www.theage.com.au/national/victoria/if-i-could-fly-it-d-be-perfect-the-glitch-with-melbourne-s-school-zones-20180416-p4z9uw.html)

Victoria's current school zones make no sense. They increase travel distances, and add more cars to our roads at the busiest time of day. They fail because they don't take into account the paths and roads we use to travel.

**My Local School** uses open data from the Australian Bureau of Statistics, PSMA and OpenStreetMap to generate alternative school intake zones. These geographic datasets represent neighbourhood units that, combined with routing analysis, form cohesive community regions for fairer school intake zones. Households within these zones have the certainty that their assigned school is the shortest travel distance for their neighbourhood.

My Local School started as a project for [GovHack 2018](https://hackerspace.govhack.org/projects/my_local_school_280), and received the runner-up award for the [Growing Wyndham challenge](https://hackerspace.govhack.org/challenges/growing_wyndham).

#### [Video](https://spark.adobe.com/video/8AmtcfeB6Lygw)

<a href="https://spark.adobe.com/video/WfP0wesXohBmw" target="_blank"><img src="https://content.screencast.com/users/groundtruth/folders/Snagit/media/784db926-9fd8-4805-9cb9-cf188e8a8c8d/09.09.2018-10.07.png" width="600" border="0"></a>

#### [Interactive Map](https://mylocalschool.pozi.com/#/layers[existingprimaryschoolzones]/layers[primaryschools]/)

## Resources

### Data Sources

* [ABS Mesh Blocks and Statistical Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/e350fd4f-c589-4804-a4e7-a1ead4987514) (Esri Shapefile)
* [Local Government Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/827752c4-a75e-4f86-9540-3bb96684e856) (Esri Shapefile)
* [Victorian Department of Education and Training](https://discover.data.vic.gov.au/dataset/school-locations-time-series) (CSV)
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

```bash
### Create Database using OpenStreetMap Roads, bounded by LGA coords ###
.\spatialite_osm_overpass --help
.\spatialite_osm_overpass -d MyLocalSchool.sqlite -minx 144.44 -maxx 144.84 -miny -38.02 -maxy -37.78 -mode ROAD
```

### Clean road data

```sql
.\spatialite .\MyLocalSchool.sqlite

-- Clean Roads Data --;
delete from road_arcs where node_from = node_to;
delete from road_arcs where st_length(geometry) = 0;
delete from road_arcs where type in ('proposed');
update road_arcs set name = '' where name is null;
```

Press Ctrl-C to return to standard command prompt.

### Generate road network

```bash
.\spatialite_network --help
.\spatialite_network -d MyLocalSchool.sqlite -T road_arcs -f node_from -t node_to -g geometry --oneway-fromto oneway_ft --oneway-tofrom oneway_tf -n name -o roads_data -v roads_net --overwrite-output
```

#### Identify nodes found to not be on the road network

*Create a list of road nodes that don't connect to a known connected road node (eg, Watton Street Werribee: node 1861271106)*

```sql
.\spatialite MyLocalSchool.sqlite

create table road_nodes_disconnected_lut as
select node_id from
(
select r.*, n.node_id, n.geometry, max ( r.cost ) as travel_distance
from roads_net r, road_nodes n
where r.NodeFrom = n.node_id and r.NodeTo = 1861271106
and n.node_id <> 1861271106
group by n.node_id
)
where travel_distance = 0 or travel_distance is null;

create index road_nodes_disconnected_lut_node_id on road_nodes_disconnected_lut ( node_id );
```

Press Ctrl-C to return to standard command prompt.

```bash
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select * from road_nodes where node_id not in ( select node_id from road_nodes_disconnected_lut )" -nln road_nodes_connected -nlt POINT -t_srs EPSG:4326 -update
```

#### Test the route between two random nodes

```sql
.\spatialite MyLocalSchool.sqlite
select * from roads_net where nodefrom = 347257370 and nodeto = 347748405;
```

Press Ctrl-C to return to standard command prompt.

### Populate database with DET and ABS datasetes

```bash
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

```sql
.\spatialite MyLocalSchool.sqlite

-- Edit DET Schools table with updated information --
insert into det_schools
    (school_no, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  values (100001, 'Government', 'Carranballac P-9 College Jamieson Way Campus', 'Pri/Sec', 726, 'Wyndham (C)', MakePoint(144.745775,-37.8951,4326));
insert into det_schools
    (school_no, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  values (100002, 'Government', 'Laverton P-12 College Laverton Primary School', 'Pri/Sec', 311, 'Hobsons Bay (C)', MakePoint(144.7704261,-37.8653317,4326));
insert into det_schools
    (school_no, education_sector, school_name, school_type, lga_id, lga_name, geometry)
  values (100003, 'Government', 'Baden Powell P-9 College Tarneit Campus', 'Pri/Sec', 726, 'Wyndham (C)', MakePoint(144.69418,-37.84238,4326));
```

### Optimise tables for processing single LGA

```sql
delete from abs_meshblocks_points where not ST_Within ( geometry , ( select geometry from psma_lga where lga_pid = 'VIC221' ) )
delete from det_schools where not lga_id in ( '726' , '275' , '515' , '465' , '118' , '311' );
```

### Update and populate tables with nearest road arcs

```sql
create virtual table knn using VirtualKNN();

-- Update Meshblocks Points with nearest road nodes (12 minutes for Wyndham LGA alone) --
alter table abs_meshblocks_points add column nearest_road_node_fid integer;
alter table abs_meshblocks_points add column nearest_road_node_id integer;
update abs_meshblocks_points set
  nearest_road_node_fid = ( select fid from knn where f_table_name = 'road_nodes_connected' and ref_geometry = geometry and max_items = 1 )
  where st_within ( geometry , ( select geometry from psma_lga where lga_pid = 'VIC221' ) );
update abs_meshblocks_points set
  nearest_road_node_id = ( select node_id from road_nodes_connected where ogc_fid = nearest_road_node_fid )
  where nearest_road_node_fid is not null;
create index abs_meshblocks_points_mb_16pid on abs_meshblocks_points ( mb_16pid );
create index abs_meshblocks_points_nearest_road_node_id on abs_meshblocks_points ( nearest_road_node_id );

alter table det_schools add column nearest_road_node_fid integer;
alter table det_schools add column nearest_road_node_id integer;

update det_schools set
  nearest_road_node_fid = ( select fid from knn where f_table_name = 'road_nodes_connected' and ref_geometry = geometry and max_items = 1 )
  where lga_id in ( '726' , '275' , '515' , '465' , '118' , '311' );
update det_schools set
  nearest_road_node_id = ( select node_id from road_nodes_connected where ogc_fid = nearest_road_node_fid )
  where nearest_road_node_fid is not null;
```

Press Ctrl-C to return to standard command prompt.

### Create separate layers for government primary and secondary schools

```bash
### Create Government Primary Schools Layer
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select * from det_schools where education_sector = 'Government' and school_type in ( 'Primary' , 'Pri/Sec' )" -nln det_gov_primary_schools -nlt POINT -t_srs EPSG:4326 -update

### Create Government Secondary Schools Layer
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select * from det_schools where education_sector = 'Government' and school_type in ( 'Secondary' , 'Pri/Sec' ) and school_name != 'Suzanne Cory High School'" -nln det_gov_secondary_schools -nlt POINT -t_srs EPSG:4326 -update
```

### Create and populate master look-up table

```sql
.\spatialite MyLocalSchool.sqlite

-- Create look-up-table of meshblock points and their nearest schools (up to 28 minutes for whole of Victoria, 1-2 minutes for Wyndham) --
create table mls_lut as
select p.mb_16pid, p.nearest_road_node_id as start_node, k.*
from knn k, abs_meshblocks_points p
where f_table_name in ( 'det_gov_primary_schools' , 'det_gov_secondary_schools' )
and k.ref_geometry = p.geometry
and k.max_items = 5;

-- Add school name to the LUT. May not be necessary for final output, but useful for debugging --;
alter table mls_lut add column school_name text;
update mls_lut set
  school_name = ( select school_name from det_gov_primary_schools d where d.ogc_fid = fid)
  where f_table_name = 'det_gov_primary_schools';
update mls_lut set
  school_name = ( select school_name from det_gov_secondary_schools d where d.ogc_fid = fid)
  where f_table_name = 'det_gov_secondary_schools';

-- Add the school's nearest road node to the LUT --;
alter table mls_lut add column end_node integer;
update mls_lut set
  end_node = ( select nearest_road_node_id from det_gov_primary_schools d where d.ogc_fid = fid)
  where f_table_name = 'det_gov_primary_schools';
update mls_lut set
  end_node = ( select nearest_road_node_id from det_gov_secondary_schools d where d.ogc_fid = fid)
  where f_table_name = 'det_gov_secondary_schools';

create index mls_lut_start_node on mls_lut ( start_node );
create index mls_lut_end_node on mls_lut ( end_node );

alter table mls_lut add column travel_distance real;
update mls_lut set
  travel_distance = ( select max ( cost ) from roads_net where nodefrom = start_node and nodeto = end_node );

-- Test results for Sassafras Close Point Cook --;
select mls_lut.*
from mls_lut
where mb_16pid = 'MB1620633179000';
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


