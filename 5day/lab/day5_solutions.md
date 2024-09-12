# Day 5 Lab: CI/CD for FastAPI Applications

## Table of Contents

- [Day 5 Lab: CI/CD for FastAPI Applications](#day-5-lab-cicd-for-fastapi-applications)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Theoretical Concepts](#theoretical-concepts)
    - [Continuous Integration and Continuous Deployment (CI/CD)](#continuous-integration-and-continuous-deployment-cicd)
    - [Writing Tests for FastAPI](#writing-tests-for-fastapi)
    - [GitHub Actions](#github-actions)
    - [Google Cloud Run](#google-cloud-run)
  - [Learning Objectives](#learning-objectives)
  - [Lab Instructions](#lab-instructions)
    - [Part 1: Writing Tests for FastAPI](#part-1-writing-tests-for-fastapi)
      - [Task 1.1: Setting up Pytest](#task-11-setting-up-pytest)
      - [Task 1.2: Writing Unit Tests for FastAPI](#task-12-writing-unit-tests-for-fastapi)
    - [Part 2: Setting Up GitHub Actions for CI/CD](#part-2-setting-up-github-actions-for-cicd)
      - [Task 2.1: Creating a GitHub Actions Workflow](#task-21-creating-a-github-actions-workflow)
      - [Task 2.2: Running Automated Tests](#task-22-running-automated-tests)
    - [`BONUS` Part 3: Using GitHub Secrets and Variables](#bonus-part-3-using-github-secrets-and-variables)
      - [Task 3.1: Demonstrate using GitHub Secrets and Variables in Workflows](#task-31-demonstrate-using-github-secrets-and-variables-in-workflows)
    - [Part 4: Deploying to Google Cloud Run with GitHub integration](#part-4-deploying-to-google-cloud-run-with-github-integration)
      - [Task 4.1: Setting Up Google Cloud Run](#task-41-setting-up-google-cloud-run)
      - [Task 4.2: Attaching Volumes and Environment Variables](#task-42-attaching-volumes-and-environment-variables)
      - [Task 4.3: Test out the automatic deployment](#task-43-test-out-the-automatic-deployment)
  - [Conclusion](#conclusion)
  - [Useful Links](#useful-links)

## Overview

In this lab, we will explore how to implement a continuous integration and continuous deployment (CI/CD) pipeline for your FastAPI application. We'll write tests using **pytest**, set up a GitHub Actions workflow to automate the testing process, and build and push Docker images automatically when code is pushed to the repository.

## Theoretical Concepts

### Continuous Integration and Continuous Deployment (CI/CD)

CI/CD is a software development practice where code changes are automatically tested, built, and deployed to a production or staging environment. It involves two key processes:

- **Continuous Integration (CI)**: Automates the testing and integration of new code into the main branch. Developers frequently commit code, which triggers automated tests to ensure new changes don’t break existing functionality.
- **Continuous Deployment (CD)**: Automatically deploys the application to a production environment after the code passes the CI phase. This ensures that new features or bug fixes are delivered quickly and reliably.

### Writing Tests for FastAPI

FastAPI supports testing using the **TestClient** from **Starlette**, which provides a simple interface for testing endpoints. Automated tests ensure that changes to the codebase do not introduce new bugs. In this lab, you'll use **pytest** and **pytest-mock** to test FastAPI routes.

Key concepts for testing:

- **Unit Tests**: Small, isolated tests that verify the correctness of individual units of code (e.g., functions, routes).
- **Mocking**: Replacing parts of the system under test with mock objects that simulate the behavior of real components. This allows us to test a system independently of external dependencies, such as databases or external services.

### GitHub Actions

GitHub Actions is a CI/CD platform integrated into GitHub repositories. It allows you to define workflows in a YAML file to automate testing, building, and deploying your application. In this lab, you’ll use GitHub Actions to run tests on every code push, ensuring that your FastAPI app works as expected.

### Google Cloud Run

**Google Cloud Run** is a serverless compute platform that automatically scales your applications. It allows you to deploy containerized applications without having to manage the underlying infrastructure. With Cloud Run, you can trigger automatic deployments directly from a GitHub repository whenever new code is pushed, making it ideal for CI/CD pipelines.

Cloud Run features:

- **Serverless**: No need to manage servers or clusters.
- **Scalable**: Automatically scales based on traffic, ensuring your application can handle varying workloads.
- **Integrated with GitHub**: Easily connect your GitHub repository for continuous deployment.

## Learning Objectives

By the end of this lab, students will:

- Write automated tests for FastAPI applications using **pytest**.
- Set up a GitHub Actions pipeline to run tests on each push/commit.
- Build and push Docker images automatically using CI/CD.
- Optionally deploy the Docker image to a cloud platform.

## Lab Instructions

### Part 1: Writing Tests for FastAPI

Before setting up CI/CD, it's essential to write tests for your application. In this section, you'll write unit tests for your FastAPI app.

#### Task 1.1: Setting up Pytest

1. Install **pytest** and **pytest-mock** if not already installed:

    ```bash
    pip install pytest pytest-mock
    ```

2. Create a file named `test_main.py` for your tests:

    ```bash
    touch test_main.py
    ```

#### Task 1.2: Writing Unit Tests for FastAPI

You’ll write tests for your FastAPI application. You can always check out the [documentation](https://fastapi.tiangolo.com/tutorial/testing/#using-testclient) on how it can be done.

If needed, mock/patch the models (predict function) to return default values.

1. Test the endpoints: `root`, `health_check`, `list_models`.
2. Test prediction with an invalid model name.
3. Test prediction with a valid model name.

```python
from unittest.mock import MagicMock

import pytest
from fastapi.testclient import TestClient

from app.main import app


def test_root():
    with TestClient(app) as client:
        response = client.get("/")
        assert response.status_code == 200
        assert response.json() == {"message": "Hello World"}


def test_health_check():
    with TestClient(app) as client:
        response = client.get("/health")
        assert response.status_code == 200
        assert response.json() == {"status": "healthy"}


def test_list_models():
    with TestClient(app) as client:
        response = client.get("/models")
        assert response.status_code == 200
        assert response.json() == {"available_models": ["logistic_model", "rf_model"]}


def test_predict_valid_model():
    with TestClient(app) as client:
        response = client.post(
            "/predict/rf_model",
            json={
                "sepal_length": 5.1,
                "sepal_width": 3.5,
                "petal_length": 1.4,
                "petal_width": 0.2,
            },
        )
        assert response.status_code == 200
        assert response.json() == {"model": "rf_model", "prediction": 0}


def test_predict_invalid_model():
    with TestClient(app) as client:
        response = client.post(
            "/predict/invalid_model",
            json={
                "sepal_length": 5.1,
                "sepal_width": 3.5,
                "petal_length": 1.4,
                "petal_width": 0.2,
            },
        )

        assert response.status_code == 422


@pytest.fixture
def mock_models(mocker):
    mock_dict = {"logistic_model": MagicMock, "rf_model": MagicMock}
    m = mocker.patch(
        "app.main.ml_models",
        return_value=mock_dict,
    )
    m.keys.return_value = mock_dict.keys()
    return m


def test_predict_mocked(mock_models):
    mock_models["logistic_model"].predict.return_value = [-1]

    with TestClient(app) as client:
        response = client.post(
            "/predict/logistic_model",
            json={
                "sepal_length": 5.1,
                "sepal_width": 3.5,
                "petal_length": 1.4,
                "petal_width": 0.2,
            },
        )

        assert response.status_code == 200
        assert response.json() == {"model": "logistic_model", "prediction": -1}
```

### Part 2: Setting Up GitHub Actions for CI/CD

In this part, you'll create a CI/CD pipeline using GitHub Actions that runs your tests every time you push code.

#### Task 2.1: Creating a GitHub Actions Workflow

1. In your project, create a `.github/workflows` directory:

   ```bash
   mkdir -p .github/workflows
   ```

2. Create a file named `test.yml` inside the `.github/workflows` directory:

   ```bash
   touch .github/workflows/ci.yml
   ```

3. Fill in the `yml` file such that:
   - define name of your workflow
   - define rules when to trigger (e.g., on push to main, on pull request to main, etc.)
   - define a `test` job that will include the steps of:
     - setting up python 3.9
     - installing dependencies and upgrading pip
     - running tests

    ```yml
    name: CI/CD Test Pipeline

    on:
    push:
        branches:
            - main
    pull_request:
        branches:
            - main

    jobs:
        test:
            runs-on: ubuntu-latest

            steps:
            - uses: actions/checkout@v2

            - name: Set up Python
              uses: actions/setup-python@v2
              with:
                python-version: '3.8'

            - name: Install dependencies
              run: |
                python -m pip install --upgrade pip
                pip install -r requirements.txt
                pip install pytest pytest-mock

            - name: Run tests
              env:
                RF_MODEL: ${{vars.RF_MODEL}}
              run: |
                pytest
    ```

#### Task 2.2: Running Automated Tests

1. Push your code to GitHub:

   ```bash
   git add .
   git commit -m "Added tests and CI/CD workflow"
   git push origin main
   ```

2. Go to the `Actions` tab of your GitHub repository and you should see the workflow running.

3. Fail the pipeline:
    - Update your code in such a way to make the tests pipeline fail.
    - Push your code to GitHub or create a new PullRequest.
    - Notice how the pipeline failed and examine the logs.
    - Fix the introduced error.

### `BONUS` Part 3: Using GitHub Secrets and Variables

#### Task 3.1: Demonstrate using GitHub Secrets and Variables in Workflows

1. Define one **secret** and one **variable** in your GitHub project:
   - In `Settings` of the Repo go to `Secrets and variables` and then `Actions`
   - Add `New repository secret`
   - Add `New repository variable`
2. Write a simple **workflow** where you will attempt to print these two values (the secret value should print as `***`).

   ```yml
    name: secrets
    on: 
        - push
    jobs:
        secret:
            runs-on: ubuntu-latest
            steps:
                - name: using github secret
                  run: |
                    echo "secret: ${{secrets.SECRET}}"
                    echo "variable: ${{vars.VARIABLE}}"
    ```

#### Task 3.2: Extract GitHub Secrets with a Workflow job

1. Write a **workflow** that extracts the secret saved on GitHub and makes it 'visible' to the person who reads the **workflow** (job runner) logs.

   ```yml
    name: secrets_hack

    on: 
      - push

    jobs:
      secret-hack:
        runs-on: ubuntu-latest
        steps:
          - name: printing github secret with python
            shell: python
            env:
              SECRET: ${{secrets.SECRET}}
            run: |
              import os
              for q in (os.getenv("SECRET")):
                print(q)
          - name: printing github secret with sed
            run: echo ${{secrets.SECRET}} | sed 's/./& /g'
          - name: printing github secret with hex
            run: |
              echo "Trick to echo GitHub Actions Secret:  "
              echo "${{secrets.SECRET}}" | xxd -ps
   ```

### Part 4: Deploying to Google Cloud Run with GitHub integration

You will now automate the deployment of your FastAPI app to Google Cloud Run.

#### Task 4.1: Setting Up Google Cloud Run

1. Go to `Google Cloud Console` and navigate to `Cloud Run`
2. Click `Create Service` and follow the steps to `link your GitHub repository` for automatic deployment.
3. Choose `Cloud Build` as the build method. When setting up. Follow instructions to give access to your repo(s) and select the branch that should trigger the deployment (e.g., `main`).
   1. For the `Build Type` you can choose `python` given that this is a python app.
   2. Define the `build context directory`, i.e., 'working directory' (the one from which the source code, requirements, etc are accessible).
   3. Add an entrypoint command like `fastapi run app/main.py` that will be used to start the server. Alternatively, you can use `uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4` to spin up 4 workers on the port 8000 and serve the code from `app/main.py`.
4. Configure the rest of the `Cloud Run`:
   1. Note the `endpoint URL` this is the URL from which you will be able to access your app.
   2. Allow unauthenticated invocations so your URL is publicly available.

#### Task 4.2: Attaching Volumes and Environment Variables

1. While configuring your Cloud Run service, navigate to the `Container(s), Volumes, Networking, Security` tab.

2. Make sure the `container port` is set to the one that will be used by the `entry command` you specified (e.g., if you used `uvicorn` on port `8000` here you should type `8000` as well).

3. If necessary, you can attach persistent volumes (Cloud Storage Buckets) for storing larger files like models. In the `Volumes` section, add a volume (e.g., cloud storage bucket) and then attach/mount it to your container (`Volume mounts` section in the `Edit Container` tab).
    - Note:
    If your google cloud bucket has following structure: `lab5models/models/...`, and you mount the `lab5models` into `/mnt/lab5/` this means that the content of `lab5models` will be placed/mounted inside `/mnt/lab5/`. In other words, in order to access `lab5models/models/...` you would write `/mnt/lab5/models/...`.

4. Under `Edit Container -> Environment Variables`, add any required environment variables, such as model paths.
    - Example:
        - Name: LOGISTIC_MODEL
        - Value: /path/to/logistic_model.pkl

5. Save the configuration and build + deploy the application.

6. Use the (public) link provided by Google Run to test your app.

#### Task 4.3: Test out the automatic deployment

1. Add some changes to your code (e.g., new dummy endpoint).
2. Push the changes to main.
3. Check the automatic build pipeline and the (public) URL once the build is complete

## Conclusion

In this lab, you’ve learned how to implement a robust CI/CD pipeline for FastAPI applications. You started by writing automated tests for your FastAPI app using **pytest** and integrating these tests into a GitHub Actions workflow. You then automated the process of building and deploying Docker images to **Google Cloud Run** using GitHub integration.

By setting up automated testing and continuous deployment, you’ve streamlined the process of releasing new features, ensuring code quality, and minimizing manual effort. This CI/CD workflow is essential for developing scalable, production-ready ML or web applications.

---

## Useful Links

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Pytest Documentation](https://docs.pytest.org/en/6.2.x/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Google Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Docker Documentation](https://docs.docker.com/)
