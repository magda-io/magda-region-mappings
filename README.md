# magda-region-mappings

This repo contains the default region mapping files used by [Magda](https://github.com/magda-io/magda) for indexing &amp; region selector map.

Magda allow users to supply a list of region mapping files that contains a list of regions available as region search filter options. The region mapping files will be fetched via HTTP protocol by Magda's [indexer](https://github.com/magda-io/magda/tree/main/deploy/helm/internal-charts/indexer) module when it's required to create region index in search engine (e.g. for the first deployment).

The region mapping files must be in `geojson` format with each "geometry" contains the following information in "properties":
- `id`: the id of the region. Must be unique across one region mapping file.
- `name`: the name of the region.
- `shortName`: (optional) possible short name of the region.

> Please note: we don't expect your region mapping files having the exact same field names above. Instead, you are required to specify the field name to access those field for your file. e.g. By set `idField` config field to "id", the indexer will know to read the `id` value of the region from your field from "id" field.

Users can config Magda's [indexer](https://github.com/magda-io/magda/tree/main/deploy/helm/internal-charts/indexer) module helm chart to load alternative region mapping files. Here is what default region mapping config looks like:

```yaml
indexer:
  appConfig:
    regionSources:
        # Australia (Mainland) and all offshore territories as a whole
        COUNTRY:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/country.geojson"
            idField: "id"
            nameField: "name"
            order: 9
        # Regions for each of Australia offshore territories
        OFFSHORE_TERRITORIES:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/off-shore-territories.geojson"
            idField: "id"
            nameField: "name"
            lv1Id: "2"
            order: 11
        # ABS Statistical Area Level 4
        SA4:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/SA4.geojson"
            idField: "SA4_CODE11"
            nameField: "SA4_NAME11"
            lv1Id: "1"
            lv2IdField: "STE_CODE11"
            order: 30
        # ABS Statistical Area Level 3
        SA3:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/SA3.geojson"
            idField: "SA3_CODE11"
            nameField: "SA3_NAME11"
            lv1Id: "1"
            lv2IdField: "STE_CODE11"
            lv3IdField: "SA4_CODE11"
            order: 40
        # ABS Statistical Area Level 2
        SA2:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/SA2.geojson"
            idField: "SA2_MAIN11"
            nameField: "SA2_NAME11"
            lv1Id: "1"
            lv2IdField: "STE_CODE11"
            lv3IdField: "SA4_CODE11"
            lv4IdField: "SA3_CODE11"
            order: 50
        # ABS Statistical Area Level 1
        SA1:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/SA1.geojson"
            idField: "SA1_MAIN11"
            nameField: "SA1_MAIN11",
            lv1Id: "1"
            lv2IdField: "STE_CODE11"
            lv3IdField: "SA4_CODE11"
            lv4IdField: "SA3_CODE11"
            lv5IdField: "SA2_MAIN11"
            includeIdInName: false,
            order: 60
        # Australia Local Government Areas
        LGA:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/LGA.geojson"
            idField: "LGA_CODE15"
            nameField: "LGA_NAME15"
            lv1Id: "1"
            steIdField: "STE_CODE11"
            order: 20
        # Australia Postal Areas
        POA:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/POA.geojson"
            idField: "POA_CODE"
            nameField: "POA_NAME"
            lv1Id: "1"
            order: 70
        # Australia Commonwealth electoral boundaries
        COM_ELB_ID_2016:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/COM_ELB_ID_2016.geojson"
            idField: "DIV_ID"
            nameField: "SORTNAME"
            lv1Id: "1"
            order: 80
        # Australia State and Territory
        STE:
            url: "https://github.com/magda-io/magda-region-mappings/releases/download/v1.0.0/STE.simplified.geojson"
            idField: "STE_CODE11"
            nameField: "STE_NAME11"
            shortNameField: "STE_ABBREV"
            lv1Id: "1"
            order: 10
```

Here:
- under `regionSources` section, you can specify one or more region mapping files. The keys (e.g. `COUNTRY`, `OFFSHORE_TERRITORIES` and  `SA4` etc) are used as the `regionType` of the regions imported into search index by indexer module.
- For the each of the mapping file entry, the following config fields are available:
  - `url`: the url of region mapping file. Only support http/https URLs.
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

### Production Use

The default config will pull region mapping files from [the github repo release download area](https://github.com/magda-io/magda-region-mappings/releases).

For production deployment, you might want to host those region mapping files yourself in a more reliable way (e.g. put into a storage bucket).


### License

Shield: [![CC BY 4.0][cc-by-shield]][cc-by]

This work is licensed under a
[Creative Commons Attribution 4.0 International License][cc-by].

[![CC BY 4.0][cc-by-image]][cc-by]

[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-image]: https://i.creativecommons.org/l/by/4.0/88x31.png
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg
