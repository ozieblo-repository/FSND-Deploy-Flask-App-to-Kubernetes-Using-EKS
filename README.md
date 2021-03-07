# Deploying a Flask API

This is the project repo for the fourth course in the [Udacity Full Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004): Server Deployment, Containerization, and Testing.

In this project the aim was to containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'. 
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token. 

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.

## Dependencies

- Docker Engine
    - Installation instructions for all OSes can be found [here](https://docs.docker.com/install/).
    - For Mac users, if you have no previous Docker Toolbox installation, you can install Docker Desktop for Mac. If you already have a Docker Toolbox installation, please read [this](https://docs.docker.com/docker-for-mac/docker-toolbox/) before installing.
 - AWS Account
     - You can create an AWS account by signing up [here](https://aws.amazon.com/#).
     
## Project Steps

Completing the project involved several steps:

1. Write a Dockerfile for a simple Flask API
2. Build and test the container locally
3. Create an EKS cluster
4. Store a secret using AWS Parameter Store
5. Create a CodePipeline pipeline triggered by GitHub checkins
6. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson [here](https://classroom.udacity.com/nanodegrees/nd004/parts/1d842ebf-5b10-4749-9e5e-ef28fe98f173/modules/ac13842f-c841-4c1a-b284-b47899f4613d/lessons/becb2dac-c108-4143-8f6c-11b30413e28d/concepts/092cdb35-28f7-4145-b6e6-6278b8dd7527).

## Validation

#### Install python dependencies

```pip install -r requirements.txt```

#### Set up the environment

JWT_SECRET - The secret used to make the JWT, for the purpose of this course the secret can be any string.

LOG_LEVEL - It represents the level of logging. It is optional to be set. It has a default value as 'INFO', but when debugging an app locally, you may want to set it to 'DEBUG'. To add these to your terminal environment, run the following

```
export JWT_SECRET='myjwtsecret'
export LOG_LEVEL=DEBUG
```

#### Run the app using the Flask server

from the top directory, run:

```python main.py```

Open http://127.0.0.1:8080/ in a new browser OR run ```curl --request GET http://localhost:8080/``` on the command-line terminal. It will give you a response as "Healthy".

#### Try the API endpoints on Command-Line

Open another terminal window, and install jq, which is a package that helps to read or manipulate JSON processors. 

For Linux

```sudo apt-get install jq```

For Mac

```brew install jq```

To try the /auth endpoint, use the following command, replacing email/password as applicable to you

```
export TOKEN=`curl --data '{"email":"abc@xyz.com","password":"mypwd"}' --header "Content-Type: application/json" -X POST localhost:8080/auth  | jq -r '.token'` 
```

This calls the endpoint 'localhost:8080/auth' with the email/password as the message body. The return value is a JWT token based on the secret string you supplied. We are assigning that secret to the environment variable 'TOKEN'. To see the JWT token, run:

```echo $TOKEN```

To try the /contents endpoint which decrypts the token and returns its content, run

```curl --request GET 'http://localhost:8080/contents' -H "Authorization: Bearer ${TOKEN}" | jq .```

To use the endpoints checking the working Docker container, you can use the same curl commands as before, except using port 80 this time

```curl --request GET 'http://localhost:80/'```

#### To get the external IP for your service (as the owner):

```kubectl get services simple-jwt-api -o wide```

For this project: [ac08005c1a24141d08bc1335b725aa47-1054649433](ac08005c1a24141d08bc1335b725aa47-1054649433.us-west-2.elb.amazonaws.com)

#### Then, use the external IP url to test the app (any user, the page returns the response 'Healthy'):
```
export URL="<EXTERNAL-IP>.us-east-2.elb.amazonaws.com"
export TOKEN=`curl -d '{"email":"<EMAIL>","password":"<PASSWORD>"}' -H "Content-Type: application/json" -X POST <EXTERNAL-IP URL>/auth  | jq -r '.token'`curl --request GET $URL:80/contents -H "Authorization: Bearer ${TOKEN}" | jq
curl --request GET $URL:80/contents -H "Authorization: Bearer ${TOKEN}" | jq
```
