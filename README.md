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

* [ABS Mesh Blocks and Statistical Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/e350fd4f-c589-4804-a4e7-a1ead4987514) (Shapefile)
* [Local Government Areas](https://data.gov.au/dataset/psma-administrative-boundaries/resource/827752c4-a75e-4f86-9540-3bb96684e856) (Shapefile)
* Victorian Department of Education and Training
  * School Locations (2018): https://discover.data.vic.gov.au/dataset/school-locations-time-series (CSV)
  * School Locations (2020): https://www.findmyschool.vic.gov.au/js/app.ee3e8565.js (JS)
  * Primary School Catchments: https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/{z}/{x}/{y}.pbf (Mapbox Vector Tiles)
* [Vicmap Features of Interest](https://discover.data.vic.gov.au/dataset/vicmap-features-of-interest) (Shapefile)
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

Install QGIS and plugin, and unzip data and Spatialite executables to new folder `C:\MyLocalSchool\`.

Follow the processes documented in [Generate Victorian Schools Layer](generate-victorian-schools-layer.md) and [Generate Victorian Schools Zones Layers](generate-victorian-schools-zones-layers.md) to create the schools GeoJSON files.

Here's how the folder should look.

```
MyLocalSchool
├── Data
│   ├── DET
│   │   └── findmyschool.vic.gov.au
│   │       ├── js
│   │       │   ├── app.ee3e8565.js
│   │       │   └── app.ee3e8565.json
│   │       └── tiles
│   │           └── catchments_primary_2020
│   │               └── 9
│   │                   ├── 456
│   │                   ├── 457
│   │                   └── etc
│   └── PSMA
│       ├── 2016 ABS Mesh Blocks and Statistical Areas NOVEMBER 2017
│       │   └── Standard
│       │       └── VIC_MB_2016_POLYGON_shp.shp
│       └── Local Government Areas AUGUST 2018
│           └── Standard
│               └── VIC_LGA_POLYGON_shp.shp
├── spatialite.exe
├── spatialite_osm_overpass.exe
├── spatialite_gui.exe
├── spatialite_network.exe
└── spatialite_osm_net.exe
```

### Build Spatialite Database, starting with OpenStreetMap roads

Open command line at `C:\MyLocalSchool`

```bash
### Create Database using OpenStreetMap Roads, bounded by LGA coords
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

-- Exlude undesired road segments --;
delete from road_arcs where osm_id = 77616006; -- Footbridge over Skeleton Creek behind Sanctuary Lakes;

```

Press Ctrl-C to return to standard command prompt.

### Generate road network

```bash
.\spatialite_network --help
.\spatialite_network -d MyLocalSchool.sqlite -T road_arcs -f node_from -t node_to -g geometry --oneway-fromto oneway_ft --oneway-tofrom oneway_tf -n name -o roads_data -v roads_net --overwrite-output
```

#### Create look-up table of the road nodes that are actually connected to the road network

*Create a list of road nodes that connect to a known connected road node (eg, Watton Street Werribee: node 1861271106). Takes 2-3 minutes to process nodes in Wyndham LGA bounding box.*

```sql
.\spatialite MyLocalSchool.sqlite

create table road_nodes_connected_lut as
select node_id from
(
select r.*, n.node_id, n.geometry, max ( r.cost ) as travel_distance
from roads_net r, road_nodes n
where r.NodeFrom = n.node_id and r.NodeTo = 1861271106
group by n.node_id
)
where travel_distance > 0 or node_id = 1861271106;

create index road_nodes_connected_lut_node_id on road_nodes_connected_lut ( node_id );
```

Press Ctrl-C to return to standard command prompt.

#### Create table of connected road nodes by including only connected ones

```bash
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select * from road_nodes where node_id in ( select node_id from road_nodes_connected_lut )" -nln road_nodes_connected -nlt POINT -t_srs EPSG:4326 -update
```

```sql
.\spatialite MyLocalSchool.sqlite

-- Remove freeway nodes located within meshblocks that aren't directly connected to frewway --;
delete from road_nodes_connected where node_id in
(
select node_from from road_arcs where name like '% freeway' union
select node_to from road_arcs where name like '% freeway'
);

-- Test the route between two random nodes --;
select * from roads_net where nodefrom = 347257370 and nodeto = 347748405;
```

Press Ctrl-C to return to standard command prompt.

### Populate database with DET and ABS datasetes

```bash
### Add Schools from Victorian Department of Education GeoJSON
ogr2ogr MyLocalSchool.sqlite ".\Data\DET\findmyschool.vic.gov.au\js\app.ee3e8565.json" Allschools_DNB_with_other_schools -nln det_schools -update

### Add ABS Meshblocks Shapefile
ogr2ogr MyLocalSchool.sqlite ".\Data\PSMA\2016 ABS Mesh Blocks and Statistical Areas NOVEMBER 2017\Standard\VIC_MB_2016_POLYGON_shp.shp" VIC_MB_2016_POLYGON_shp -nlt POLYGON -t_srs EPSG:4326 -nln abs_meshblocks -update

### Create centroid layer for Meshblocks layer
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -sql "select mb_16pid, ST_PointOnSurface ( geometry ) as geometry from abs_meshblocks" -nlt POINT -t_srs EPSG:4326 -nln abs_meshblocks_points -update

### Add PSMA LGA Shapefile
ogr2ogr MyLocalSchool.sqlite ".\Data\PSMA\Local Government Areas AUGUST 2018\Standard\VIC_LGA_POLYGON_shp.shp" VIC_LGA_POLYGON_shp -nlt POLYGON -t_srs EPSG:4326 -nln psma_lga -update
```

### Optimise tables for processing single LGA

```sql
delete from abs_meshblocks_points where not ST_Within ( geometry , ( select ST_Envelope ( geometry ) from psma_lga where lga_pid = 'VIC221' ) );
delete from det_schools where not ST_Within ( geometry , ( select ST_Envelope ( geometry ) from psma_lga where lga_pid = 'VIC221' ) );
```

### Update and populate tables with nearest road arcs

```sql
create virtual table knn using VirtualKNN();

-- Update Meshblocks Points with nearest road nodes (12 minutes for Wyndham LGA alone) --
alter table abs_meshblocks_points add column nearest_road_node_fid integer;
alter table abs_meshblocks_points add column nearest_road_node_id integer;
update abs_meshblocks_points set
  nearest_road_node_fid = ( select fid from knn where f_table_name = 'road_nodes_connected' and ref_geometry = geometry and max_items = 1 );
update abs_meshblocks_points set
  nearest_road_node_id = ( select node_id from road_nodes_connected where ogc_fid = nearest_road_node_fid )
  where nearest_road_node_fid is not null;
create index abs_meshblocks_points_mb_16pid on abs_meshblocks_points ( mb_16pid );
create index abs_meshblocks_points_nearest_road_node_id on abs_meshblocks_points ( nearest_road_node_id );

alter table det_schools add column nearest_road_node_fid integer;
alter table det_schools add column nearest_road_node_id integer;

update det_schools set
  nearest_road_node_fid = ( select fid from knn where f_table_name = 'road_nodes_connected' and ref_geometry = geometry and max_items = 1 );
update det_schools set
  nearest_road_node_id = ( select node_id from road_nodes_connected where ogc_fid = nearest_road_node_fid )
  where nearest_road_node_fid is not null;
```

Press Ctrl-C to return to standard command prompt.

### Create layer for primary schools

```bash
### Create Government Primary Schools Layer
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select * from det_schools where zone = 'Yes' and campus_year_levels like 'Prep%'" -nln det_primary_schools -nlt POINT -t_srs EPSG:4326 -update
```

### Create and populate master look-up table

```sql
.\spatialite MyLocalSchool.sqlite

-- Create look-up-table of meshblock points and their nearest schools (5 seconds for Wyndham)
create table mls_lut as
select p.mb_16pid, p.nearest_road_node_id as start_node, k.*
from knn k, abs_meshblocks_points p
where f_table_name in ( 'det_primary_schools' , 'det_secondary_schools' )
and k.ref_geometry = p.geometry
and k.max_items = 5;

-- Add school id to the LUT
alter table mls_lut add column school_no text;
update mls_lut set
  school_no = ( select entity_code from det_primary_schools d where d.ogc_fid = fid)
  where f_table_name = 'det_primary_schools';

-- Add school name to the LUT. May not be necessary for final output, but useful for debugging
alter table mls_lut add column school_name text;
update mls_lut set
  school_name = ( select school_name from det_primary_schools d where d.ogc_fid = fid)
  where f_table_name = 'det_primary_schools';

-- Add the school's nearest road node to the LUT
alter table mls_lut add column end_node integer;
update mls_lut set
  end_node = ( select nearest_road_node_id from det_primary_schools d where d.ogc_fid = fid)
  where f_table_name = 'det_primary_schools';

create index mls_lut_start_node on mls_lut ( start_node );
create index mls_lut_end_node on mls_lut ( end_node );

-- Add travel distance to nearest road node to the LUT (20 seconds for Wyndham)
alter table mls_lut add column travel_distance real;
update mls_lut set
  travel_distance = ( select max ( cost ) from roads_net where nodefrom = start_node and nodeto = end_node );

-- Test results for Sassafras Close Point Cook
select mls_lut.*
from mls_lut
where mb_16pid = 'MB1620633179000';
```

### Create spatial tables of neighbourhoods and zones

```bash
### Generate Neighbourhood Local Primary School Table
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select l.mb_16pid, l.school_no, l.school_name, l.distance as straignt_line_distance, min ( travel_distance ) as travel_distance, m.geometry as geometry from mls_lut l join abs_meshblocks m on l.mb_16pid = m.mb_16pid where f_table_name = 'det_primary_schools' group by l.mb_16pid" -nln mls_neighbourhood_local_primary_school -nlt MULTIPOLYGON -t_srs EPSG:4326 -update

### Generate Local Primary School Zone Table
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select school_no, school_name, st_simplifypreservetopology ( st_union ( geometry ) , 0.00005 ) as geometry from mls_neighbourhood_local_primary_school group by school_no" -nln mls_primary_school_zones -nlt MULTIPOLYGON -t_srs EPSG:4326 -update

### Export GeoJSON file of school zones
ogr2ogr -f GeoJSON Data/mls_primary_school_zones.json MyLocalSchool.sqlite -dialect SQLite -sql "select *, '#' || substr(abs(random()),1,1) || substr(abs(random()),1,1) || substr(abs(random()),1,1) as fill from mls_local_primary_school_zone" -lco SIGNIFICANT_FIGURES=8
```

## Pros and cons of this approach

### Pros

* zones reflect natural and man-made boundaries, such as creeks, highways

### Cons

* all paths are considered equal, therefore it doesn't take into account crossing busy roads

## To Do

* [ ] test if deleting nodes solves issue with creek and freeway
* [ ] web map
  * [ ] add findmyschool vector tiles
  * [ ] add DET schools
  * [ ] add generated school zones
* [ ] populate meshblock point layer with lga to make it easy to filter by different lgas
* [ ] investigate "disconnected" road nodes issue
* [ ] switch process to whole of Victoria instead of Wyndham (`.\spatialite_osm_overpass -d MyLocalSchool.sqlite -minx 141 -maxx 150 -miny -39 -maxy -34 -mode ROAD`)
* [ ] experiment with excluding schools that have a `type` of 'Non standard' (eg Saltwater P-9 College) when generating zones, then add the zones at the end, cutting out a hole in the generated zones
```

