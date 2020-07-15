# Trial Insights API
Search and Analysis API for Trial Insights Clinical Trial Search, Intelligence, and Surveillance

## Overview
The API is comprised of a [Trials API](#data-access-api) for querying clinical trials and rendering trial documents, an [Investigators API](#investigator-access-api) with extended annotation for clinical trial investigators, and an [Aggregate Analysis API](#aggregate-analysis-api) for analyzing and computing statistics over collections of clinical trials and related investigators.

### Authentication
Access to https://app.trialiinsights.com/api/v1/ endpoints requires an `api_key`. Request an API Key at: [Trial Insights API Key Request](mailto:ron@scientist.com)

Once you have the key, submit your request using the `Trialinsights-Api-Key` custom header. Alternatively, you can include the property in the body of the *POST* `{api_key: "provided-key"}` or as a url query parameter `?api_key=key`. 

All endpoints are available at https://app.trialinsights.com/api/v1

## Trials API
Use the __Trials API__ to submit queries for relevant clinical trials and receive sorted, JSON formatted documents to be rendered by your application. In addition to the initial page of results the response object includes `recordsTotal` and the `nct_ids` array of clinical trial identifiers. Query parameters inclue options for sorting, paging, and selecting only requested fields.

### POST /find
Post an object containing a `values` property which is an array of terms. Each term consists of a `name` and a `value`. Each entry in the array is an object with two properties: `name` and `value`.  

A `name` is a string value corresponding to a valid __primary search__ or __secondary search__ name shown in the [table below](#primary-search-and-secondary-search-names). The server responds with an object with a `recordsFiltered` number for the total documents found and `nct_ids` array of trial identifiers. Use this information to request pages of documents using the `/fetch` endpoint.

A `value` is your keyword(s) or a valid value of a clinical trial property such as `Phase 1` for _Phase_. 

```
// The following example values entry returns 
// trials with breast cancer in any text field.
{
  values: [{
    name: "other_terms",
    value: "breast cancer|prostate cancer"
  }]
}

// The following example values entry returns 
// trials with breast cancer OR prostate cancer found in diseases section of the trial.
{
  values: [{
    name: "diseases",
    value: "breast cancer|prostate cancer"
  }]
}

// The following example values entry returns 
// trials with breast cancer AND prostate cancer found in diseases section of the trial.
{
  values: [{
    name: "diseases",
    value: "breast cancer"
  },{
    name: "diseases",
    value: "prostate cancer"
  }]
}


// The request response is an object as follows:
{
  nct_ids: 'array' of trial identifiers,
  recordsTotal: 'number' of total trials matching first search term,
}
```

##### Request Errors
A request that produces no results will return a statusCode 200 but will have an empty array in the `nct_ids` property of the response object and -1 `recordsTotal`.

In most cases, the server will respond with a statusCode >= 400 and a JSON object with `error` and `message` properties if the request was issued impropertly.

```
// Example error response:
{error: 'failed', message:'Request denied.'}
```

##### Primary Search and Secondary Search Names
The valid strings for the `name` property of a name/value object is shown in the table here:

| name  | Type | Behavior |
| ------------- | ------------- | --------------------------------------- |
| other_terms | Primary | Searches all relevant text fields in trial document looking for matches, including title and description. |
| diseases | Primary | Searches condition, condition_browse, and keyword fields for matches. |
| interventions | Primary | Searches intervention, intervention_browse, and keyword fields for matches. |
| drug | Primary | Searches intervention, intervention_browse, keyword and other_names for matches. Note: other_names is extended by Trialinsights and will find matches to drug synonyms found at [PubChem](https://pubchem.ncbi.nlm.nih.gov/) and [Therapeutic Target Database](https://db.idrblab.org/ttd/) |
| phase  | Secondary | Filter results by valid clinical trial phase.  |
| overall_status  | Secondary | Filter results by valid clinical trial overall status.  |
| intervention_type | Secondary | Filter results by valid clinical trial intervention type. |
| study_type | Secondary | Filter results by valid clinical trial study type. |
| agency_class | Secondary | Filter results by valid clinical trial agency class designation. |

Primary search value strings are free text and secondary search value strings are controlled filters using the vocabulary of ClinicalTrials.gov. 

__The values array must include at least 1 primary search name/value and can have zero or more secondary search name/values.__

```
// Example invalid request
// Request must include at least 1 term with primary search name value. 
{
  values: [{
    name: "phase",
    value: "Phase 2"
  }]
}
```

###### When to use "other_terms" as a Primary Search term
Specifying `name: "other_terms"` causes the search engine to include additional text fields found in trial documents in its effort to find trials. Additional fields include the `title` and `description` whereas limiting the name to `drug` for example will restrict the search to only the `intervention`, `other_names`, and `keyword` areas of the trial documents. 

__For maximum recall, use `other_terms`. For more specific recall, use the appropriate primary search `name` specifier such as `diseases` when searching disease or `drug` when searching using drug names.__

If more than 1 Primary Search term is provided within the `values` array object, matches result when __all__ terms are found in the trial document. All of the name/value terms have to be met for the document to be selected.

For __primary search__ terms, any string in the associated field of the trial document containing the supplied string will trigger a match, including a string describing a JavaScript regular expression. For __secondary search__ terms, only strings listed in [Valid Values](#valid-values) in the table below will trigger matches.

###### Valid Values
| name  | Valid Values |
| ------------- | ------------- |
| phase  | Phase 1, Phase 2, Phase 3, Phase 4, Phase 1/Phase 2, Phase 2/Phase 3, Early Phase 1, N/A  |
| overall_status | 'Active, not recruiting', Completed, Enrolling by invitation, Not yet recruiting, Recruiting, Suspended, Terminated, Withdrawn |
| intervention_type | Behavioral, Biological, Combination Product, Device, Diagnostic Test, Dietary Supplement, Drug, Genetic, Procedutre, Radiation, Other |
| study_type | Expanded Access, Interventional, N/A, Observational, Observational [Patient Registry] | | agency_class | NIH, 'U.S. Fed', Industry, Other |

__Note: Value strings are case in-sensitive.__

###### Multiple entries for a given Secondary Search term.
Multiple entries for a secondary search term will form a logical "OR", as shown in the example below:

```
// Example Request Body
// This query will find all `Phase 1` or `Phase 2` _breast cancer_ trials whose overall status is `Recruiting`. 
// The default document ordering is by `study_first_posted`
// Note: value strings are case-in sensitive.
{
  values: [{
    name: 'other_terms', value:'breast cancer'
  },{
    name:'phase', value: 'phase 1'
  },{
    name: 'phase', value: 'phase 2'
  },{
    name:'overall_status', value:'Recruiting'
  }]
}
```
### POST /fetch
Post a document with an array of `nct_ids` to request associated trial documents. The response object from `/find` is compatible as input to `/fetch`. 

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
	agency_class: 1,
	overall_status: 1,
	phase: 1,
	intervention_type: 1,
	study_type: 1,
	biospec_retention: 1,
	biospec_retention: 1,
	study_type: 1,
	has_expanded_access: 1,
	study_design_info: 1,
	structured_eligibility: 1,

	number_of_arms: 1,
	enrollment: 1,
	location: 1,
	location_countries: 1,
	duration: 1,
	completion_date: 1,
	primary_completion_date: 1,
	start_date: 1,
	study_first_posted: 1,
	last_update_posted: 1,
	results_first_posted: 1,

	registry: 1,
	source_registry: 1,

	lead_sponsor: 1,
	collaborator: 1,
	condition_browse: 1,
	intervention_browse: 1,
	keyword: 1,
	drug: 1,
	other_names: 1,
	target: 1,
	biomarker: 1,
	investigator: 1
}

```

### POST /trials
The `/trials` endpoint combines `/find` and `/fetch` in one shot. 

``

// Example request
$.ajax({
	url: "https://app.trialinsights.com/api/v1/trials",
	dataType: "json",
	type : "POST",
	data: {
		values: [{name: "other_terms", value: "breast cancer"},{name: "other_terms", value: "egfr|kras"}],
		location: {country: "united states", city: "worcester", state: "massachusetts"},
		output: "json",
		start: 0,
		length: 10,
		api_key: "abc0123456789xyz"		
	},
	success : function(r) {
		console.log(r);
	}
});
```

###### Location sorting
To get trials sorted by distance from a location, use `sort: "distance"` and include a `location` property. Results will be sorted in the `data` response based on the distance from from nearest to farthest from the requested location. Resulting trial documents will have a `sites` property. `sites` is an array of objects where each object includes the detail for the participating site, including the responsible contacts at the site, if available at ClinicalTrials.gov.

```
// Example sites property of response object
{
  recordsTotal: 24,
  recordsFiltered: 24,
  docs: [{
    nct_id: 'NCT0001234',
    brief_title: 'A Clinical Trial...',
    conditions: ['condition1', 'condition2'...],
    ...
    sites: [{
      site: 'site name',
      city: 'city',
      state: 'state',
      country: 'country',
      contacts: [{
        firstname: 'firstname',
	lastname: 'lastname',
	phone: '111-234-5678',
	email: 'some@email.com'
      }{
        ...
      }]
    }]
  },{
     ...
  }]
}
```


### GET /:id/trials
Returns an individual trial document.

If `:id` resolves to a clinical trial identifier, that trial document will be returned.

### GET	/facets
Returns an overview of available facet “keys”.

Facets are used to populate menus and picklists throughout a client application. Facet _keys_ are the available fields in the CT.gov archive, for example "lead_sponsor" or "condition". Facet _values_ are the intance value for a given key, such as "Pfizer" or "heart disease".

Certain facets such as "Facility" and "Investigators" have a large number of unique values, thus the the picklists can get quite large. The API provides a summary and a detail endpoint for retrieving facet names and the number of values for each.

```
// Request: /facets
// Response:
["diseases", 
"interventions", 
"lead_sponsor",
"drug",
"target",
"phase"...]
```
### GET /facets/:facet

Returns Picklist of available facet “values” for requested :facet and global counts of occurrence in trials.
```
// Request: /facets/phase
// Response:
{'Phase 1': 267, 'Phase 2': 213, 'Phase 3': 411, ...}
```

## Aggregate Analysis API
The aggregate analysis API builds on the data access API. However, in lieu of trial lists and documents, the endpoints calculate aggregate values for entire collections of trial documents resulting from searches. 

### POST /profile[?facet=facet]
Use the request object for `/trials` endpoint to access additional properties about the aggregate analysis of the trials that met the search criteria.

```
// Example card_values response
{
  nct_ids: [ array of trial identifiers ],
  activity_history: { object of time series and aggregate statistics about the set of trials },
  card: { object of aggregate statistics for each facet and its values },
  card_values: {
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
}

```
Query using the `?facet=` parameter to return the detail aggregate analysis of facet value counts found in trial documents connected to `queryId` for the specific facet.

```
// Example /:id/profile?facet=agency_class
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

### POST /portfolio
Add a query to a portfolio named `portfolio` of results.

| name  | type | value |
| ------------- | ------------- | --------------------------------------- |
| portfolio | String | The name of the portfolio. POST multiple queries with the same `portfolio`string to create a group of associated queries. |
| description | String | A string describing the portfolio. |
| feeds | String or Array | An array of email addresses of subscribers eligible to view and receive alerts regarding changes to this portfolio. If string 'private' is provided, the portfolio will be private to the owner creating the portfolio. |

An sample array of POST objects is found [here](https://raw.githubusercontent.com/rranauro/trialio/master/rare_disease.json)

> Remember: The endpoint is asynchronous so that each request should receive its response before sending the next request.

The successful POST returns `ok` and `id` of the portfolio` object.
```
// Example response object:
{ok: true, id: "12345678"}
```
An error response will include a statusCode >= 400 and a JSON object with `error` and `message` properties.
```
// Example error response:
{error: 'failed', message:'Request denied.'}
```
### GET /portfolio?portfolio=name
Initiate a re-computation of an existing portfolio of queries where `name` is a string name of the portfolio to compute.

The successful response is a JSON object with two properties: `ok` and `collections`.

```
// Example response object:
{ok: true, collections: 17}
```
An error response will include a statusCode >= 400 and a JSON object with `error` and `message` properties.

```
// Error response object:
{error: 'failed', message:'Request denied.'}
```

### DELETE /portfolio?portfolio=name
Removes all entries with `portfolio=name` where `name` is a string name of the portfolio to remove.

__This operation cannot be reversed.__

The successful response is a JSON object with two properties: `ok` and `deleted`.

```
// Example success response object.
{ok: true, deleted: 17}
```

**Notes:**
- July 14, 2020
Removed `queryId` from POST `/trials` response object and GET `/trials` request.
Updated default `fields`.
Response object is only JSON.

- December 16, 2018

Consolidated search API with output property. 

- June 14, 2018

Updated and merged Data and Aggregate API docs with Portfolio and Patient Report API doc.

- May 21, 2018

The `values` property is defined to be an array of `name/value` pairs.

- April 17, 2018

The *api_key* is acquired manually.

The requestors logo can be substituted for the Trialinsights logo in the HTML and PDF versions by prior arrangement.

The requestors *name* and *affiliation* used in the report can be added to the request if HTML or PDF responses are requested. Otherwise, the user and affiliation of the owner of the *api_key* will be used for those outputs.

If an unrecognized country/city/state is requested, the location search will not work. An API for confirming coordinates is planned.

#
app.trialinsights.com and trialinsights.com are trademarks of Incite Advisors, Inc. and Scientist.com
