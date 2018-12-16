# app.trialio.com API

## Overview
The API is comprised of a [Data Access API](#data-access-api) for querying clinical trials and rendering trial documents and an [Aggregate Analysis API](#aggregate-analysis-api) for analyzing and computing statistics over collections of clinical trial documents.

### Authentication
Access to app.trialio.com is relies on `api_key` property in the body of the *POST* or as a url query parameter `?api_key=key`. To generate a key go to settings for your account in the app and scroll down to the bottom. There you copy/paste the generated key.

## Data Access API
Use the __Data Access API__ to submit queries for relevant clinical trials and receive sorted, JSON formatted documents to be rendered by your application. In addition to the initial page of results, the API returns a query identifier used on subequent requests to to control paging, sorting, and requested fields of documents to be rendered by your app. 

### POST /trials
Returns an object containing an array for the initial page of trial documents meeting search criteria and some meta information to allow the client app to request subsequent pages. 

#### Request Body
The only required propery is a `values` array of objects, i.e., "terms" each with `name` and `value` properties. __The values array must include at least 1 primary search name and can have zero or more secondary search names.__

If more than 1  is provided within the object, then __ALL__ of the name/values terms have to be met for the document to be selected. 

A `name` is a string value corresponding to a valid __primary search__ or __secondary search__ term shown in the tables below.

A `value` is a string or a be a valid JavaScript regular expression. In the latter case, the regular expression will be applied to find matches for the field. In this way complex AND/OR expressions can be requested. 

```
// Example Request Body
// This query will find all `Phase 1` or `Phase 2` _breast cancer_ trials whose overall status is `Recruiting`. 
// The default document ordering is by `study_first_posted`
{
  values: [{name: 'other_terms', value:'breast cancer'},{name:'phase', value: 'phase1|phase2'},{name:'overall_status', value:'Recruiting'}]
}
```
The valid `name` __primary search__ names are:
- other_terms
- diseases
- interventions
- drug

| name  | value |
| ------------- | --------------------------------------- |
| other_terms | Searches all relevant text fields in trial document looking for matches, including title and description. |
| diseases | Searches condition, condition_browse, and keyword fields for matches. |
| interventions | Searches intervention, intervention_browse, and keyword fields for matches. |
| drug | Searches intervention, intervention_browse, keyword and other_names for matches. Note: other_names is extended by TrialIO and includes drug synonyms found at [PubChem](https://pubchem.ncbi.nlm.nih.gov/) and [Therapeutic Target Database](https://db.idrblab.org/ttd/) |

The valid `value` specification is a search string of valid JavaScript regular expressions. For example, `breast cancer|prostate cancer` returns trials with breast cancer OR prostate cancer in the requested field.

> Specifying `name: "other_terms"` causes the search engine to include additional text fields in trial documents in its recall such as the `title` and `description` whereas limiting the name to `drug` for example will search only the `intervention` and `keyword` areas of the trial documents. For maxumum recall, use `other_terms`. For more specific recall, use the appropriate `name` specifier.

The valid `name` __secondary search__ names and allowed 'value` specifications are:

| name  | value |
| ------------- | --------------------------------------- |
| phase  | Phase 1, Phase 2, Phase 3, Phase 4, Phase 1/Phase 2, Phase 2/Phase 3, Early Phase 1, N/A  |
| overall_status  | 'Active, not recruiting', Completed, Enrolling by invitation, Not yet recruiting, Recruiting, Suspended, Terminated, Withdrawn   |
| intervention_type | Behavioral, Biological, Combination Product, Device, Diagnostic Test, Dietary Supplement, Drug, Genetic, Procedutre, Radiation, Other |
| study_type | Expanded Access, Interventional, N/A, Observational, Observational [Patient Registry] |
| agency_class | NIH, 'U.S. Fed', Industry, Other |

#### Response
The response to the above request is an object as follows:
```
{
  queryId: 'string' identifier used for subsequent requests for documents,
  data: 'array' of trial documents,
  recordsTotal: 'number' of total trials matching first search term,
  recordsFiltered: 'number' of trials after subsequent term(s) applied
}
```
##### Additional Request Properties
There are a number of optional properties developers can specify to control the response to the `/trials` POST request.

| Request Property  | Type | Default | Description |
| ------------- | ---------- | ---------- | --------------------------------------- |
| start | Number | 0 | The index of the first requested document from the search result |
| length | Number | 10 | The number of documents requested by the client. Use -1 to request all. |
| sort | object | See below | An object describing the request sort behavior. The default is a descending `desc` sort. Specify  `direction: "asc"` to sort ascending. Additional sort values are: last_update_posted, start_date, completion_date |
| location | object | null | An object specifying `country`, `city`, `state`. By specifying a `location` the sort request property is ignored and the resulting list of trials is sorted in ascending geographic distance from the supplied location. |
| fields | object | See below | An object specifying which fields to include for each document in the response object. |

```
// Default sort object
{sort:'study_first_posted', direction: 'desc'}

// Default fields object
{
  nct_id: 1,
  brief_title: 1,
  phase: 1,
  overall_status: 1,
  sponsors: 1,
  enrollment: 1,
  drug: 1,
  other_names: 1,
  target: 1,
  condition: 1,
  intervention_name: 1,
  agency_class: 1,
  study_type: 1,
  intervention_type: 1,
  primary_outcome: 1,
  last_history_update: 1,
  last_update_posted: 1,
  last_update_submitted: 1,
  study_first_submitted: 1,
  study_first_posted: 1,
  start_date: 1,
  completion_date: 1
}
```



Search and Analysis API for Content at [ClinicalTrials.gov](http://clinicaltrials.gov)



Application developers can use the API for programmatic access to all clinical trial data. The API provides for a __Data Access API__ and an __Aggregate Analysis API__ for clinical trial analytics. 

Clinical trial "facets" and trial "documents" represent the data and a data "profile" interface provides accees to analytics. Both aggregate analytics and trial content fetches are supported by a very powerful query object format, allowing for sensitive and precise control over the documents analyzed or returned from any request.

## Portfolio Management
The __Portfolio Management API__ allows organizing related queries for bulk computation, access control, and monitoring. 

## Patient Reports
The __Patient Reports API__ provides a mechanism to email a set of results to a recipient as either a link back to the report in the TrialIO application or a formatted PDF report.

- [Data Access](#data-access-api)
- [Aggregate Analysis](#aggregate-analysis-api)
- [Portfolio Management](#portfolio)
- [Patient Reports](#patient-report)


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
Delete portfolio of clinical trial queries.
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
**Compute Portfolio**
----
Force a re-computation of an existing portfolio of clinical trial queries.
* **URL**

  /api/v1/portfolio

* **Method**

  GET
  
*  **URL Params**

	?api_key=key&portfolio=name

The `api_key` is a string supplied by TrialIO.

The `portfolio` is a string name of the portfolio to remove.

* **Data Params**

	None.

* **Success Response:**

The successful response is a JSON object with two properties: `ok` and `deleted`.

```
{ok: true, collections: 17}
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

- June 14, 2018

Updated and merged Data and Aggregate API docs with Portfolio and Patient Report API doc.

- May 21, 2018

The `values` property is defined to be an array of `name/value` pairs.

- April 17, 2018

The *api_key* is acquired manually.

The requestors logo can be substituted for the TrialIO logo in the HTML and PDF versions by prior arrangement.

The requestors *name* and *affiliation* used in the report can be added to the request if HTML or PDF responses are requested. Otherwise, the user and affiliation of the owner of the *api_key* will be used for those outputs.

If an unrecognized country/city/state is requested, the location search will not work. An API for confirming coordinates is planned.

#
app.trialio.com and trialio.com are trademarks of Incite Advisors, Inc.
