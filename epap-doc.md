# UD4H's EPA-Prime (EPAP) Health Module Variant - API Documentation

## Overview
As part of its ongoing work on the Health Impact Assessment Application (HIAA), UD4H (http://www.ud4h.com) has created a dataset called EPAP ("EPA Prime") that contains, for each of the 220,653 census block groups in the United States, a set of baseline inputs and health outcomes derived from these inputs.  

The baseline inputs are derived from census block group (CBG) polygons attributed with data fields from the Smart Location Database (SLD), the National Land Cover Database (NLCD) and from other census data.  The health outcomes are calculated using coefficients derived from analysis related to indepth research sources like the American Community Survey (ACS).

Access to this dataset and its calculated outcomes is available via a web-served health module application programming interface (API).  The API responds to three types of requests:

1. Metadata requests.  The API returns a JSON list of all the input and health outcome fields in the EPAP health module variant's data model.  Contains data types and descriptions.

2. Detail data requests.  If only 12-digit CBG unique identifier codes (GEOID10s) are supplied as parameters, the API returns EPAP baseline inputs and health outcomes attached to a feature collection of the requested CBGs in GeoJSON format.  If, additionally, the request also includes custom input values for one or more keys matching the EPAP schema, the API returns the custom inputs, with EPAP baseline defaults for any inputs missing from the request, and custom health outcomes calculated using the custom inputs and needed baseline defaults. 

3. Summary data requests.  The API takes a supplied requestid (returned from a previous detail data request-- see #2 above) and returns a GeoJSON-formatted response containing a single polygon representing the merge/dissolve of all CBG polygons included in the original detail request.  This summary "study area" geometry is attributed with summarized/aggregated input amd health outcome EPAP schema field values.


## ArcGIS Tool

Non-technical users can download and install an <a href="">ArcToolbox plug-in tool which handles communication with the UD4H API</a>.  User documentation is available in an appendix of <a href="">this document</a>.

The remainder of this document is intended to describe how to implement access to the UD4H Health Plugin API programatically, from inside a client application.  It is intended for technical users familiar with programming HTTP requests that send and return JSON.

## User Registration

To access the API, users must first register an email address and other information with UD4H.  Contact http://urbandesign4health.com/contact-us-3 to register.  All API requests returning EPAP data must contain a registered email address in a parameter called "clientid".


## EPAP Schema Metadata API
Base URL: http://api.ud4htools.com/hmapi_get_varmeta_json/EPAP/

HTTP Request Type: GET

### Request Parameters

Parameter | Description
--------- | -----------
None | n/a

### Response Output
EPAP schema is returned in JSON format, with the following key-value pairs for each field in the EPAP health module variant:

Key | Description
--- | -----------
ordinal_position | Default field order
hmvar | Field's case-sensitive name
hmvartype | Field data type as defined for PostgreSQL: https://www.postgresql.org/docs/9.3/static/datatype.html#DATATYPE-TABLE
description | Description of the field, with its data source indicated in brackets

### Example Usage

Copy/paste the following link into a browser: http://api.ud4htools.com/hmapi_get_varmeta_json/EPAP/

The above link will return the following JSON (snipped for brevity):
```
[
  {
    "ordinal_position": 1,
    "hmvar": "geoid10cbg",
    "hmvartype": "character varying",
    "description": "2010 Census block group ID (EPA SLD geoid10)"
  },
  {
    "ordinal_position": 2,
    "hmvar": "totpop2010",
    "hmvartype": "numeric",
    "description": "2010 Census total population (EPA SLD totpop10)"
  },
  {
    "ordinal_position": 3,
    "hmvar": "tothhs2010",
    "hmvartype": "numeric",
    "description": "2010 Census total households (EPA SLD hh)"
  },
< snipped for brevity >
  {
    "ordinal_position": 63,
    "hmvar": "mnt_health",
    "hmvartype": "double precision",
    "description": "Percent of population experiencing psychological distress (health survey model)"
  }
]
```

## Detail Data Request API
Base URL: http://api.ud4htools.com/hmapi_post_custom_inputs/EPAP/

HTTP Request Type: POST

### Request Parameters

Parameter | Description
--------- | -----------
clientid | Required. Email address associated with the client registration. See __User Registration__ section above. 
postjson | Required. JSON-formatted string containing at least a set of one or more 12-character GEOID10s.  This parameter can also contain other custom input values whose keys case-sensitive match the EPAP schema; use __EPAP Schema Metadata API__ (above) to get a list of field names in the EPAP schema.  The 'postjson' parameter must adhere to the GeoJSON "FeatureCollection" format; see __Example Usage__ in the Summary Request API (below) for an example of a 'postjson' parameter format.  Though 'postjson' otherwise needs to be in GeoJSON "FeatureCollection" format, the actual 'geometry' section (large one with polygon coordiantes) is not used and can be omitted to save request space.
baseonly | Optional.  If set to '1', directs API to ignore any custom inputs in the supplied postjson parameter and simply return all baseline data.

Detail request API calls are usually combined with Summary request calls (see below).  See __Example Usage__ in the Summary Request API at the end of this document for such a combined usage example.


### Response Output
a GeoJSON-formatted string is returned with two main branches:

1. A GeoJSON-formatted FeatureCollection section containing each CBG polygon feature in the request, attributed with EPAP schema properties (inputs and health outcomes, baseline and/or custom).
2. A __responseinfo__ section containing information about the request:

Key | Description
--- | -----------
requestid | An id associated with the request. The __Summary Data Request API__ (see below) requires requestid.
clientid | Email address associated with the client making the request.
hm_variant | Name of the health module variant requested, e.g. EPAP.
baseline_only_request | Boolean indicating that the server received a 'baseonly' of True.
custom_request | Boolean indicating that custom fields were detected by the API in the request's 'postjson' parameter
featurecount | Total number of census block groups in the request.
request_processing_time | Total time the API took to fulfill request. This total does not include processing time in the client before or after the request is made.
date_time_stamp | When the request was made.

Detail request API calls are usually combined with Summary request calls.  See __Example Usage__ in the Summary Request API (below) for such a combined example.



## Summary Data Request API
Base URL: http://api.ud4htools.com/hmapi_get_summary_json/EPAP/

HTTP Request Type: GET

### Request Parameters

Parameter | Description
--------- | -----------
clientid | Required. Email address associated with the client registration. See __User Registration__ section above. 
requestid | Required. The integer requestid returned from a previous call to the detail request API.  Use __Response Output__ section of __Detail Data Request API__ (above) for more information about requestid.  

### Response Output
a GeoJSON-formatted string is returned with two main branches:

1. A GeoJSON-compliant FeatureCollection section containing, for each feature in the  the polygonal 
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
Summary request API calls typically follow Detail request calls. The following code demonstrates an example where a json variable containing both GEOID10 values and other values whose keys match the EPAP schema are contained in the postjson parameter, i.e. a "custom" request.

```
custDetail = '{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature", 
      "properties": {
        "geoid10cbg": "060014003004", 
        "totemp2010": 111, 
        "totpop2010": 222, 
        "tothhs2010": 333
      }
    },
    {
      "type": "Feature", 
      "properties": {
        "geoid10cbg": "060014003003", 
        "totemp2010": 777, 
        "totpop2010": 888, 
        "tothhs2010": 999
      }
    }    
  ]
}';

$("body").css("cursor", "wait");

jsonDetail = JSON.parse(custDetail);

    $.post("/hmapi_post_custom_inputs/EPAP/",
      {
        postjson: JSON.stringify(jsonDetail),
        clientid: 'registered_email@client.com',
        baseonly: '0'
      },
      function(data,status){
    console.log("Response Status: " + status);
    customResponseType = data.type ;
    if (customResponseType == "FeatureCollection") {
      console.log("Valid JSON FeatureCollection returned.  Adding to CUSTOM map feature group");
      // add detail json to leaflet map directly. Very easy!
      // get requestid from json
      requestid = data.responseinfo.requestid ;

      // follow up with summary request
      $.getJSON("/hmapi_get_summary_json/EPAP?clientid=registered_email@client.com&requestid=" + requestid, function(data) { 
        summaryResponseType = data.type ;
        if (summaryResponseType == "FeatureCollection") {
          console.log("Valid JSON FeatureCollection returned.  Adding to SUMMARY map feature group");
          // add detail json to a leaflet map directly. Very easy!
        } else {
          console.log("summaryResponseType = " + summaryResponseType);
          alert("Error returned from " + data.ErrorSource + " API: " + data.Error);
        }
      });       

    } else {
      console.log("Error based on non-GIS customResponseType: " + customResponseType);
      alert("Error returned from " + data.ErrorSource + " API: " + data.Error);
    };
    $("body").css("cursor", "default");

    });

```

The following is a sample of the response output returned from a call to the detail request API:

```
{
   "type": "FeatureCollection",
   "responseinfo": {
      "hm_variant": "EPAP",
      "request_processing_time": "0.624 secs",
      "date_time_stamp": "2016-07-18 22:15:53.516710",
      "baseline_only_request": "False",
      "custom_request": "True",
      "clientid": "registered_email@client.com",
      "featurecount": "3",
      "requestid": "9876"
   },
   "features": [
      {
         "geometry": {
            "type": "MultiPolygon",
            "coordinates": [
               [ [ [
                      -89.303226235,
                      43.07796197
                   ],
                  < snipped for brevity >                    
                   [
                      -89.3032322349999,
                      43.07775997
                   ]
                ] ] ]
         },
         "type": "Feature",
         "properties": {
            "biketr_p_t": 0.0398637961257863,
            "totemp2010": 777,
            "pct_nohsed": 0.160287081339713,
            "walkle_d_h": 63381.9524348232,
            "retailempl": 57,
            "pct_childr": 0.194885361552028,
            "pct_worker": 0.696604600219058,
            "pct_higinc": 0.319796954314721,
           < snipped for brevity > 
            "autotr_p_t": 0.619032580022476,
            "geoid10cbg": "060250031004"
         },
         "id": "060250031004"
      }
   ]
}
```

The following is a sample of the GeoJSON returned from a summary request:
```
{
  "type": "FeatureCollection",
  "responseinfo": {
    date_time_stamp: "2016-07-18 23:18:11.664985",
    detail_feature_count: "2",
    custom_request: "True",
    request_processing_time: "0.345 secs",
    requestid: "2608",
    clientid: "registered_email@client.com"
  },
 "features": [
    {
       "geometry": {
          "type": "MultiPolygon",
          "coordinates": [
             [ [ [
                    -89.303226235,
                    43.07796197
                 ],
                < snipped for brevity >                    
                 [
                    -89.3032322349999,
                    43.07775997
                 ]
              ] ] ]
       },
       "type": "Feature",
       "properties": {
          "biketr_p_t": 0.343534543543,
          "totemp2010": 1254,
          "pct_nohsed": 0.8567744,
         "pct_higinc": 0.1233556435,
       < snipped for brevity > 
          "autotr_p_t": 0.657546463563
       }
    }
 ]
}
```



## Error Handling

If a problem with the request is encountered and therefore it cannot be fulfilled, instead of a GeoJSON-formatted response, a JSON-formatted error message will be returned.  All reponses from the API whould be immediately inspected in the client application to determine if such an error message was returned and to handle it appropriately.  The error message format is consistent.  THe following is an example request error message:

```
{
type: "Error",
Application: "HIAA",
ErrorSource: "hmapi_get_summary_json",
Error: "Error occured saving geoid10 list. Likely one of your geoid10 values is invalid.",
DateTimeStamp: "2016-07-18 23:13:22.183920"
}
```
