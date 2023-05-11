Challenge 1. The API returns a list instead of an object

As you can see, the API returns a list in the two exposed endpoints:

/api/v1/restaurant: Returns a list containing all the restaurants.
/api/v1/restaurant/{id}: Returns a list with a single restaurant that match the id path parameter.

------

We want to fix the second endpoint. Return a json object instead of a json array if there is a match or a http 204 status code if no match found.

Within the function, it performs a database query to find the restaurant corresponding to the provided ID. mongo.db.restaurant.find_one() method is used to get a single document instead of a list of documents. 

replace restaurants = find_restaurants(mongo, id) with  restaurant = mongo.db.restaurant.find_one({"_id": ObjectId(id)}).

Then, it checks if a restaurant was found. If restaurant is None, it means that no match was found. In this case, a response with HTTP status code 204 is returned using the abort(204) function.

If a restaurant was found, it is returned as a JSON object instead of a list. To do this we use Flask's jsonify() function to serialise the JSON object and replace return jsonify(restaurants) with return jsonify(restaurant).

####

Challenge 2. Test the application in any cicd system

As a good devops engineer, you know the advantages of running tasks in an automated way. There are some cicd systems that can be used to make it happen. Choose one, travis-ci, gitlab-ci, circleci... whatever you want. Give us a successful pipeline.

------

A workflow is configured for a Python application. The main stages of the workflow are as follows:

Environment setup: The execution environment is set, in this case, "ubuntu-latest," which uses the latest version of Ubuntu. The Python version to be used is also specified, in this case, "3.10."

Clone the repository: The actions/checkout@v3 step is used to clone the repository into the execution environment.

Install dependencies: The actions/setup-python@v3 step is used to set up the specified Python version. Then, the necessary dependencies are installed by running the command pip install -r requirements.txt to install the dependencies defined in the requirements.txt file.

Linting with flake8: The command flake8 . --exclude=.git,pycache,.venv --filename=*.py is executed to perform linting on the Python source code. Flake8 is a linting tool that checks for code style compliance and detects potential errors and issues in the code.

The command tox -e py is executed to run the tests defined in the virtual environment configured by tox.

####

Challenge 3. Dockerize the APP

What about containers? As this moment (2018), containers are a standard in order to deploy applications (cloud or in on-premise systems). So the challenge is to build the smaller image you can. Write a good Dockerfile :)

------

FROM python:3.10-slim: This line sets the base image of the container as a lightweight Python 3.10 image. The "slim" version refers to a smaller and optimized image compared to the full version.

WORKDIR /app: This line sets the working directory inside the container to "/app". All subsequent commands will be executed from this directory.

COPY app.py /app/app.py: This line copies the "app.py" file from the local directory to the "/app" directory inside the container.

COPY src /app/src: This line copies the entire "src" directory from the local directory to the "/app/src" directory inside the container. This ensures that all files and subdirectories within "src" are available in the container.

COPY requirements.txt /app/requirements.txt: This line copies the "requirements.txt" file from the local directory to the "/app" directory inside the container. The "requirements.txt" file usually contains the necessary Python dependencies to run the application.

RUN pip install --no-cache-dir -r requirements.txt: This line installs the Python dependencies using the "pip install" command. "--no-cache-dir" prevents downloaded packages from being cached during installation, helping to reduce the final image size.

EXPOSE 8080: This line exposes port 8080 within the container. It allows external communication with the application running on that port.

CMD [ "python", "app.py" ]: This line sets the command to be executed when the container starts. In this case, it runs the Flask application defined in "app.py" using the Python interpreter.

####

Challenge 4. Dockerize the database

We need to have a mongodb database to make this application run. So, we need a mongodb container with some data. Please, use the restaurant dataset to load the mongodb collection before running the application.

The loaded mongodb collection must be named: restaurant. Do you have to write code or just write a Docker file?

------

