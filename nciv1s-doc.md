# UD4H's NCI 5-States (NCIV1S) Health Module Variant 

## API Documentation Overview
As part of its ongoing work on the Health Impact Assessment Application (HIAA), UD4H (http://www.ud4h.com) has created a health module variant called NCIV1S ("NCI 5-States").  The module API connects clients to a dataset that contains, for each of the census block groups (CBGs) in 5 states-- California, Michigan, Washington, Florida and Maryland-- a set of baseline characteristics (variables).

Access to this dataset is available via a web-served health module application programming interface (API).  The NCIV1S API exposes 3 different interfaces to registered clients:

1. NCIV1S Schema Metadata API.  The API returns a JSON list of all the baseline fields in the NCIV1S health module variant's data model.  Contains data types and descriptions for each variable.  The data is returned in JSON format.

2. NCIV1S Data Detail API.  For each 12-digit CBG GEOID10 included in a properly-formatted JSON object posted to the API, the API returns NCIV1S baseline variables for each requested CBG, as well as geometries representing the requested CBGs, and an overall RequestID (see #3 next).  The data is returned in GeoJSON format.

3. NCIV1S Request Summary API.  Clients submit a RequestID returned from a previously executed "NCIV1S Data Detail" request (#2 above), and the API returns a single feature containing a summary aggregation of the baseline variable numeric values, as well as a geometry representing the spatial aggregation of all the CBG geometries in the original request.  The data is returned in GeoJSON format.


## User Registration

To access the API, users must first register an email address and other organizational information with UD4H.  Contact http://urbandesign4health.com/contact-us-3 to register.  All API requests returning NCIV1S data must contain a registered email address in a parameter called "clientid".


## NCIV1S Schema Metadata API
Base URL: http://api.ud4htools.com/hmapi_get_varmeta_json/NCIV1S/

HTTP Request Type: GET

### Request Parameters

Parameter | Description
--------- | -----------
None | n/a

### Response Output
NCIV1S schema is returned in JSON format, with the following key-value pairs for each field in the NCIV1S health module variant:

Key | Description
--- | -----------
ordinal_position | Default field order
hmvar | Field's case-sensitive name
hmvartype | Field data type as defined for PostgreSQL: https://www.postgresql.org/docs/9.3/static/datatype.html#DATATYPE-TABLE
description | Description of the field, with its data source indicated in brackets

### Example Usage

Copy/paste the following link into a browser: http://api.ud4htools.com/hmapi_get_varmeta_json/NCIV1S/

The above link will return the following JSON (snipped for brevity):
```
[
  {
    "ordinal_position": 3,
    "hmvar": "totpop10",
    "hmvartype": "integer",
    "description": "Total population"
  },
  {
    "ordinal_position": 4,
    "hmvar": "ac_land",
    "hmvartype": "double precision",
    "description": "Total land area"
  },
  {
    "ordinal_position": 5,
    "hmvar": "ac_unpr",
    "hmvartype": "double precision",
    "description": "Unprotected land area"
  }

]
```

## NCIV1S Data Detail API
Base URL: http://api.ud4htools.com/hmapi_post_custom_inputs/NCIV1S/

HTTP Request Type: POST

### Request Parameters

Parameter | Description
--------- | -----------
clientid | Required. Email address associated with the client registration. See __User Registration__ section above. 
postjson | Required. JSON-formatted string containing at least a set of one or more 12-character GEOID10s.  Valid GEOID10 key/field names are GEOID10, geoid10cbg and geoid10c.  Ensure there is only one key/field name with one of these 3 names.  The 'postjson' parameter must adhere to the GeoJSON "FeatureCollection" format; see __Example Usage__ in the Request Summary API (below) for an example of a 'postjson' parameter format.  Though 'postjson' otherwise needs to be in GeoJSON "FeatureCollection" format, the actual 'geometry' section (large with polygon coordinates) is not used and can be omitted to save request space.

Detail request API calls are usually combined with Summary request calls (see below).  See __Example Usage__ in the Summary Request API at the end of this document for such a combined usage example.

### Response Output
a GeoJSON-formatted string is returned with two main branches:

1. A GeoJSON-formatted FeatureCollection section containing each CBG feature in the request, attributed with NCIV1S schema variables.  *NOTE: For individual requests up to 500 CBGs, the API returns the actual CBG polygons in each feature; for requests above 500, the API returns CBG centroid points in each feature.*
2. A __responseinfo__ section containing information about the request:

Key | Description
--- | -----------
requestid | An id associated with the request. The __Summary Data Request API__ (see below) requires requestid.
clientid | Email address associated with the client making the request.
hm_variant | Name of the health module variant requested, e.g. NCIV1S.
baseline_only_request | Boolean indicating that the request if for baseline data only.
custom_request | Boolean indicating that custom fields were detected by the API in the request's 'postjson' parameter
featurecount | Total number of census block groups in the request.
request_processing_time | Total time the API took to fulfill request. This total does not include processing time in the client before or after the request is made.
date_time_stamp | When the request was made.

Detail request API calls are usually followed immediately by a related Summary request API call.  See __Example Usage__ in the Summary Request API (next) for such a typical combined example.



## NCIV1S Request Summary API
Base URL: http://api.ud4htools.com/hmapi_get_summary_json/NCIV1S/

HTTP Request Type: GET

### Request Parameters

Parameter | Description
--------- | -----------
clientid | Required. Email address associated with the client registration. See __User Registration__ section above. 
requestid | Required. The integer requestid returned from a previous call to the detail request API.  Use __Response Output__ section of __Detail Data Request API__ (above) for more information about requestid.  

### Response Output
a GeoJSON-formatted string is returned with two main branches:

1. A GeoJSON-formatted FeatureCollection section containing a single summary "study area" feature attributed with input and health outcome NCIV1S schema field values summarized/aggregated from all component CBG attributes in the original detail request.  *NOTE: For original detail requests up to 500 CBGs, the summary API returns a polygon that is the exact merge/dissolve of all component CBG polygons that were in the original detail request; for original detail requests above 500, the API returns a rectangular bounding box polygon that spans the extent of all component CBG polygons that were in the original detail request.*
2. A __responseinfo__ section containing information about the request just made:

Key | Description
--- | -----------
requestid | An id associated with the request. The __Summary Data Request API__ (see below) requires requestid.
clientid | Email address associated with the client making the request.
custom_request | Boolean indicating that custom fields were detected by the API in the request's 'postjson' parameter
detail_feature_count | Total number of census block groups in the original detail request.
request_processing_time | Total time the API took to fulfill request. This total does not include processing time in the client before or after the request is made.
date_time_stamp | When the request was made.


### Example Usage
Summary request API calls typically follow Detail request calls. The following code demonstrates an example where a json variable containing both GEOID10 values and other values whose keys match the NCIV1S schema are contained in the postjson parameter, i.e. a "custom" request.

```
clientToken = "myregisteredemail@domain.com" ; // REPLACE with your unique registered email token

custDetail = '{
 "type": "FeatureCollection",
 "features": [
   {
    "type": "Feature", 
    "properties": {"GEOID10": "060530107013"}
   },
   {
    "type": "Feature", 
    "properties": {"GEOID10": "060530107011"}
   } 
 ] 
}';
jsonDetail = JSON.parse(custDetail);

$.post("http://api.ud4htools.com/hmapi_post_custom_inputs/NCIV1S/",
    {
      postjson: JSON.stringify(jsonDetail),
      clientid: clientToken
    },
    function(data,status){
      console.log("Response Status: " + status);
      customResponseType = data.type ;
      if (customResponseType == "FeatureCollection") {
        console.log("Valid JSON FeatureCollection returned.  Adding to CUSTOM map feature group");
        // add detail json to map or otherwise process it
        addJsonDataToMap(data, map, lyrGroupCustom, "gjtaCustom"); 

        // get requestid from json
        requestid = data.responseinfo.requestid ;

        // follow up with summary request
        $.getJSON("http://api.ud4htools.com/hmapi_get_summary_json/NCIV1S?clientid=" + clientToken + "&requestid=" + requestid, function(data) { 
          summaryResponseType = data.type ;
          if (summaryResponseType == "FeatureCollection") {
            console.log("Valid JSON FeatureCollection returned.  Adding to SUMMARY map feature group");
            // add summary json to map or otherwise process it
            addJsonDataToMap(data, map, lyrGroupSummary, "gjtaSummary"); 

          } else {
            console.log("summaryResponseType = " + summaryResponseType);
            alert("Error returned from " + data.ErrorSource + " API: " + data.Error);
          }
        });				

      } else {
        console.log("Error based on non-GIS customResponseType: " + customResponseType);
        alert("Error returned from " + data.ErrorSource + " API: " + data.Error);
      };

    }
  );

```

The following is a sample of the response output returned from a call to the detail request API:

```
{
   "type": "FeatureCollection",
   "responseinfo": {
      "hm_variant": "NCIV1S",
      "request_processing_time": "0.275 secs",
      "date_time_stamp": "2017-06-06 14:59:27.256204",
      "baseline_only_request": "True",
      "custom_request": "False",
      "clientid": "registered_email@client.com",
      "featurecount": "2",
      "requestid": "15"
   },
   "features": [
      {
         "id": "060530107011",
         "type": "Feature",
         "properties": {
            "ac_unpr": 4039.6799316,
            "pnpl": 0.100300602988,
            "retail_acc": 254.04713,
            "totpop10": 1624,
            "ac_land": 8848.84,
            "ac_denom": 4039.68066406209,
            "geoid10cbg": "060530107011",
            "sfips": 6,
            "d1b": 0.402011974473,
            "lowincpct": 0.13383208645054,
            "d3b": 1.7408018,
            "denom_type": "AC_UNPR",
            "pm25": 6.65956284153005
         },
         "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
               [
                  [
                     [
                        -121.695847926,
                        36.611498936
                     ],
                     [
                        -121.696275926,
                        36.610125935
                     ],
            < snipped for brevity >                    
                     [
                        -121.692035926,
                        36.608187936
                     ]
                  ]
               ]
            ]
         },
      {
         "id": "060530107013"
         "type": "Feature",
         "properties": {
            "ac_unpr": 276.1329956,
            "pnpl": 0.194241603967,
            "retail_acc": 296.04705,
            "totpop10": 1374,
            "ac_land": 388.11,
            "ac_denom": 276.132598877068,
            "geoid10cbg": "060530107013",
            "sfips": 6,
            "d1b": 4.97587030864,
            "lowincpct": 0.109403254972875,
            "d3b": 39.5725594,
            "denom_type": "AC_UNPR",
            "pm25": 6.65956284153005
         },         
         "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
               [
                  [
                     [
                        -121.695847926,
                        36.611498936
                     ],
                     [
                        -121.696275926,
                        36.610125935
                     ],
            < snipped for brevity >                    
                     [
                        -121.695847926,
                        36.611498936
                     ]
                  ]
               ]
            ]
         }
      }
   ]
}


```

The following is a sample of the GeoJSON returned from a summary request:
```
{
   "type": "FeatureCollection",
   "responseinfo": {
      "date_time_stamp": "2017-06-06 14:59:27.684399",
      "detail_feature_count": "2",
      "custom_request": "False",
      "request_processing_time": "0.247 secs",
      "requestid": "15",
      "clientid": "registered_email@client.com"
   },
   "features": [
      {
         "id": 15,
         "type": "Feature",
         "properties": {
            "ac_unpr": 4315.8129272,
            "pnpl": 0.143354283890317,
            "totpop10": 2998,
            "ac_land": 9236.95,
            "ac_denom": 4315.81326293916,
            "retail_acc": 273.295925890594,
            "sfips": null,
            "d1b": 0.694654707548283,
            "lowincpct": 0.122636217721283,
            "d3b": 4.16133784030609,
            "denom_type": null,
            "pm25": 6.65956284153005
         },         
         "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
               [
                  [
                     [
                        -121.692035926,
                        36.608187936
                     ],
                     [
                        -121.674971922,
                        36.6003089370001
                     ],
                     [
                        -121.692035926,
                        36.608187936
                     ]
                  ]
               ]
            ]
         }
      }
   ]
}
```



## Error Handling

If a problem with the request is encountered and therefore it cannot be fulfilled, instead of a GeoJSON-formatted response, a JSON-formatted error message will be returned.  All reponses from the API should be immediately inspected in the client application to determine if such an error message was returned and to handle it appropriately.  The error message format is consistent.  The following is an example request error message:

```
{
type: "Error",
Application: "HIAA",
ErrorSource: "hmapi_post_custom_inputs",
Error: "Error occured saving geoid10 list. Likely one of your geoid10 values is invalid.",
DateTimeStamp: "2017-05-18 23:13:22.183920"
}
```
