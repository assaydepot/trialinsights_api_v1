- [Portfolio](#portfolio)
- [Patient Report](#patient-report)


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
	api_key: 'string'
}
```
The `values` property describes the filters to apply.

Filters are applied sequentialy with the initial filter operating on the entire trials corpus and subsequent filters applied to the prior result. Thus each subsequent term is a logical "AND" with the prior term. 

Available values for the `name` property are: `other_terms, diseases, interventions, condition, condition_browse, keyword, drug, target, biomarker, lead_sponsor`. Using `other_terms` for `name` will search *all* fields listed here, plus the `brief_title` and `brief_summary` sections of the trial document. 


__Using any value other than `other_terms` will limit the search to only that field. This usually results in fewer, more specific results for the given query.__


__To search for trial sponsors you must use `lead_sponsor` as the value for `name`.__


Strings can be javascript regular expressions, i.e., `kras|egfr` or simple boolean expressions `"kras" or "egfr"`. Unless otherwise specified multiple words such as "breast cancer" are searched as phrases.

The `api_key` is a string supplied by TrialIO.

**Optional:**

```
feeds: 'private'
```
Unless `feeds:'private'` option is provided, the portfolio will be visible to the owner creating the portfolio and members of his group. 

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


