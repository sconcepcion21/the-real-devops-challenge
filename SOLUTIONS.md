Challenge 1. The API returns a list instead of an object
As you can see, the API returns a list in the two exposed endpoints:

/api/v1/restaurant: Returns a list containing all the restaurants.
/api/v1/restaurant/{id}: Returns a list with a single restaurant that match the id path parameter.

We want to fix the second endpoint. Return a json object instead of a json array if there is a match or a http 204 status code if no match found.

Within the function, it performs a database query to find the restaurant corresponding to the provided ID. mongo.db.restaurant.find_one() method is used to get a single document instead of a list of documents. 

replace restaurants = find_restaurants(mongo, id) with  restaurant = mongo.db.restaurant.find_one({"_id": ObjectId(id)}).

Then, it checks if a restaurant was found. If restaurant is None, it means that no match was found. In this case, a response with HTTP status code 204 is returned using the abort(204) function.

If a restaurant was found, it is returned as a JSON object instead of a list. To do this we use Flask's jsonify() function to serialise the JSON object and replace return jsonify(restaurants) with return jsonify(restaurant).

