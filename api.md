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
	
	`output=[string allowed values "html" or "json"]`

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

```
// => response
{
	html: "<html ...",
	queryId: "unique identifier"
}
```

  * **Code:** 201 <br />
    **Content:**
    	`{ html: "<html ...", queryId: "unique identifier"}`
 
* **Error Response:**

  <_Most endpoints will have many ways they can fail. From unauthorized access, to wrongful parameters etc. All of those should be liste d here. It might seem repetitive, but it helps prevent assumptions from being made where they should be._>

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
	
