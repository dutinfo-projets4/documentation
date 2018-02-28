# Errors
The Alohomora API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your token is wrong or expired.
403 | Forbidden -- Either you are not supposed to use this endpoint (e.g. admin endpoints) or you are targetting content that you don't own.
404 | Not Found -- The ID you are using don't exists
410 | Gone -- When you use a delete request, this will be returned if it was successful
429 | Too Many Requests -- I've told you to not refresh on a timed basis !
