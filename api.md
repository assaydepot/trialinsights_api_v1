**Printable Referral Report**
----
Returns HTML or JSON list of trials for given disease and geographic location.

* **URL**

  /api/v1/publication

* **Method:**
  
  `POST`
  
*  **URL Params**

	None.

* **Data Params**

	**Required:**

	`values=[ array of query strings ]`
	
	`api_key=[string provided by TrialIO]`
		
	**Optional**
	
	`location={ object containing country, city, state properties}`
	
	`start=[integer first record]`
	
	`length=[integer length of page requested]`
	
	`output=[string either "html" or "json"]`

```
// example body search "breast cancer" and filter for keywords "egfr OR kras"
{
	values: ["breast cancer", "egfr|kras"],
	location: {country: "united states", city: "worcester", state: "massachusetts"},
	output: "html",
	start: 0,
	length: 100,
	api_key: "abc0123456789xyz"
}
```	

* **Success Response:**
The successful response includes either the JSON or formatted HTML string. The `recordsTotal` is the number of trials found _before_ any subsequent filters are applied. `recordsFiltered` is the number of remaining trials _after_ filtering. `sitesAvailable` are the number of trial-site pairs since a given trial can have zero or more sites connected to it.

The geographic sort is applied _after_ all filters are applied. The response includes trials that are found within 200 miles (350kM) of the reference location specified.

The `sitesAvailable` figure is the limit for paging. 
```
// => response
{
	html: "<html ...",
	queryId: "unique identifier",
	recordsTotal: 1627,
	recordsFiltered: 301,
	sitesAvailable: 287
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

  <_Just a sample call to your endpoint in a runnable format ($.ajax call or a curl request) - this makes life easier and more predictable._> 

* **Notes:**

  <_This is where all uncertainties, commentary, discussion etc. can go. I recommend timestamping and identifying oneself when leaving comments here._> 
  
  
  # TrialIO API

## Patient Report

### POST /api/v1/publication

	// example body
	{
		values: [{name:"other_terms", value:"breast cancer"}, {name:"other_terms", value: "kras|egfr"}],
		location: {country: "united states", city: "worcester", state: "massachusetts"},
		sort: "distance",
		output: "html",
		start: 0,
		length: 100,
		api_key: "some_key"
	}
	
