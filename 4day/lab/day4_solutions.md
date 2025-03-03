# Day 4 Lab: Building and Deploying Machine Learning APIs with FastAPI

## Table of Contents

- [Day 4 Lab: Building and Deploying Machine Learning APIs with FastAPI](#day-4-lab-building-and-deploying-machine-learning-apis-with-fastapi)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Theoretical Concepts](#theoretical-concepts)
    - [What is FastAPI?](#what-is-fastapi)
    - [Asynchronous Programming in Python](#asynchronous-programming-in-python)
    - [Why Use FastAPI for ML APIs?](#why-use-fastapi-for-ml-apis)
    - [Containerizing and Deploying FastAPI](#containerizing-and-deploying-fastapi)
  - [Lab Instructions](#lab-instructions)
    - [Part 0: Understanding Async in Python](#part-0-understanding-async-in-python)
      - [Task 0.1: Basic synchronous example](#task-01-basic-synchronous-example)
      - [Task 0.2: Basic async Python example](#task-02-basic-async-python-example)
      - [Task 0.2: Understanding Concurrency with Async Tasks](#task-02-understanding-concurrency-with-async-tasks)
      - [Key Learning Points](#key-learning-points)
    - [Part 1: Setting up FastAPI](#part-1-setting-up-fastapi)
      - [Task 1.1: Installing FastAPI and Uvicorn](#task-11-installing-fastapi-and-uvicorn)
      - [Task 1.2: Setting Up a Basic FastAPI App](#task-12-setting-up-a-basic-fastapi-app)
      - [Task 1.3: Running the FastAPI App on localhost](#task-13-running-the-fastapi-app-on-localhost)
    - [Part 2: Model Training and Setup](#part-2-model-training-and-setup)
      - [Task 2.1: Train and Save Logistic Regression and RandomForest Models](#task-21-train-and-save-logistic-regression-and-randomforest-models)
      - [Task 2.2: Attach Models in FastAPI using Lifespan](#task-22-attach-models-in-fastapi-using-lifespan)
      - [Task 2.3: Create a GET endpoint to list available models](#task-23-create-a-get-endpoint-to-list-available-models)
      - [Task 2.4: Use a `.env` file for the location of models](#task-24-use-a-env-file-for-the-location-of-models)
    - [`[BONUS]` Part 3: Building a Simple Prediction API](#bonus-part-3-building-a-simple-prediction-api)
      - [Task 3.1: Setup FastAPI with Prediction Endpoint](#task-31-setup-fastapi-with-prediction-endpoint)
      - [Task 3.2: Adding Asynchronous Predictions](#task-32-adding-asynchronous-predictions)
      - [Task 3.3: Enhanced Schema Validation](#task-33-enhanced-schema-validation)
    - [Part 4: Dockerizing and Deploying FastAPI](#part-4-dockerizing-and-deploying-fastapi)
      - [Task 4.1: Dockerizing the Application](#task-41-dockerizing-the-application)
      - [Task 4.2: Deploy to Cloud (GCP)](#task-42-deploy-to-cloud-gcp)
  - [Conclusion](#conclusion)
  - [Useful Links](#useful-links)

## Overview

In this lab, we will build a machine learning API using FastAPI. We'll start by understanding asynchronous programming in Python, followed by building a simple API to serve multiple ML models using FastAPI. We’ll also add schema validation, error handling, and containerize the FastAPI application using Docker, followed by deployment to a cloud platform (GCP or Azure).

## Theoretical Concepts

### What is FastAPI?

**FastAPI** is a modern, fast web framework for building APIs with Python. It is designed to be easy to use and powerful, offering automatic request validation, async support, and more.

- **Main Features**:
  - High performance, comparable to NodeJS and Go.
  - Automatic interactive API documentation (Swagger UI and ReDoc).
  - Built-in support for asynchronous code.
  - Automatic validation of request data using **Pydantic**.

### Asynchronous Programming in Python

**Asynchronous programming** allows Python programs to handle multiple requests concurrently. Using `async` and `await`, you can write non-blocking code that performs tasks in parallel without waiting for one task to finish before starting another.

For example, when an ML model takes time to predict, async allows the API to handle other requests while waiting for the prediction to finish.

### Why Use FastAPI for ML APIs?

FastAPI is ideal for machine learning APIs because of:

- **Speed**: FastAPI is designed for high-performance applications.
- **Async Support**: Handle multiple requests simultaneously, which is crucial when serving large models or performing time-consuming inference tasks.
- **Built-in Validation**: FastAPI uses **Pydantic** for request validation, ensuring correct data input.
- **Ease of Deployment**: FastAPI can be easily deployed using Docker and cloud platforms like **GCP** and **Azure**.

### Containerizing and Deploying FastAPI

Using **Docker**, you can create lightweight containers that package your API and its dependencies, making it portable across different environments. Once containerized, you can deploy your FastAPI app to cloud services like **Google Cloud Run** or **Azure App Services**.

## Lab Instructions

### Part 0: Understanding Async in Python

Before diving into FastAPI, let's explore the concepts of synchronous and asynchronous programming in Python. We'll show how asynchronous tasks can help reduce wait times when handling multiple tasks concurrently.

**Goal**: By the end of this section, you will understand how Python’s async and await keywords allow you to run tasks concurrently, improving efficiency in scenarios where tasks involve waiting (e.g., I/O-bound tasks).

#### Task 0.1: Basic synchronous example

Run the following code and note how Task 2 waits for Task 1 to finish before starting:

  ```python
  import time

  def slow_task(name, delay):
      print(f"Starting task {name}: {time.strftime('%X')}")
      time.sleep(delay)
      print(f"Finished task {name}: {time.strftime('%X')}")

  def main():
      slow_task("Task 1", 3)
      slow_task("Task 2", 7)
      print("All tasks finished")

  main()
  ```

#### Task 0.2: Basic async Python example

Now, let's convert the previous example into an asynchronous one. Even though this is asynchronous, we're still awaiting each task one after another.

**Explanation**:

- The `async def` keyword marks the function as asynchronous.
- The `await` keyword allows the function to "pause" without blocking other tasks.

  ```python
  import asyncio
  import time

  async def slow_task(name, delay):
      print(f"Starting task {name}: {time.strftime('%X')}")
      await asyncio.sleep(delay)
      print(f"Finished task {name}: {time.strftime('%X')}")

  async def main():
      await slow_task("Task 1", 3)
      await slow_task("Task 2", 7)
      print("All tasks finished")

  asyncio.run(main())
  ```

#### Task 0.2: Understanding Concurrency with Async Tasks

Let's now modify the code to run both tasks concurrently.

**Explanation**:

- `asyncio.create_task()` schedules the tasks to run concurrently.
- The program doesn’t wait for `task1` to complete before starting `task2`. Instead, both tasks are "created" at nearly the same time and will run concurrently.

  ```python
  import asyncio
  import time

  async def slow_task(name, delay):
      print(f"Starting task {name}: {time.strftime('%X')}")
      await asyncio.sleep(delay)
      print(f"Finished task {name}: {time.strftime('%X')}")

  async def main():
      print(f"Starting: {time.strftime('%X')}")

      task1 = asyncio.create_task(slow_task("Task 1", 3))
      print(f"Created task 1: {time.strftime('%X')}")

      task2 = asyncio.create_task(slow_task("Task 2", 7))
      print(f"Created task 2: {time.strftime('%X')}")

      await task1
      print(f"Awaited task 1: {time.strftime('%X')}")

      await task2
      print(f"Awaited task 2: {time.strftime('%X')}")

  asyncio.run(main())
  ```

#### Key Learning Points

- Synchronous programming: Tasks are executed one after another, causing delays when tasks involve waiting.
- Asynchronous programming: Tasks run concurrently, reducing overall execution time, especially when waiting for I/O operations.

### Part 1: Setting up FastAPI

In this part, we will install FastAPI and Uvicorn, set up a simple "health check" endpoint, and run it on your local machine.

#### Task 1.1: Installing FastAPI and Uvicorn

To start building our FastAPI app, we need to install FastAPI and a web server, Uvicorn, that will serve the application.

1. **Install FastAPI and Uvicorn**

   ```bash
   pip install fastapi[standard]
   pip instal uvicorn
   ```

#### Task 1.2: Setting Up a Basic FastAPI App

Next, we'll create a basic FastAPI app with a health check endpoint.

1. Create a new Python file called `main.py`:

   ```bash
   touch main.py
   ```

2. Add the following code to main.py:

   ```python
   from fastapi import FastAPI

   # Create a FastAPI instance
   app = FastAPI()

   # Health check endpoint
   @app.get("/health")
   async def health_check():
       return {"status": "healthy"}
   
   @app.get("/")
   async def root():
       return {"message": "Hello World"}
   ```

This code creates a FastAPI instance and defines a single GET endpoint `/health` that returns a simple JSON response indicating the service is healthy. Moreover, the root endpoint `/` return a simple `Hello World` message.

#### Task 1.3: Running the FastAPI App on localhost

Now that we have our FastAPI app, let's run it using Uvicorn.

1. Run the FastAPI app:

   ```bash
   uvicorn main:app --reload
   ```

   or

   ```bash
   fastapi dev main.py
   ```

   - **Explanations**:
     - `main` refers to the name of the Python file (main.py).
     - `app` refers to the FastAPI instance created inside the file.
     - The `--reload` flag enables auto-reloading of the app when code changes are detected.

2. Test the health check endpoint:

   - visit `http://127.0.0.1:8000/health`

3. Explore the Interactive API docs:
   - visit `http://127.0.0.1:8000/docs`

### Part 2: Model Training and Setup

In this part, we'll train the **Logistic Regression** and **RandomForest** models using the Iris dataset. After training, we'll save these models and attach them in FastAPI using the **lifespan** feature. Additionally, we will create a GET endpoint to list the available models.

#### Task 2.1: Train and Save Logistic Regression and RandomForest Models

1. **Train the models and save them to disk**:

   ```python
   import pickle
   from sklearn.datasets import load_iris
   from sklearn.linear_model import LogisticRegression
   from sklearn.ensemble import RandomForestClassifier

   # Load the Iris dataset
   iris = load_iris()
   X, y = iris.data, iris.target

   # Train Logistic Regression model
   logreg_model = LogisticRegression(max_iter=200)
   logreg_model.fit(X, y)

   # Train Random Forest model
   rf_model = RandomForestClassifier()
   rf_model.fit(X, y)

   # Save models to disk
   with open("logistic_regression.pkl", "wb") as f:
       pickle.dump(logreg_model, f)

   with open("random_forest.pkl", "wb") as f:
       pickle.dump(rf_model, f)
   ```

2. **Run the script** to train and save the models:

   ```bash
   python train_models.py
   ```

#### Task 2.2: Attach Models in FastAPI using Lifespan

We'll use FastAPI's lifespan feature to load the models when the application starts, and they'll be shared across routes for making predictions.

1. **Create a FastAPI app with lifespan**:

   ```python
   from fastapi import FastAPI
   from contextlib import asynccontextmanager
   import pickle

   ml_models = {} # Global dictionary to hold the models.
   def load_model(path: str):
       model = None
       with open(path, "rb") as f:
           model = pickle.load(f)
       return model

   @asynccontextmanager
   async def lifespan(app: FastAPI):
       # Load models when the app starts
       ml_models['logistic_model'] = load_model("./models/logistic_regression.pkl)
       ml_models['rf_model'] = load_model("./models/random_fores.pkl)

       yield
       # This code will be executed after the application finishes handling requests, right before the shutdown
       # Clean up the ML models and release the resources
       ml_models.clear()

   app = FastAPI(lifespan=lifespan)

   @app.get("/")
   async def root():
       return {"message": "Models loaded and FastAPI is ready!"}
   ```

2. **Run the FastAPI app**:

   ```bash
   uvicorn main:app --reload
   ```

#### Task 2.3: Create a GET endpoint to list available models

We will now create an additional endpoint that returns a list of all the available models (i.e., the ones loaded in the lifespan).

1. Add a new route to list the models:

   ```python
   @app.get("/models")
   async def list_models():
       # Return the list of available models' names
       return {"available_models": list(ml_models.keys())}
   ```

2. Run the FastAPI app and test the `/models` endpoint:

   - Run following in command line:

   ```bash
   uvicorn main:app --reload
   ```

   - Visit `http://127.0.0.1:8000/models` or `http://127.0.0.1:8000/docs`.

#### Task 2.4: Use a `.env` file for the location of models

We will now create a `.env` file where we will store the location (path) to our models.

1. Create a `.env` file and add two lines in it:

   ```
   LOGISTIC_MODEL={PATH_TO_YOUR_LOGISITC_MODEL}
   RF_MODEL={PATH_TO_YOUR_RF_MODEL}
   ```

2. Load the `.env` varibles and access them in code.

   - Install the required package.

   ``` bash
   pip install python-dotenv
   ```

   - Load the `.env` variables in code.

   ``` python
   from dotenv import load_dotenv
   load_dotenv()
   ```

   - Access the loaded environemnt variables where needed.

   ``` python
    # Load the ML model
    ml_models["logistic_model"] = load_model(os.getenv("LOGISTIC_MODEL"))
    ml_models["rf_model"] = load_model(os.getenv("RF_MODEL"))
   ```

### `[BONUS]` Part 3: Building a Simple Prediction API

Now that the models are loaded, let's serve them through an API.

#### Task 3.1: Setup FastAPI with Prediction Endpoint

We will first create a basic prediction API that accepts input data and uses the pre-loaded models to make predictions.

1. Create a `BaseModel` for the input data:

   ```python
   from pydantic import BaseModel

   class IrisData(BaseModel):
       sepal_length: float
       sepal_width: float
       petal_length: float
       petal_width: float
   ```

   This `BaseModel` will enforce type validation and ensure the correct input schema for our API.

2. Add a POST endpoint for model predictions:

    ```python
    from fastapi import HTTPException

    @app.post("/predict/{model_name}")
    async def predict(model_name: ModelName, iris: IrisData):
        input_data = [[iris.sepal_length, iris.sepal_width, iris.petal_length, iris.petal_width]]
        
        if model_name not in ml_models.keys():
            raise HTTPException(status_code=404, detail="Model not found")
        
        ml_model = ml_models[model_name]
        prediction = ml_model.predict(input_data)

        return {"model": model_name, "prediction": int(prediction[0])}
    ```

3. Test the API using `curl`:
   - Make a POST request with sample data to the`/predict` endpoint.

    ```bash
    curl -X 'POST' \
      'http://localhost:8000/predict/rf_model' \
      -H 'accept: application/json' \
      -H 'Content-Type: application/json' \
      -d '{
      "sepal_length": 0.1,
      "sepal_width": 0.1,
      "petal_length": 0.1,
      "petal_width": 0.1
    }'
    ```

#### Task 3.2: Adding Asynchronous Predictions

To demonstrate FastAPI's asynchronous capabilities, we will simulate long-running tasks by adding artificial delays using `asyncio.sleep()`.

1. Simulate the long-running tasks:

   ```python

   import asyncio

   @app.post("/predict/{model_name}")
   async def predict(model_name: ModelName, iris: IrisData):
       await asyncio.sleep(5) # Mimic heavy workload.
        
       input_data = [[iris.sepal_length, iris.sepal_width, iris.petal_length, iris.petal_width]]
        
       if model_name not in ml_models.keys():
           raise HTTPException(status_code=404, detail="Model not found")
        
       ml_model = ml_models[model_name]
       prediction = ml_model.predict(input_data)

       return {"model": model_name, "prediction": int(prediction[0])}
   ```

2. Test asynchronous behavior with multiple `curl` requests:
   - Make a POST request with sample data to the`/predict/logistic_regression` and `/predict/random_forest` endpoints.

    ```bash
      curl -X 'POST' \
        'http://localhost:8000/predict/rf_model' \
        -H 'accept: application/json' \
        -H 'Content-Type: application/json' \
        -d '{
        "sepal_length": 0.1,
        "sepal_width": 0.1,
        "petal_length": 0.1,
        "petal_width": 0.1
      }' &
      curl -X 'POST' \
        'http://localhost:8000/predict/rf_model' \
        -H 'accept: application/json' \
        -H 'Content-Type: application/json' \
        -d '{
        "sepal_length": 0.1,
        "sepal_width": 0.1,
        "petal_length": 0.1,
        "petal_width": 0.1
      }'
    ```

#### Task 3.3: Enhanced Schema Validation

We'll modify the IrisData schema to include constraints such as minimum and maximum values and add descriptions for each field. We'll also include default values for convenience.
Add validation to ensure all input features are positive.

```python

from pydantic import BaseModel, Field

class IrisData(BaseModel):
    sepal_length: float = Field(..., gt=0, lt=10, description="Sepal length must be between 0 and 10", example=5.1)
    sepal_width: float = Field(..., gt=0, lt=10, description="Sepal width must be between 0 and 10", example=3.5)
    petal_length: float = Field(..., gt=0, lt=10, description="Petal length must be between 0 and 10", example=1.4)
    petal_width: float = Field(..., gt=0, lt=10, description="Petal width must be between 0 and 10", example=0.2)

```

### Part 4: Dockerizing and Deploying FastAPI

In this part, we will Dockerize the FastAPI application and deploy it to a cloud platform. Make sure you have installed and configured docker on your machine: [tutorial](https://docs.docker.com/get-started/get-docker/).

#### Task 4.1: Dockerizing the Application

0. Make sure to have a **requirements.txt** file:

   ```
   fastapi~=0.114.0
   fastapi-cli~=0.0.5
   uvicorn~=0.30.6
   scikit-learn~=1.5.1
   ```

1. **Create a `Dockerfile`** to containerize the FastAPI app:

   ```Dockerfile
   # Use a python image as the base
   FROM python:3.9

   # Set the working directory
   WORKDIR /code

   # Copy the requirements to the working directory
   COPY ./requirements.txt /code/requirements.txt

   # Install dependencies
   RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

   # Copy the app directory contents to the working directory
   COPY ./app /code/app

   # Copy the models into working directory
   COPY ./models /code/models

   # Run the FastAPI application using Uvicorn
   CMD ["fastapi", "run", "main.py", "--port", "80"]
   ```

2. **Build and run the Docker image**:

   ```bash

   # Build the Docker image
   docker build -t fastapi-ml-api .

   # Run the container and supply the .env file
   docker run -d -p 8080:8080 --env-file ./.env fastapi-ml-api
   ```

3. **Test the Dockerized API** by visiting `http://localhost` on your browser or sending requests via Postman.

#### Task 4.2: Deploy to Cloud (GCP)

1. **Google Cloud Platform (GCP) Deployment**:
   - **Make sure the Image is built for the needed platform**:

     ```bash
     docker build --platform=linux/amd64 -t fastapi-ml-api .
     ```

   - **Push Docker Image to Google Container Registry**:

     ```bash
     gcloud auth configure-docker
     docker tag fastapi-ml-api gcr.io/[YOUR_PROJECT_ID]/fastapi-ml-api
     docker push gcr.io/[YOUR_PROJECT_ID]/fastapi-ml-api
     ```

   - **Deploy to Google Cloud Run**:
     - Follow the UI
     - Attach a volume (Google Storage Bucket)
       - Note: If your google cloud bucket has following structure: `lab4models/models/...`, and you mount the `lab4models` into `/mnt/lab4/` this means that the content of `lab4models` will be placed/mounted inside `/mnt/lab4/`. In other words, in order to access `lab4models/models/...` you would write `/mnt/lab4/models/...`.
     - Add environment variables

   - Access the deployed application via the URL provided by GCP.

## Conclusion

In this lab, we covered:

- Training and saving machine learning models.
- Setting up FastAPI to serve multiple models.
- Demonstrating async behavior in FastAPI for handling multiple requests.
- Validating input data using Pydantic's schema validation.
- Adding error handling for invalid requests.
- Dockerizing the FastAPI application.
- Deploying the FastAPI app to cloud platforms like GCP or Azure.

This workflow mirrors real-world production-ready ML systems, providing a robust foundation for scalable and maintainable APIs.

## Useful Links

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Docker Documentation](https://docs.docker.com/)
- [Uvicorn Documentation](https://www.uvicorn.org/)
- [Google Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Azure Container Registry Documentation](https://docs.microsoft.com/en-us/azure/container-registry/)
