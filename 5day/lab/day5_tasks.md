# Day 5 Lab: CI/CD for FastAPI Applications

## Table of Contents

- [Day 5 Lab: CI/CD for FastAPI Applications](#day-5-lab-cicd-for-fastapi-applications)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
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
      - [Task 3.2: Extract GitHub Secrets with a Workflow job](#task-32-extract-github-secrets-with-a-workflow-job)
    - [Part 4: Deploying to Google Cloud Run with GitHub Integration](#part-4-deploying-to-google-cloud-run-with-github-integration)
      - [Task 4.1: Setting Up Google Cloud Run](#task-41-setting-up-google-cloud-run)
      - [Task 4.2: Attaching Volumes and Environment Variables](#task-42-attaching-volumes-and-environment-variables)
      - [Task 4.3: Test out the Automatic Deployment](#task-43-test-out-the-automatic-deployment)
  - [Conclusion](#conclusion)
  - [Useful Links](#useful-links)

## Overview

In this lab, you will implement a continuous integration and continuous deployment (CI/CD) pipeline for your FastAPI application. You will write tests using **pytest**, set up a GitHub Actions workflow to automate the testing process, and configure Google Cloud Run to automatically deploy your application when changes are made.

## Learning Objectives

By the end of this lab, you will be able to:

- Write unit tests for FastAPI applications using **pytest**.
- Set up a GitHub Actions pipeline to automate testing.
- Configure continuous deployment using **Google Cloud Run**.

## Lab Instructions

### Part 1: Writing Tests for FastAPI

In this section, you will write unit tests to verify the functionality of your FastAPI application.

#### Task 1.1: Setting up Pytest

1. Install **pytest** and **pytest-mock**.
2. Create a file named `test_main.py` for your tests.
3. Prepare your FastAPI app for testing by importing the `TestClient`.

#### Task 1.2: Writing Unit Tests for FastAPI

1. Write unit tests for the following endpoints:
   - `GET /` (root endpoint).
   - `GET /health` (health check).
   - `GET /models` (list available models).
2. Write tests to verify the behavior of the prediction endpoint:
   - Test prediction with a valid model.
   - Test prediction with an invalid model.
3. Mock the model's `predict` function to simulate the model's behavior during tests.

---

### Part 2: Setting Up GitHub Actions for CI/CD

Next, you will set up a GitHub Actions pipeline to automatically run your tests on every push to the repository.

#### Task 2.1: Creating a GitHub Actions Workflow

1. Create a `.github/workflows` directory in your repository.
2. Create a `test.yml` file inside this directory.
3. Define the GitHub Actions workflow that:
   - Runs on every push to the `main` branch.
   - Sets up Python 3.8.
   - Installs dependencies from `requirements.txt`.
   - Runs your **pytest** tests.

#### Task 2.2: Running Automated Tests

1. Push your code to GitHub.
2. Check the GitHub Actions tab to verify that your pipeline is triggered and runs the tests.
3. Experiment by making a change that causes the tests to fail, then fix the error and rerun the pipeline.

---

### `BONUS` Part 3: Using GitHub Secrets and Variables

#### Task 3.1: Demonstrate using GitHub Secrets and Variables in Workflows

1. Define one **secret** in your GitHub project.
2. Define one **variable** in your GitHub project.
3. Write a simple **workflow** where you will attempt to print these two values (the secret value should print as `***`).

#### Task 3.2: Extract GitHub Secrets with a Workflow job

1. Write a **workflow** that extracts the secret saved on GitHub and makes it 'visible' to the person who reads the **workflow** (job runner) logs.

---

### Part 4: Deploying to Google Cloud Run with GitHub Integration

In this section, you will set up continuous deployment to Google Cloud Run so that your application is automatically deployed whenever changes are pushed to your repository.

#### Task 4.1: Setting Up Google Cloud Run

1. In Google Cloud Console, navigate to **Cloud Run**.
2. Create a new Cloud Run service linked to your GitHub repository.
3. Configure **Cloud Build** to automatically build and deploy your Docker container from the `main` branch. If needed, add any startup commands that your project requires.
4. Allow unauthenticated invocations so your URL is publicly available.

#### Task 4.2: Attaching Volumes and Environment Variables

1. Set the appropriate container port.
2. Attach a Google Cloud Storage bucket as a volume if your application requires persistent storage for models or other files.
3. Configure environment variables for your application, such as model paths.

#### Task 4.3: Test out the Automatic Deployment

1. Make a small change to your code (e.g., add a new endpoint).
2. Push the change to the `main` branch.
3. Verify that Google Cloud Run automatically rebuilds and redeploys your application.
4. Test the live version of your application using the provided public URL.

## Conclusion

In this lab, you implemented a CI/CD pipeline for a FastAPI application. You wrote unit tests, set up automated testing with GitHub Actions, and configured Google Cloud Run for continuous deployment. By automating testing and deployment, you created a streamlined development workflow for fast, reliable updates to your application.

## Useful Links

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Pytest Documentation](https://docs.pytest.org/en/6.2.x/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Google Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Docker Documentation](https://docs.docker.com/)
