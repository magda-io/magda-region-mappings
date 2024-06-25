# magda-regions

This repo contains the default region files used by [Magda](https://github.com/magda-io/magda) for indexing &amp; region selector map.

### Region Files

Magda allow users to supply a list of region files that contains a list of regions available as region search filter options. The region files will be fetched via HTTP protocol by Magda's [indexer](https://github.com/magda-io/magda/tree/main/deploy/helm/internal-charts/indexer) module when it's required to create region index in search engine (e.g. for the first deployment).

The region files must be in `geojson` format with each "geometry" contains the following information in "properties":
- `id`: the id of the region. Must be unique across one region file.
- `name`: the name of the region.
- `shortName`: (optional) possible short name of the region.
- `requireSimplify`: (optional, default to `true`) region files geometries often contains too much details. Therefore, by default, the indexer will attempt to simply the geometries before index the region. If the region file is a simplified version, you will want to set this field to `false` in your region file to skip this processing step.

> Please note: we don't expect your region files having the exact same field names above. Instead, you are required to specify the field name to access those field for your file. e.g. By set `idField` config field to "id", the indexer will know to read the `id` value of the region from your field from "id" field.

### Config Magda Indexer with Region Files

Users can config Magda's [indexer](https://github.com/magda-io/magda/tree/main/deploy/helm/internal-charts/indexer) module helm chart to load alternative region files. Here is what default region config looks like:

```yaml
indexer:
  appConfig:
    regionSources:
      # Australia (Mainland) and all offshore territories as a whole
      COUNTRY:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/country.geojson"
        idField: "id"
        nameField: "name"
        order: 9
      # Regions for each of Australia offshore territories
      OFFSHORE_TERRITORIES:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/off-shore-territories.geojson"
        idField: "id"
        nameField: "name"
        lv1Id: "2"
        order: 11
      # ABS Statistical Area Level 4
      SA4:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/SA4_2021.geojson"
        idField: "SA4_CODE_2021"
        nameField: "SA4_NAME_2021"
        lv1Id: "1"
        lv2IdField: "STATE_CODE_2021"
        order: 30
      # ABS Statistical Area Level 3
      SA3:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/SA3_2021.geojson"
        idField: "SA3_CODE_2021"
        nameField: "SA3_NAME_2021"
        lv1Id: "1"
        lv2IdField: "STATE_CODE_2021"
        lv3IdField: "SA4_CODE_2021"
        order: 40
      # ABS Statistical Area Level 2
      SA2:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/SA2_2021.geojson"
        idField: "SA2_CODE_2021"
        nameField: "SA2_NAME_2021"
        lv1Id: "1"
        lv2IdField: "STATE_CODE_2021"
        lv3IdField: "SA4_CODE_2021"
        lv4IdField: "SA3_CODE_2021"
        order: 50
      # ABS Statistical Area Level 1
      SA1:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/SA1_2021.geojson"
        idField: "SA1_CODE_2021"
        nameField: "SA1_CODE_2021"
        lv1Id: "1"
        lv2IdField: "STATE_CODE_2021"
        lv3IdField: "SA4_CODE_2021"
        lv4IdField: "SA3_CODE_2021"
        lv5IdField: "SA2_CODE_2021"
        order: 60
      # Australia Local Government Areas
      LGA:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/LGA_2023.geojson"
        idField: "LGA_CODE_2023"
        nameField: "LGA_NAME_2023"
        lv1Id: "1"
        lv2IdField: "STATE_CODE_2021"
        order: 20
      # Australia Postal Areas
      POA:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/POA_2021.geojson"
        idField: "POA_CODE_2021"
        nameField: "POA_NAME_2021"
        lv1Id: "1"
        order: 70
      # Australia Commonwealth electoral boundaries
      ELB:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/ELB_2021.geojson"
        idField: "FID"
        nameField: "Elect_div"
        lv1Id: "1"
        order: 80
      # Australia State and Territory
      STE:
        url: "https://github.com/magda-io/magda-regions/releases/download/v2.0.0/STE.simplified.geojson"
        idField: "STE_CODE11"
        nameField: "STE_NAME11"
        shortNameField: "STE_ABBREV"
        lv1Id: "1"
        order: 10
```

Here:
- under `regionSources` section, you can specify one or more region files. The keys (e.g. `COUNTRY`, `OFFSHORE_TERRITORIES` and  `SA4` etc) are used as the `regionType` of the regions imported into search index by indexer module.
- For the each of the file entry, the following config fields are available:
  - `url`: the url of region file. Only support http/https URLs.
  - `idField`: Which field of the `properties` object of the region in geojson file should be used as the `id` of the imported region.
  - `nameField`: Which field of the `properties` object of the region in geojson file should be used as the `name` of the imported region.
  - `shortNameField`: (optional) which field of the `properties` object should be used as region `shortName`.
  - `includeIdInName`: (optional, default to `false`) When `true`, region name will be stored as string in format of `name-id`.
  - `disabled`: (optional, default to `false`) When `true`, the config entry will be ignore.
  - `order`: an integer number defines the sorting order of [region search api](https://dev.ai4m-p11.magda.io/api/v0/apidocs/index.html#api-Search-GetV0SearchRegions) search result.
  - `lv1IdField`: (optional) Which field of the `properties` object should be used as region's `lv1Id`.
    - The region records also used in Magda dataset authoring tool dataset spatial coverage select UI. `lv1Id` means level 1 region ID, which is used to represent "State" level regions used by the dataset spatial coverage select tool.
    - When supplied, the value is also available for searching via the [region search api](https://dev.ai4m-p11.magda.io/api/v0/apidocs/index.html#api-Search-GetV0SearchRegions).
  - `lv2IdField`: (optional) Which field of the `properties` object should be used as region's `lv2Id`.
    - `lv2Id` is used to represent "Region" level regions used by the dataset spatial coverage select tool.
    - When supplied, the value is also available for searching via the [region search api](https://dev.ai4m-p11.magda.io/api/v0/apidocs/index.html#api-Search-GetV0SearchRegions).
  - `lv3IdField`: (optional) Which field of the `properties` object should be used as region's `lv3Id`.
    - `lv3Id` is used to represent "Area" level regions used by the dataset spatial coverage select tool.
    - When supplied, the value is also available for searching via the [region search api](https://dev.ai4m-p11.magda.io/api/v0/apidocs/index.html#api-Search-GetV0SearchRegions).
  - `lv4IdField`: (optional) Which field of the `properties` object should be used as region's `lv4Id`.
    - When supplied, the value is available for searching via the [region search api](https://dev.ai4m-p11.magda.io/api/v0/apidocs/index.html#api-Search-GetV0SearchRegions).
  - `lv5IdField`: (optional) Which field of the `properties` object should be used as region's `lv5Id`.
    - When supplied, the value is available for searching via the [region search api](https://dev.ai4m-p11.magda.io/api/v0/apidocs/index.html#api-Search-GetV0SearchRegions).
  - `lv1Id`: (optional) Set the `lv1Id` all regions in this file to the supplied value.
  - `lv2Id`: (optional) Set the `lv2Id` all regions in this file to the supplied value.
  - `lv3Id`: (optional) Set the `lv3Id` all regions in this file to the supplied value.
  - `lv4Id`: (optional) Set the `lv4Id` all regions in this file to the supplied value.
  - `lv5Id`: (optional) Set the `lv5Id` all regions in this file to the supplied value.

### Config Magda Search API with Region Mapping

Region mapping file is served by Magda search api to frontend via [Get Region Types
](https://dev.magda.io/api/v0/apidocs/index.html#api-Search-GetV0SearchRegionTypes) API.

The region mapping file is created during the MVT (Mapbox Vector Tiles) generation and contains the meta-information such as:
- MVT server url
- min/max zoom
- bbox etc.

Those information will be used by frontend for render region boundaries in a map (e.g. region filter map).

> Please see `How to generate default region files` section for information of how to generate region files, MVT & region mapping files.

The JSON version of mapping file can be located [here](./regionMapping.json).

To config Magda's search api to use alternative region mapping, here is an example:

```yaml
search-api:
  appConfig:
    regionMapping:
      regionWmsMap:
        STE:
          layerName: STE_2021
          server: https://tiles.magda.io/STE_2021/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-STE_2021.json
          uniqueIdProp: FID
          regionProp: STATE_CODE_2021
          nameProp: STATE_NAME_2021
          aliases:
            - ste_code_2021
            - ste_code
            - ste
          description: States and Territories 2021
          bbox:
            - 96.81
            - -43.74
            - 168
            - -9.14
        SA1:
          layerName: SA1_2021
          server: https://tiles.magda.io/SA1_2021/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-SA1_2021.json
          uniqueIdProp: FID
          regionProp: SA1_CODE_2021
          nameProp: SA1_CODE_2021
          aliases:
            - sa1_code_2021
            - sa1_maincode_2021
            - sa1
            - sa1_code
          description: Statistical Area Level 1 2021 (ABS)
          bbox:
            - 96.81
            - -43.75
            - 159.11
            - -9.14
        SA2:
          layerName: SA2_2021
          server: https://tiles.magda.io/SA2_2021/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-SA2_2021.json
          uniqueIdProp: FID
          regionProp: SA2_CODE_2021
          nameProp: SA2_NAME_2021
          aliases:
            - sa2_code_2021
            - sa2_maincode_2021
            - sa2
            - sa2_code
          description: Statistical Area Level 2 2021 by code (ABS)
          bbox:
            - 96.81
            - -43.75
            - 168
            - -9.14
        SA3:
          layerName: SA3_2021
          server: https://tiles.magda.io/SA3_2021/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-SA3_2021.json
          uniqueIdProp: FID
          regionProp: SA3_CODE_2021
          nameProp: SA3_NAME_2021
          aliases:
            - sa3_code_2021
            - sa3_maincode_2021
            - sa3
            - sa3_code
          description: Statistical Area Level 3 2021 by code (ABS)
          bbox:
            - 96.81
            - -43.75
            - 168
            - -9.14
        SA4:
          layerName: SA4_2021
          server: https://tiles.magda.io/SA4_2021/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-SA4_2021.json
          uniqueIdProp: FID
          regionProp: SA4_CODE_2021
          nameProp: SA4_NAME_2021
          aliases:
            - sa4_code_2021
            - sa4_maincode_2021
            - sa4
            - sa4_code
          description: Statistical Area Level 4 2021 by code (ABS)
          bbox:
            - 96.81
            - -43.75
            - 168
            - -9.14
        LGA:
          layerName: LGA_2023
          server: https://tiles.magda.io/LGA_2023/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-LGA_2023.json
          uniqueIdProp: FID
          regionProp: LGA_CODE_2023
          nameProp: LGA_NAME_2023
          aliases:
            - lga_code_2023
            - lga_code
            - lga
          description: Local Government Areas 2023
          bbox:
            - 96.81
            - -43.74
            - 168
            - -9.14
        ELB:
          layerName: ELB_2021
          server: https://tiles.magda.io/ELB_2021/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-ELB_NAME_2021.json
          uniqueIdProp: FID
          regionProp: Elect_div
          nameProp: Elect_div
          aliases:
            - com_elb_name_2021
            - com_elb_name
          description: Commonwealth Electoral Divisions as at 2 August 2021 (AEC)
          bbox:
            - 96.81
            - -43.73
            - 168
            - -9.1
        POA:
          layerName: POA_2021
          server: https://tiles.magda.io/POA_2021/{z}/{x}/{y}.pbf
          serverType: MVT
          serverMaxNativeZoom: 12
          serverMinZoom: 0
          serverMaxZoom: 28
          regionIdsFile: build/TerriaJS/data/regionids/region_map-POA_2021.json
          uniqueIdProp: FID
          regionProp: POA_CODE_2021
          nameProp: POA_CODE_2021
          aliases:
            - poa_code_2021
            - poa_code
            - poa
            - postcode_2021
            - postcode
          description: Postal Areas 2021 (ABS)
          bbox:
            - 96.81
            - -43.74
            - 168
            - -9.14
```

> You can use [json-to-yaml](https://codebeautify.org/json-to-yaml) tool to convert json to yaml

### Production Use

The default config will pull region files from [the github repo release download area](https://github.com/magda-io/magda-regions/releases).

For production deployment, you might want to host those region files yourself in a more reliable way (e.g. put into a storage bucket).

### How to generate default region files

- Install GDAL 2.4.0 or later
- Install [Tippecanoe](https://github.com/mapbox/tippecanoe) version 1.32.10 or later
- Git clone [boundary-tiles](https://github.com/magda-io/boundary-tiles/tree/magda) repo and checkout `magda` branch.
- Run `yarn install`
- Find generation config file [config-magda.json5)](https://github.com/magda-io/boundary-tiles/blob/magda/config-magda.json5)
  - Find all required download files from the config files
  - Put download files into either `srcdata/geopackages` or `srcdata/[region type]` folder. 
- Run `yarn gulp all`
- You can find all region files from generated `geojson` folder.
  - files with name pattern `[region type].geojson` are the ones we need.

> Please note: the process also generates MVT (Mapbox Vector Tiles) that can be found from `mbtiles` folder.

> The region mapping file generated can be located from `regionMapping/regionMapping.json`. 
> Before use the region mapping file, you might want to change the server domain name in the region mapping file if you plan to host the tiles on your own infrastructure.

### License

Shield: [![CC BY 4.0][cc-by-shield]][cc-by]

This work is licensed under a
[Creative Commons Attribution 4.0 International License][cc-by].

[![CC BY 4.0][cc-by-image]][cc-by]

[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-image]: https://i.creativecommons.org/l/by/4.0/88x31.png
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg
