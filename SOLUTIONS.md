Challenge 1. The API returns a list instead of an object

As you can see, the API returns a list in the two exposed endpoints:

/api/v1/restaurant: Returns a list containing all the restaurants.
/api/v1/restaurant/{id}: Returns a list with a single restaurant that match the id path parameter.

-----

We want to fix the second endpoint. Return a json object instead of a json array if there is a match or a http 204 status code if no match found.

Within the function, it performs a database query to find the restaurant corresponding to the provided ID. mongo.db.restaurant.find_one() method is used to get a single document instead of a list of documents. 

replace restaurants = find_restaurants(mongo, id) with  restaurant = mongo.db.restaurant.find_one({"_id": ObjectId(id)}).

Then, it checks if a restaurant was found. If restaurant is None, it means that no match was found. In this case, a response with HTTP status code 204 is returned using the abort(204) function.

If a restaurant was found, it is returned as a JSON object instead of a list. To do this we use Flask's jsonify() function to serialise the JSON object and replace return jsonify(restaurants) with return jsonify(restaurant).

Challenge 2. Test the application in any cicd system

As a good devops engineer, you know the advantages of running tasks in an automated way. There are some cicd systems that can be used to make it happen. Choose one, travis-ci, gitlab-ci, circleci... whatever you want. Give us a successful pipeline.

-----

A workflow is configured for a Python application. The main stages of the workflow are as follows:

Environment setup: The execution environment is set, in this case, "ubuntu-latest," which uses the latest version of Ubuntu. The Python version to be used is also specified, in this case, "3.10."

Clone the repository: The actions/checkout@v3 step is used to clone the repository into the execution environment.

Install dependencies: The actions/setup-python@v3 step is used to set up the specified Python version. Then, the necessary dependencies are installed by running the command pip install -r requirements.txt to install the dependencies defined in the requirements.txt file.

Linting with flake8: The command flake8 . --exclude=.git,pycache,.venv --filename=*.py is executed to perform linting on the Python source code. Flake8 is a linting tool that checks for code style compliance and detects potential errors and issues in the code.

The command tox -e py is executed to run the tests defined in the virtual environment configured by tox.