FROM mongo:6.0.5: This line specifies that we are using the official MongoDB image with the tag "6.0.5" as the base for our container. This means we are building our container on top of the MongoDB version 6.0.5 image.

COPY data/restaurant.json /restaurant.json: This line copies the restaurant.json file from the data directory of our local system to the root directory (/) of the container. This means we are adding the restaurant.json file to the container so that it is available during the build and execution.

COPY data/import.sh /docker-entrypoint-initdb.d/import.sh: This line copies the import.sh script from the data directory of our local system to the /docker-entrypoint-initdb.d/ directory of the container. This directory is special in MongoDB, as any script placed in this directory will automatically run when the database starts. In this case, we are placing the import.sh script in that directory to execute when the MongoDB container is started.

####

Challenge 5. Docker Compose it

Once you've got dockerized all the API components (python app and database), you are ready to make a docker-compose file. KISS.

------

version: '3': Specifies the version of the Docker Compose syntax being used.

services: Defines the services that make up the application.

mongodb: Defines the MongoDB service.

build: Specifies how to build the MongoDB container image.

context: .: Indicates that the build context is the current directory where the docker-compose.yml file is located.

dockerfile: Dockerfile-mongodb: Specifies the name of the Dockerfile to use for building the MongoDB container image.

ports: Maps the host ports to the container. In this case, port 27017 of the host is mapped to port 27017 of the container, allowing access to the MongoDB instance from the host.

volumes: Defines the volumes to be used in the MongoDB container. In this case, a volume named "dbdata" is created and mounted to the "/data/db" directory in the container. This allows for persistent storage of MongoDB data even if the container is stopped or removed.

app: Defines the application service.

build: Specifies how to build the application container image.

context: .: Indicates that the build context is the current directory.

dockerfile: Dockerfile-app: Specifies the name of the Dockerfile to use for building the application container image.

ports: Maps the host ports to the container. In this case, port 8080 of the host is mapped to port 8080 of the container, allowing access to the application from the host.

depends_on: Indicates that this service depends on the MongoDB service. Docker Compose ensures that the MongoDB service is started before this service.

environment: Defines the environment variables to be used in the application container. In this case, the "MONGO_URI" environment variable is set to "mongodb://mongodb:27017/test", which is the connection URL to the MongoDB instance.

volumes: Defines the volumes to be used in the services.

dbdata: Defines the "dbdata" volume to be used for persistent storage of MongoDB data.

####

Final Challenge. Deploy it on kubernetes

If you are a container hero, an excellent devops... We want to see your expertise. Use a kubernetes system to deploy the API. We recommend you to use tools like minikube or microk8s.

Write the deployment file (yaml file) used to deploy your API (python app and mongodb).

------

apiVersion and kind specify the Kubernetes API version and the object type (Deployment in this case).
metadata contains metadata like the deployment name.
spec describes the characteristics of the deployment.
replicas: 1 indicates that one replica of the pod should be created.
selector defines how pods will be selected based on labels. In this case, pods are selected based on the label app: my-app.
template defines the template for pods created by this deployment.
metadata under template contains labels for the created pods.
spec under template describes the characteristics of the created pods.
containers specifies the containers to run in the pods.
name defines the container name.
image specifies the container image.
ports defines the ports to expose within the container.
volumeMounts specifies the mount points for the container.
volumes defines the volumes available for the pods.
In summary, this YAML file defines a Deployment that creates a pod with two containers: one with the MongoDB image and another with the Python image. The MongoDB container exposes port 27017 and uses a volume named "dbdata" to store the database data. The Python container exposes port 8080 and uses the environment variable "MONGO_URI" to establish a connection to the MongoDB database.

------

Conclusion

I was unable to complete this final challenge as I haven't worked with Kubernetes before and don't have enough experience to solve it. I was only able to bring up one of the desired containers. However, I enjoyed the challenge and it provided me with an opportunity to learn and further explore tools like Docker, which I had some prior experience with but not at this level.

Thank you very much.