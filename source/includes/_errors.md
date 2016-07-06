# Errors

<aside class="notice">The SFOX API communicates error codes through HTTP status codes.  The following are the error codes currently in use.</aside>

The SFOX API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request was malformed in some way
401 | Unauthorized -- Your API key is wrong
403 | Forbidden -- The API key is not authorized for this endpoint
404 | Not Found -- The specified endpoint could not be found
405 | Method Not Allowed -- You tried to access a endpoint with an invalid method
406 | Not Acceptable -- You requested a format that isn't json
429 | Too Many Requests -- You have exceeded your request limit
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarially offline for maintanance. Please try again later.
