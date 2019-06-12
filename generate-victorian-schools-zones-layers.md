# Generate Victorian Schools Zones Layers

This document describes how to obtain the zone boundaries of all current Victorian school intake zones from the Victorian Department of Education and Training website findmyschool.vic.gov.au and create a map layer file that can be used in GIS software.

## Background

The Victorian Department of Education and Training website findmyschool.vic.gov.au contains an interactive map that shows their current school intake zones. However this data is not available to download directly, making it hard to use for enquiry and analysis purposes.

## Data Contents

Within the contents of a hundreds of vector tile files (`.pbf`) on the findmyschool.vic.gov.au website are the boundaries of the school intake zones.

Example school:

```json
{ "type": "Feature", "properties": { "School_Name": "Woorinen District Primary School", "Campus_Name": "Woorinen District Primary School", "Year_Level": "P6", "Boundary_Year": "2020", "Entity_Code": "1543901" }, "geometry": { "type": "MultiPolygon", "coordinates": [ [ [ [ 143.437, -35.2484 ], [ 143.44, -35.2523 ], [ 143.448, -35.2523 ], [ 143.449, -35.2512 ], [ 143.452, -35.2523 ], [ 143.457, -35.2523 ], [ 143.459, -35.2552 ], [ 143.461, -35.2557 ], [ 143.462, -35.2591 ], [ 143.464, -35.2596 ], [ 143.464, -35.2624 ], [ 143.475, -35.2636 ], [ 143.475, -35.2669 ], [ 143.479, -35.2725 ], [ 143.48, -35.2714 ], [ 143.487, -35.2708 ], [ 143.49, -35.2692 ], [ 143.492, -35.2692 ], [ 143.496, -35.2669 ], [ 143.5, -35.2658 ], [ 143.503, -35.2664 ], [ 143.501, -35.2703 ], [ 143.502, -35.272 ], [ 143.505, -35.2725 ], [ 143.505, -35.2753 ], [ 143.502, -35.2781 ], [ 143.497, -35.2826 ], [ 143.498, -35.2871 ], [ 143.494, -35.291 ], [ 143.497, -35.2955 ], [ 143.497, -35.2978 ], [ 143.493, -35.3006 ], [ 143.497, -35.3073 ], [ 143.498, -35.3078 ], [ 143.501, -35.3067 ], [ 143.503, -35.3084 ], [ 143.501, -35.3151 ], [ 143.497, -35.3151 ], [ 143.487, -35.319 ], [ 143.481, -35.3157 ], [ 143.469, -35.3162 ], [ 143.466, -35.3207 ], [ 143.465, -35.328 ], [ 143.466, -35.3303 ], [ 143.471, -35.3325 ], [ 143.471, -35.3499 ], [ 143.468, -35.3504 ], [ 143.462, -35.3611 ], [ 143.457, -35.3639 ], [ 143.455, -35.37 ], [ 143.449, -35.3695 ], [ 143.437, -35.3717 ], [ 143.437, -35.2484 ] ] ] ] } },
```

## Extraction and Conversion

The following process shows how to extract the school zone data from the vector tile files on the findmyschool.vic.gov.au website and save it into a usable format.

*Note: The vector tiles at display in the website's map at zoom level 9 are sufficiently accurate for the purposes of visualising the zones at the street level. Greater precision can be achieved with level 10 or greater, but every additional zoom level will quadruple the number of tiles that must be downloaded.*

### Methodology

*Skip to code block below if you're not interested in recreating the methodology.*

