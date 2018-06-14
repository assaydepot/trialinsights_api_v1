# app.trialio.com API

- [Data Access](#data-access-api)
- [Aggregate Analysis](#aggregate-analysis-api)
- [Portfolio Management](#portfolio)
- [Patient Reports](#patient-report)

## Analysis and Applications for Content at [ClinicalTrials.gov](http://clinicaltrials.gov)
Application developers can use the API for programmatic access to all clinical trial data. The API provides for a __Data Access API__ and an __Aggregate Analysis API__ for clinical trial analytics. Clinical trial "facets" and trial "documents" represent the data and a data "profile" interface provides accees to analytics. Both aggregate analytics and trial content fetches are supported by a very powerful query object format, allowing for sensitive and precise control over the documents analyzed or returned from any request.

## Portfolio Management
The __Portfolio Management API__ allows organizing related queries for bulk computation, access control, and monitoring. 

## Patient Reports
The __Patient Reports API__ provides a mechanism to email a set of results to a recipient as either a link back to the report in the TrialIO application or a formatted PDF report.

## Authentication
Access to app.trialio.com is relies on `api_key` property in the body of the *POST* or as a url query parameter `?api_key=key`. To generate a key go to settings for your account in the app and scroll down to the bottom. There you copy/paste the generated key.

## Data Access API
The Data Access consists of _facets_ and trial _documents_. Facets are used to populate menus and picklists throughout a client application. Facet _keys_ are the available fields in the CT.gov archive, for example "lead_sponsor" or "condition". Facet _values_ are the intance value for a given key, such as "Pfizer" or "heart disease" in our example.

Certain facets such as "Facility" and "Investigators" have a large number of unique values, thus the the picklists can get quite large. The API provides a summary and a detail endpoint for retrieving facet names and the number of values for each.

All endpoints are available at https://app.trialio.com/api/v1

### GET	/facets?api_key=key
Returns an overview of available facet “keys”.
```
// Example response
	['diseases', 
	'interventions', 
	'lead_sponsor',
	'drug',
	'target',
	'biomarker']
```
### GET /facets/:facet?api_key=key
Returns Picklist of available facet “values” for requested :facet
```
// Example query /facets/biomarker?api_key=key
{"biomarker":[
	"123i-ibvm",
	"18f-fluorodeoxyglucose (fdg)","25-oh vitamin d",
	"5-hiaa",
	"8-isoprostane", ... ]}

```

### POST /trials
Returns an array of trial IDs meeting search criteria. The query object allows specification of an array of objects. If more than 1 property is provided within the object, then ALL of the key/values of that object have to be met for the document to be selected. If multiple array elements are provided to find or filter, then ANY object where ALL key/values are a match will result in that document being "found" or "filtered". 

#### Request
```
// Example Request object
```
{
	name: 'string',
	description: 'string',
	format: {values: [{name: 'other_terms', value:'amnesia'},{name:'phase', value: 'phase1|phase2']},
	api_key: 'string'
}
```
// This query will find all phase 1 or phase 2 trials with "amnesia". documents matching either find criteria. The documents will
// be ordered descending by `last_update_posted`
```
#### Response
The response to the above request is an object as follows:
```
{
  queryId: 'string',
  nct_ids: 'array' of clinical trial identifiers,
  recordsTotal: 'number' of total trials matching primary search term,
  recordsFiltered: 'number' of trials after filter term(s) applied,
  activity_history: 'object' with statistics,
  card_values: 'object' with facet counts
}
```

### GET /:id/trials?start={start}&length={length}
Returns an array of requested trial documents or an individual trial document.

If the url parameter `:id` is the `queryId` from a prior *POST* response object. If `:id` resolves to a clinical trial identifier, that trial document will be returned.

When `:id` is a `queryId` Use `start` and `length` to request a specific page of documents.

#

## Aggregate Analysis API
The aggregate analysis API builds on the data access API. However, in lieu of trial lists and documents, the endpoints calculate aggregate values for trial documents containing 

### GET /:queryId/profile?api_key=key
Returns the response object returned from the `/trials` endpoint.
```
// Example card_values response
	"card_values": {
		"agency_class": 4,
		"completion_date": 469,
		"diseases": 258,
		"has_expanded_access": 1,
		"intervention_type": 15,
		"interventions": 2503,
		"lead_sponsor": 647,
		"location_countries": 69,
		"overall_status": 12,
		"phase": 8,
		"site": 5062,
		"start_date": 492,
		"structured_eligibility": 6,
		"study_design_info": 8,
		"study_type": 4,
		"collaborator": 594,
		"drug": 249,
		"target": 147,
		"biomarker": 40,
		"biospec_retention": 3
	}
```

### GET /:queryId/profile?api_key=key[&facet=facet]
Returns the detail aggregate analysis of facet value counts found in trial documents connected to `queryId`. 

If you omit the facet_key specification, the endpoint will return an array of objects for ALL facet keys. To request a single facet provide the facet_key as part of the URL.

#### Response
An example response object for `agency_class` is shown below:
```
// Example /:queryId/profile/api_key=key&facet=agency_class
{
   "card":"agency_class",
   "draw":"0",
   "queryId":"NbaJRFRk486DaNchY",
   "created_at":"2018-05-24T14:08:48.360Z",
   "agency_class":[
      {
         "value":"Other",
         "count":1261,
         "name":"agency_class"
      },
      {
         "value":"Industry",
         "count":639,
         "name":"agency_class"
      },
      {
         "value":"NIH",
         "count":91,
         "name":"agency_class"
      },
      {
         "value":"U.S. Fed",
         "count":40,
         "name":"agency_class"
      }
   ]
}
```

# Portfolio
**Create Portfolio**
----
Create portfolio of clinical trial queries.
* **URL**

  /api/v1/portfolio

* **Method**

  POST
  
*  **URL Params**

	None.

* **Data Params**

**Required:**

```
{
	name: 'string',
	description: 'string',
	values: [{name: 'other_terms', value:'amnesia'}...],
	portfolio: 'string',
	feeds: 'private',
	api_key: 'string'
}
```
`name` is a string describing the query.

`description` is an optional string with additional information to be shown on the portfolio landing page.

`values` is a required property describes the filters to apply when searching. It is an array where each element of the array is an object with two properties: `name` and `value`.

The `name` describes _where_ to search in the trial document.

The `value` describes _what_ to search. Strings can be javascript regular expressions, i.e., `kras|egfr` or simple boolean expressions `"kras" or "egfr"`. Unless otherwise specified multiple words such as "breast cancer" are searched as phrases.

Filters are applied sequentialy with the initial filter operating on the entire trials corpus and subsequent filters applied to the prior result. Thus each subsequent term is a logical "AND" with the prior term. 

Available values for the `name` property are: 

```
other_terms
diseases
interventions
condition
condition_browse
keyword
drug
target
biomarker
lead_sponsor
```

Using `other_terms` for `name` will search *all* fields listed here, plus the `brief_title` and `brief_summary` sections of the trial document. 

Using any value other than `other_terms` will limit the search to only that field. This usually results in fewer, more specific results for the given query.

To search for trial sponsors you must use `lead_sponsor` as the value for `name`.

The `feeds` optional property indicates whether to keep the portfolio private to the requestor, or allow other members of the group to access it from the TrialIO app.

Unless `feeds:'private'` option is provided, the portfolio will be visible to the owner creating the portfolio and members of his group. 

The `api_key` is a string supplied by TrialIO.

* **Success Response:**

The successful response is a JSON object with the identifier of the generated query. 

```
{id: 'string'}
```
Upon entering the TrialIO web application, users with privilege to this portfolio can navigate to the portfolio using the dropdown widget on the `Content` page of the app.

* **Error Response:**

An error response will include a statusCode >= 400 and a JSON object with `error` and `message` properties.

```
{error: 'failed', message:'Request denied.'}
```

* **Example List of POST objects**

An sample array of POST objects is found [here](https://raw.githubusercontent.com/rranauro/trialio/master/rare_disease.json)

> Remember: The endpoint is asynchronous so that each request should receive its response before sending the next request.


**Delete Portfolio**
----
Create portfolio of clinical trial queries.
* **URL**

  /api/v1/portfolio

* **Method**

  DELETE
  
*  **URL Params**

	?api_key=key&portfolio=name

The `api_key` is a string supplied by TrialIO.

The `portfolio` is a string name of the portfolio to remove.

* **Data Params**

	None.

* **Success Response:**

The successful response is a JSON object with two properties: `ok` and `deleted`.

```
{ok: true, deleted: 17}
```

* **Error Response:**

An error response will include a statusCode >= 400 and a JSON object with `error` and `message` properties.

```
{error: 'failed', message:'Request denied.'}
```

# Patient Report
**Printable Referral Report**
----
Returns HTML, PDF, or JSON list of trials for given disease and geographic location.

* **URL**

  /api/v1/publication

* **Method:**
  
  `POST`
  
*  **URL Params**

	None.

* **Data Params**

**Required:**

```
values=[{
	name: "diseases",
	value: "lung cancer"
},{
	name: "overall_status",
	value: "recruiting"
}];
```
	
Filters are applied sequentialy with the initial filter operating on the entire trials corpus and subsequent filters applied to the prior result. Thus each subsequent term is a logical "AND" with the prior term. 

Strings can be javascript regular expressions, i.e., `kras|egfr` or simple boolean expressions `"kras" or "egfr"`. Unless otherwise specified multiple words such as "breast cancer" are searched as phrases.

```
api_key=[string provided by TrialIO]
```

Supplied by TrialIO.
		
* **Optional**

```
location={ object containing country, city, state properties}
start=[integer first record]
length=[integer length of page requested]
output=[string either "html" or "json"]
```

* **Example**
```
// example POST body: will search trials for "breast cancer" and filter for keywords "egfr OR kras" within 200 miles of Worcester Ma.
{
	values: [{name: "other_terms", value: "breast cancer"},{name: "other_terms", value: "egfr|kras"}],
	location: {country: "united states", city: "worcester", state: "massachusetts"},
	output: "html",
	start: 0,
	length: 100,
	api_key: "abc0123456789xyz"
}
```	

* **Success Response:**

The successful response is a JSON object. The object includes either the JSON or formatted HTML string as requested. The `recordsTotal` is the number of trials found _before_ any subsequent filters are applied. `recordsFiltered` is the number of remaining trials _after_ filtering. `sitesAvailable` are the number of trial-site pairs since a given trial can have zero or more sites connected to it.

The geographic sort is applied _after_ all filters are applied. The response includes trials that are found within 200 miles (350kM) of the reference location specified.

The `sitesAvailable` figure is the limit for paging. 

All content-type response headers are 'application/json'.

```
// => HTML response
{
	html: "<html ...",
	queryId: "unique identifier",
	recordsTotal: 1627,
	recordsFiltered: 301,
	sitesAvailable: 287,
	start: 0,
	length: 100
}
```

```
// => JSON response
{
	json: {
		location: {country:"united states", city:"worcester", state:"massachusetts"},
		pretty: '( "breast cancer" ) AND ( "egfr" OR "kras" )',
		trials: [{ trial 1}, {trial 2} ...]
	},
	queryId: "unique identifier",
	recordsTotal: 1627,
	recordsFiltered: 301,
	sitesAvailable: 287,
	start: 0,
	length: 100
}
```


  * **Code:** 201 <br />
    **Content:**
    	`{ html: "<html ...", queryId: "unique identifier"}`
 
* **Error Response:**

  * **Code:** 401 UNAUTHORIZED <br />
    **Content:** `{ error : "Authentication Failed." }`

  OR

  * **Code:** 422 UNPROCESSABLE ENTRY <br />
    **Content:** `{ error : "Invalid Request", message: "<property name> not recognized." }`

* **Sample Call:**

```
	$.ajax({
		url: "https://app.trialio.com/api/v1/publications",
		dataType: "json",
		type : "POST",
		data: {
			values: [{name: "other_terms", value: "breast cancer"},{name: "other_terms", value: "egfr|kras"}],
			location: {country: "united states", city: "worcester", state: "massachusetts"},
			output: "html",
			start: 0,
			length: 100,
			api_key: "abc0123456789xyz"		
		},
		success : function(r) {
			console.log(r);
		}
	});
```

**Notes:**

- May 21, 2018

The `values` property is defined to be an array of `name/value` pairs.

- April 17, 2018

The *api_key* is acquired manually.

The requestors logo can be substituted for the TrialIO logo in the HTML and PDF versions by prior arrangement.

The requestors *name* and *affiliation* used in the report can be added to the request if HTML or PDF responses are requested. Otherwise, the user and affiliation of the owner of the *api_key* will be used for those outputs.

If an unrecognized country/city/state is requested, the location search will not work. An API for confirming coordinates is planned.

#
app.trialio.com and trialio.com are trademarks of Incite Advisors, Inc.
