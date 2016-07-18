# UD4H Health Plugin API 

## Overview
As part of its ongoing work on the Health Impact Assessment Application (HIAA), UD4H (http://www.ud4h.com) has created a dataset called EPAP ("EPA Prime") that contains, for each of the 220,653 census block groups in the United States, a set of baseline inputs and health outcomes derived from these inputs.  

The baseline inputs are derived from census block group (CBG) polygons attributed with data fields from the Smart Location Database (SLD), the National Land Cover Database (NLCD) and from other census data.  The health outcomes are calculated using coefficients derived from analysis related to indepth research sources like the American Community Survey (ACS).

Access to this dataset and its calculated outcomes is available via a cloud-based web-served health module application programming interface (API).  The API responds to three types of requests:

1. Metadata requests.  The API returns a JSON list of all the input and health outcome fields in the EPAP health module variant's data model.  Contains descriptions.

2. Read-only "baseline" requests containing one or more 12-digit CBG unique identifier codes (called GEOID10).  The API returns EPAP baseline inputs and health outcomes attached to a feature collection in GeoJSON format. 

3. Custom-input requests containing CBG GEOID10s plus, for each GEOID10, one or more fields containing custom input values for one or more of the EPAP input fields.  The API returns the custom inputs, EPAP baseline inputs for any inputs missing from the request, and custom health outcomes calculated using the custom inputs.  These custom results are attached to a feature collection in GeoJSON format. 

This document is intended to describe the use of the HIAA API by client applications.  It is intended for technical users familiar with programming HTTP requests that send and return JSON.


## User Registration

To access the API, users must first register an email address and other information with UD4H.  Contact http://urbandesign4health.com/contact-us-3 to register.  All API requests returning EPAP data must contain a registered email address in a parameter called "clientid".


## EPAP Metadata API

Base URL: http://api.ud4htools.com/hmapi_get_varmeta_json/EPAP/

### Request Parameters

Parameter | Description
--------- | -----------
None | n/a

### Response Output
JSON is returned with the following key-value pairs for each field in the EPAP health module variant:

Key | Description
--- | -----------
ordinal_position | Default field order
hmvar | Field's case-sensitive name
hmvartype | Field data type as defined for PostgreSQL: https://www.postgresql.org/docs/9.3/static/datatype.html#DATATYPE-TABLE
description | Description of the field, with its data source indicated in brackets

### Example Usage

Copy/paste the following link into a browser: http://api.ud4htools.com/hmapi_get_varmeta_json/EPAP/

The above link will return the following JSON (truncated below):
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
  {
    "ordinal_position": 4,
    "hmvar": "p_wrkage",
    "hmvartype": "numeric",
    "description": "Percent of population that is working age (EPA SLD p_wrkage)"
  },
_< snipped for brevity >_
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

### Request Parameters

Parameter | Description
--------- | -----------
clientid | Email address associated with the client registration.  See __User Registration__ section above. 

### Response Output
JSON is returned with two main branches:

1. A GeoJSON-compliant FeatureCollection section containing
2. A __responseinfo__ section containing information about the request just made:

Key | Description
--- | -----------
requestid | An id associated with the request and required by the __Summary Data Request API__ (below)
clientid | Email address associated with the client making the request

### Error Messages

#### Missing GEOID10s
Here's a description.
