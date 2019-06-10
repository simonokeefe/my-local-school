# Generate Victorian Schools Layer

This document describes how to obtain the location of all Victorian schools from the Victorian Department of Education and Training website findmyschool.vic.gov.au and create a map layer file that can be used in GIS software.

## Background

The Victorian Department of Education and Training website findmyschool.vic.gov.au contains a map that shows the locations of all Victorian schools. However this school data is not available in an open format, making it hard to use for enquiry and analysis purposes.

*(The Department has published a list of schools at https://discover.data.vic.gov.au/dataset/school-locations-time-series, but this data is incomplete and out of date.)*

## Data Contents

Within the contents of a `.js` file on the findmyschool.vic.gov.au website are the details of 1774 schools.

Example school:

```json
{
    "type": "Feature",
    "properties": {
        "School_Name": "Carranballac P-9 College",
        "School_Type": "Pri/Sec",
        "Specialisation": "",
        "Campus_Name": "Jamieson Way Campus",
        "Campus_Type": "Pri/Sec",
        "Campus_Address_Line_1": "Cnr Jamieson Way & La Rochelle",
        "Campus_Address_Town": "Point Cook",
        "Campus_Address_State": "VIC",
        "Campus_Postcode": 3030,
        "School_Phone": "03 9395 3533",
        "School_Website": "http://www.carranballac.vic.edu.au/",
        "Managed_Enrolments": "",
        "Campus_Status": "Open",
        "Campus_Year_Levels": "Prep to 9",
        "primary_2020": "x",
        "year7_2020": "x",
        "year8_2020": "x",
        "year9_2020": "x",
        "year10_2020": "",
        "year11_2020": "",
        "year12_2020": "",
        "DET_Region": "South West",
        "type": "Straight line",
        "Zone": "Yes",
        "Entity_Code": "1548602",
        "juniorsec_2020": "x",
        "seniorsec_2020": "",
        "singlesex_2020": "",
        "specialist_2020": ""
    },
    "geometry": {
        "type": "Point",
        "coordinates": [
            144.74576459559174,
            -37.89516211547399
        ]
    }
},
```

## Conversion

The following process shows step-by-step how to extract the school location data from the .js file on the findmyschool.vic.gov.au website and save it into a usable format.

### Generate GeoJSON

* open a new tab in your browser, and press F12 to open the console window
* click on the Network tab
* open website https://www.findmyschool.vic.gov.au/
* sort Network list by size
* select largest file (eg app.ee3e8565.js, 1.4 MB) > right click > Save as... save to your local PC
* open downloaded file in a text editor (eg Notepad++)
* search for `e.exports=`
* delete the text `e.exports=` and everything before it, all the way back to the start of the file (ie, after deletion, the file should start with `{type:"FeatureCollection"`)
* place the cursor at the `{` bracket character at the start of file
* scroll to the end of file
* near the bottom, Notepad++ will highlight the `}` bracket that matches the first bracket
* delete everything after the matching bracket
* select all, and copy entire file content into http://jsonparseronline.com/
* paste parsed output from http://jsonparseronline.com/ into file, replacing original content
* find and replace every field property key:
  * `School_Name:` > `"School_Name":`
  * `School_Type:` > `"School_Type":`
  * ...
  * `specialist_2020:` > `"specialist_2020":`
* find and replace all other property keys:
  * `type:` > `"type":`
  * `features:` > `"features":`
  * `properties:` > `"properties":`
  * `crs:` > `"crs":`
  * `name:` > `"name":`
  * `geometry:` > `"geometry":`
  * `coordinates:` > `"coordinates":`
* save as a `.json` file

The file is now a GeoJSON file, which is an open format that can be used in a variety of web and desktop applications.

Example (generated on 28 May 2019): https://github.com/simonokeefe/my-local-school/blob/master/data/app.ee3e8565.json

To quickly preview the file, open http://geojson.io and drag the file onto the map.

### Generate CSV

The new GeoJSON file can be converted to CSV format for viewing as a spreadsheet or importing into a database.

* run OSGeo4W Shell (included in QGIS installation)
* `cd` into folder containing the json file
* run the following:

```
ogr2ogr -f CSV app.ee3e8565.csv app.ee3e8565.json
```