* open a new tab in your browser, and press F12 to open the console window
* click on the Network tab
* open website https://www.findmyschool.vic.gov.au/
* zoom map and check the zoom level on the tile request URLs (eg https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/8/230/157.pbf indicates that it's a level 8 request because of the `/8/`)
* zoom until you start seeing level 9 requests (when it contains `/9/`)
* use Ctrl- (Control and minus) to make the page contents as small as possible (enabling the browser to fit as much of the map extent as possible)
* pan the map around until you've covered all of Victoria; the console will now contain all the level 9 vector tile URLs
* right-click on any request > Copy > Copy all as cURL
* paste into empty spreadsheet
* order alphabetically
* copy all the lines starting with `curl "https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/`
* use text editor and spreadsheet to manipulate text (find and replace text, combine multiple columns, etc) to generate commands in the format of `curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/307.pbf -o 9/456/307.pbf`
* make folders for all subfolder numbers found in the tile URLs (eg `9/456/`)

### Automation of Tile Download

```bash
cd C:/MyLocalSchool/Data/DET/findmyschool.vic.gov.au/tiles/catchments-primary-2020

#### Make folders to store tiles
mkdir 9
mkdir 9/456
mkdir 9/457
mkdir 9/458
mkdir 9/459
mkdir 9/460
mkdir 9/461
mkdir 9/462
mkdir 9/463
mkdir 9/464
mkdir 9/465
mkdir 9/466
mkdir 9/467
mkdir 9/468
mkdir 9/469

#### Download vector tiles from findmyschoo.vic.gov.au
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/307.pbf -o 9/456/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/308.pbf -o 9/456/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/309.pbf -o 9/456/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/310.pbf -o 9/456/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/311.pbf -o 9/456/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/312.pbf -o 9/456/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/313.pbf -o 9/456/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/314.pbf -o 9/456/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/315.pbf -o 9/456/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/456/316.pbf -o 9/456/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/307.pbf -o 9/457/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/308.pbf -o 9/457/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/309.pbf -o 9/457/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/310.pbf -o 9/457/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/311.pbf -o 9/457/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/312.pbf -o 9/457/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/313.pbf -o 9/457/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/314.pbf -o 9/457/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/315.pbf -o 9/457/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/457/316.pbf -o 9/457/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/307.pbf -o 9/458/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/308.pbf -o 9/458/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/309.pbf -o 9/458/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/310.pbf -o 9/458/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/311.pbf -o 9/458/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/312.pbf -o 9/458/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/313.pbf -o 9/458/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/314.pbf -o 9/458/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/315.pbf -o 9/458/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/458/316.pbf -o 9/458/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/307.pbf -o 9/459/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/308.pbf -o 9/459/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/309.pbf -o 9/459/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/310.pbf -o 9/459/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/311.pbf -o 9/459/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/312.pbf -o 9/459/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/313.pbf -o 9/459/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/314.pbf -o 9/459/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/315.pbf -o 9/459/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/459/316.pbf -o 9/459/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/307.pbf -o 9/460/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/308.pbf -o 9/460/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/309.pbf -o 9/460/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/310.pbf -o 9/460/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/311.pbf -o 9/460/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/312.pbf -o 9/460/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/313.pbf -o 9/460/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/314.pbf -o 9/460/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/315.pbf -o 9/460/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/460/316.pbf -o 9/460/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/307.pbf -o 9/461/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/308.pbf -o 9/461/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/309.pbf -o 9/461/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/310.pbf -o 9/461/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/311.pbf -o 9/461/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/312.pbf -o 9/461/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/313.pbf -o 9/461/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/314.pbf -o 9/461/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/315.pbf -o 9/461/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/461/316.pbf -o 9/461/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/307.pbf -o 9/462/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/308.pbf -o 9/462/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/309.pbf -o 9/462/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/310.pbf -o 9/462/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/311.pbf -o 9/462/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/312.pbf -o 9/462/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/313.pbf -o 9/462/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/314.pbf -o 9/462/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/315.pbf -o 9/462/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/462/316.pbf -o 9/462/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/307.pbf -o 9/463/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/308.pbf -o 9/463/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/309.pbf -o 9/463/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/310.pbf -o 9/463/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/311.pbf -o 9/463/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/312.pbf -o 9/463/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/313.pbf -o 9/463/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/314.pbf -o 9/463/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/315.pbf -o 9/463/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/463/316.pbf -o 9/463/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/307.pbf -o 9/464/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/308.pbf -o 9/464/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/309.pbf -o 9/464/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/310.pbf -o 9/464/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/311.pbf -o 9/464/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/312.pbf -o 9/464/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/313.pbf -o 9/464/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/314.pbf -o 9/464/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/315.pbf -o 9/464/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/464/316.pbf -o 9/464/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/307.pbf -o 9/465/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/308.pbf -o 9/465/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/309.pbf -o 9/465/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/310.pbf -o 9/465/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/311.pbf -o 9/465/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/312.pbf -o 9/465/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/313.pbf -o 9/465/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/314.pbf -o 9/465/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/315.pbf -o 9/465/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/465/316.pbf -o 9/465/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/307.pbf -o 9/466/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/308.pbf -o 9/466/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/309.pbf -o 9/466/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/310.pbf -o 9/466/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/311.pbf -o 9/466/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/312.pbf -o 9/466/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/313.pbf -o 9/466/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/314.pbf -o 9/466/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/315.pbf -o 9/466/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/466/316.pbf -o 9/466/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/307.pbf -o 9/467/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/307.pbf -o 9/467/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/308.pbf -o 9/467/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/309.pbf -o 9/467/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/310.pbf -o 9/467/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/311.pbf -o 9/467/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/312.pbf -o 9/467/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/313.pbf -o 9/467/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/314.pbf -o 9/467/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/315.pbf -o 9/467/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/467/316.pbf -o 9/467/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/307.pbf -o 9/468/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/308.pbf -o 9/468/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/309.pbf -o 9/468/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/310.pbf -o 9/468/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/311.pbf -o 9/468/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/312.pbf -o 9/468/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/313.pbf -o 9/468/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/314.pbf -o 9/468/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/315.pbf -o 9/468/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/468/316.pbf -o 9/468/316.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/307.pbf -o 9/469/307.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/308.pbf -o 9/469/308.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/309.pbf -o 9/469/309.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/310.pbf -o 9/469/310.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/311.pbf -o 9/469/311.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/312.pbf -o 9/469/312.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/313.pbf -o 9/469/313.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/314.pbf -o 9/469/314.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/315.pbf -o 9/469/315.pbf
curl https://www.findmyschool.vic.gov.au/tiles/catchments-primary-2020/9/469/316.pbf -o 9/469/316.pbf

```

The `catchments-primary-2020/9` now contains all the vector tiles needed to build the zones layer.

### Create Map Layer From Vector Tiles

```bash
cd C:/MyLocalSchool

#### Import folder of vector tiles into SQLite database (to enable further processing)
ogr2ogr MyLocalSchool.sqlite C:/MyLocalSchool/Data/DET/findmyschool.vic.gov.au/tiles/catchments-primary-2020/9 catchments -nln det_primary_school_zones_unmerged -s_srs EPSG:3857 -t_srs EPSG:4326 -nlt PROMOTE_TO_MULTI -update -skipfailures

#### Merge zone polygons that were split by tile boundaries
ogr2ogr MyLocalSchool.sqlite MyLocalSchool.sqlite -dialect sqlite -sql "select school_name, campus_name, year_level, boundary_year, entity_code, st_union ( geometry ) as geometry from det_primary_school_zones_unmerged group by entity_code" -nln det_primary_school_zones -nlt MULTIPOLYGON -t_srs EPSG:4326 -update

#### Export to GeoJSON
ogr2ogr -f GeoJSON Data/det_primary_school_zones.json MyLocalSchool.sqlite det_primary_school_zones -lco SIGNIFICANT_FIGURES=8

```

The output file is a GeoJSON file, which is an open format that can be used in a variety of web and desktop applications.

Example (generated on 12 Jun 2019): https://github.com/simonokeefe/my-local-school/blob/master/data/det_primary_school_zones.json

To preview the file, open http://geojson.io and drag the file onto the map.

---
