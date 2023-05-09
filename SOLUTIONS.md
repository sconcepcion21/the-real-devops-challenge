Challenge 1. The API returns a list instead of an object
As you can see, the API returns a list in the two exposed endpoints:

/api/v1/restaurant: Returns a list containing all the restaurants.
/api/v1/restaurant/{id}: Returns a list with a single restaurant that match the id path parameter.

We want to fix the second endpoint. Return a json object instead of a json array if there is a match or a http 204 status code if no match found.