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
	
	`location={ object containing _country_, _city_, _state_ properties}`
	`start=[integer first record]`
	`length=[integer length of page requested]`
	`output=[string allowed values "html" or "json"]`
	
	// example body
	{
		values: ["breast cancer", "egfr|kras"],
		location: {country: "united states", city: "worcester", state: "massachusetts"},
		output: "html",
		start: 0,
		length: 100,
		api_key: "abc0123456789xyz"
	}
	

* **Success Response:**
  
  <_What should the status code be on success and is there any returned data? This is useful when people need to to know what their callbacks should expect!_>

  * **Code:** 200 <br />
    **Content:** `{ id : 12 }`
 
* **Error Response:**

  <_Most endpoints will have many ways they can fail. From unauthorized access, to wrongful parameters etc. All of those should be liste d here. It might seem repetitive, but it helps prevent assumptions from being made where they should be._>

  * **Code:** 401 UNAUTHORIZED <br />
    **Content:** `{ error : "Log in" }`

  OR

  * **Code:** 422 UNPROCESSABLE ENTRY <br />
    **Content:** `{ error : "Email Invalid" }`

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
	
	// => response
	{
		html: "<html ...",
		queryId: "unique identifier"
	}
