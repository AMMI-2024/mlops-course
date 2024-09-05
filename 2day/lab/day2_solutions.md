# Day 2 Lab: Machine Learning Lifecycle Management

**Table of contents:**

- [Day 2 Lab: Machine Learning Lifecycle Management](#day-2-lab-machine-learning-lifecycle-management)
  - [Theory Overview](#theory-overview)
    - [ML Model Development and Lifecycle](#ml-model-development-and-lifecycle)
    - [Experiment Tracking and Management](#experiment-tracking-and-management)
    - [Version Control for Data and Models](#version-control-for-data-and-models)
    - [Tools and Frameworks for ML Lifecycle Management](#tools-and-frameworks-for-ml-lifecycle-management)
  - [Part 1: Version Control Data and Models with DVC](#part-1-version-control-data-and-models-with-dvc)
    - [Task 1: Initialize DVC and Track Data](#task-1-initialize-dvc-and-track-data)
    - [`[BONUS]` Task 2: Storing DVC Data Remotely](#bonus-task-2-storing-dvc-data-remotely)
    - [Task 3: Make Local Changes to the Dataset](#task-3-make-local-changes-to-the-dataset)
  - [Part 2: Integrating DVC with Cloud Storage](#part-2-integrating-dvc-with-cloud-storage)
    - [Task 1: Integrate DVC with Cloud Storage](#task-1-integrate-dvc-with-cloud-storage)
      - [Using Azure Blob Storage with DVC](#using-azure-blob-storage-with-dvc)
      - [Using Google Cloud Storage (GCS) with DVC](#using-google-cloud-storage-gcs-with-dvc)
      - [Using AWS S3 with DVC](#using-aws-s3-with-dvc)
  - [Part 3: Set Up Experiment Tracking with MLflow](#part-3-set-up-experiment-tracking-with-mlflow)
    - [Task 1: Install and Set Up MLflow](#task-1-install-and-set-up-mlflow)
    - [`[BONUS]` Task 2: Log Experiments Using MLflow](#bonus-task-2-log-experiments-using-mlflow)
      - [Autologging](#autologging)
      - [Manual Logging](#manual-logging)
  - [Part 4: Integrate MLflow with Azure ML](#part-4-integrate-mlflow-with-azure-ml)
    - [Task 1: Log Experiments in Azure](#task-1-log-experiments-in-azure)
    - [Task 2: Deploy an MLflow Model to Azure ML](#task-2-deploy-an-mlflow-model-to-azure-ml)
  - [Lab Wrap-Up: What We Learned](#lab-wrap-up-what-we-learned)
  - [Bonus Material](#bonus-material)
    - [Best Practices and Useful Links](#best-practices-and-useful-links)

## Theory Overview

### ML Model Development and Lifecycle

- **ML Model Development**: The process involves several stages from data collection, preprocessing, model training, evaluation, deployment, and monitoring.
  - **Lifecycle Management**: Managing the lifecycle ensures that models are reproducible, scalable, and maintainable.

### Experiment Tracking and Management

- **Experiment Tracking**: The process of logging experiments, including hyperparameters, metrics, and artifacts, to facilitate analysis and comparison.
  - **Tools**: MLflow, Weights & Biases, TensorBoard.

### Version Control for Data and Models

- **Version Control for Data**: Tracking changes to datasets over time using tools like DVC.
- **Version Control for Models**: Managing different versions of models to maintain a history of changes and facilitate rollback if necessary.

### Tools and Frameworks for ML Lifecycle Management

- **MLflow**: An open-source platform for managing the ML lifecycle, including experimentation, reproducibility, and deployment.
- **DVC**: Data Version Control (DVC) is a tool for managing datasets and model versions with Git-like functionality.

## Part 1: Version Control Data and Models with DVC

### Task 1: Initialize DVC and Track Data

**Objective**: Learn how to use DVC to version control datasets.

**Instructions**:

  1. Install DVC:

     ```bash
     pip install dvc
     ```

  2. Initialize DVC in your Git repository (make sure to be in the root folder) and commit the changes:

     ```bash
     dvc init
     git commit -m "Initialize DVC"
     ```

  3. Start tracking a local dataset with DVC (add it to DVC + push changes to git):

     ```bash
     dvc add data/iris.json
     git add data/iris.json.dvc data/.gitignore
     git commit -m "Track Iris dataset with DVC"
     ```

**Note**: You can look into the `name_of_your_dataset.dvc` metafile created for you local dataset. It contains a md5 hash value that is used for referencing the file. If you check the cache inside the `.dvc` directory you can see the file there with the name of the hash value. On GitHub only the metafile is stored/tracked.

### `[BONUS]` Task 2: Storing DVC Data Remotely

**Objective**: Understand how to store and retrieve data using DVC remote storage.

**Instructions:**

  1. Create a "local remote" folder (i.e., a directory in the local file system) to serve as storage. Make sure it is ignored by `.gitignore` if set inside the git repo.

     ```bash
     mkdir /tmp/dvc-store # directory to serve as local remote storage
     ```

  2. Push the data to the "local remote":

     ```bash
     dvc remote add -d myremote /tmp/dvc-store
     dvc push
     ```

  3. Test the storage. Delete the data from your initial data folder and `.dvc` cache, then pull it from 'remote':

     ```bash
     rm -rf .dvc/cache
     rm -f data/iris.json
     dvc pull
     ```

  4. Check that the data was correctly pulled from the local folder that serves as storage.

**Note**: The `-d` flag of the `dvc remote add` flag sets the `--default` remote to be used when executing `push`, `pull`, `status`, etc. If you want to use a different remote with these functions you can always provide the `--remote` (`-r`) flag to them.

### Task 3: Make Local Changes to the Dataset

**Objective**: Learn how to manage dataset versions with DVC after making changes.

**Instructions:**

  1. Make some changes to the local data (e.g., duplicate/remove a row).

  2. Track the latest version:

     ```bash
     dvc add data/iris.json
     ```

  3. Push the changes to the remote and commit the new metafile to git:

     ```bash
     dvc push
     git commit data/iris.json.dvc -m "Dataset updates"
     ```

## Part 2: Integrating DVC with Cloud Storage

### Task 1: Integrate DVC with Cloud Storage

**Objective**: Learn how to set up and use DVC with different cloud storage providers (AWS S3, Azure Blob Storage, and Google Cloud Storage).

#### Using Azure Blob Storage with DVC

1. **Set Up Azure Blob Storage**:
   - Create a storage account and container in Azure Blob Storage.

2. **Configure DVC to Use Azure Blob Storage**:
   - Install Azure CLI if not already installed and log in:

     ```bash
     brew update && brew install azure-cli # on Mac
     az login
     ```

   - Configure DVC to use Azure Blob Storage:

     ```bash
     dvc remote add azure-remote azure://<your-container-name>
     dvc remote modify --local azure-remote account_name <your-storage-account-name>
     dvc remote modify --local azure-remote account_key <your-storage-account-key>
     ```

   - **Note:** It is very important that you use the `--local` flag when adding sensitive information. The `config.local` file is not tracked by git, but the `config` file is tracked and you should never leak your credentials.

3. **Push and Pull Data**:
   - Push data to Azure Blob Storage (install `dvc-azure` if required):

     ```bash
     pip install dvc-azure
     dvc push -r azure-remote
     ```

   - Pull data from Azure Blob Storage:

     ```bash
     dvc pull -r azure-remote
     ```

#### Using Google Cloud Storage (GCS) with DVC

1. **Set Up Google Cloud Storage**:
   - Create a Google Cloud project and storage bucket.

2. **Configure DVC to Use GCS**:
   - Install Google Cloud SDK and authenticate:

     ```bash
     gcloud init
     gcloud auth application-default login
     ```

   - Configure DVC to use GCS:

     ```bash
     dvc remote add gs-remote gs://<your-bucket-name>
     ```

3. **Push and Pull Data**:
   - Push data to Google Cloud Storage (install `dvc-gs` if required):

     ```bash
     pip install dvc-gs
     dvc push -r gs-remote
     ```

   - Pull data from Google Cloud Storage:

     ```bash
     dvc pull -r gs-remote
     ```

#### Using AWS S3 with DVC

1. **Set Up AWS S3 Bucket**:
   - Create an S3 bucket in your AWS account to store datasets and models.

2. **Configure DVC to Use S3**:
   - Install AWS CLI and configure your AWS credentials if not already done. Create access key.

     ```bash
     pip install awscli
     aws configure
     ```

   - Add an S3 remote storage in your DVC project:

     ```bash
     dvc remote add s3-remote s3://<your-bucket-name>
     dvc remote modify s3-remote access_key_id <your-access-key-id>
     dvc remote modify s3-remote secret_access_key <your-secret-access-key>
     ```

3. **Push and Pull Data**:
   - Push data to AWS (install `dvc-s3` if required):

     ```bash
     pip install dvc-s3
     dvc push -r s3-remote
     ```

   - Pull data from AWS:

     ```bash
     dvc pull -r s3-remote
     ```

## Part 3: Set Up Experiment Tracking with MLflow

### Task 1: Install and Set Up MLflow

**Objective**: Set up MLflow for logging experiments and tracking model performance.

**Instructions:**

  1. Install MLflow:

     ```bash
     pip install mlflow
     ```

  2. Set up an MLflow tracking server:

     ```bash
     mlflow ui
     ```

     - This will start the MLflow UI, usually accessible at `http://127.0.0.1:5000`.

### `[BONUS]` Task 2: Log Experiments Using MLflow

**Objective**: Learn how to log experiments, including parameters, metrics, and artifacts.

**Instructions:**

#### Autologging

  1. Create a new Python script `mlflow_autologging.py`:

     ```python
      import mlflow
      from sklearn.model_selection import train_test_split
      from sklearn.datasets import load_diabetes
      from sklearn.ensemble import RandomForestRegressor

      # Set our tracking server uri for logging
      mlflow.set_tracking_uri(uri="http://127.0.0.1:5000")

      # Set the experiment name
      mlflow.set_experiment("MLflow Autologging Quickstart")

      # Enable autologging
      mlflow.autolog()

      db = load_diabetes()

      X_train, X_test, y_train, y_test = train_test_split(db.data, db.target)

      # Create and train models
      rf = RandomForestRegressor(n_estimators=100, max_depth=6, max_features=3)
      rf.fit(X_train, y_train)

      # Use the model to make predictions on the test dataset
      predictions = rf.predict(X_test)
     ```
  
  2. Run your script and then navigate to the MLflow UI to visualize the logged experiments and compare different runs.

#### Manual Logging

  1. Modify a model training script to log experiments with MLflow.
  2. Example code to log experiments:

     ```python
      import mlflow
      import mlflow.sklearn
      from sklearn.model_selection import train_test_split
      from sklearn.linear_model import LogisticRegression
      from sklearn.datasets import load_iris
      from sklearn.metrics import accuracy_score
      from mlflow.models import infer_signature


      def load_dataset():
          iris = load_iris()
          X_train, X_test, y_train, y_test = train_test_split(
              iris.data, iris.target, test_size=0.2, random_state=42
          )
          return X_train, X_test, y_train, y_test


      def train_and_log_model(params):
          X_train, X_test, y_train, y_test = load_dataset()

          model = LogisticRegression(**params)

          with mlflow.start_run():

              model.fit(X_train, y_train)
              accuracy = inference(model, X_test, y_test)

              # Log the hyperparameters
              mlflow.log_params(params)

              # Log the loss metric
              mlflow.log_metric("accuracy", accuracy)

              # Set a tag that we can use to remind ourselves what this run was for
              mlflow.set_tag("Training Info", "Basic LR model for iris data")

              # Infer the model signature
              signature = infer_signature(X_train, model.predict(X_train))

              # Log the model
              model_info = mlflow.sklearn.log_model(
                  sk_model=model,
                  artifact_path="model",
                  signature=signature,
                  input_example=X_train,
                  registered_model_name="tracking-quickstart",
              )

          return model_info


      def inference(model, X_test, y_test):
          predictions = model.predict(X_test)
          accuracy = accuracy_score(y_test, predictions)
          print(f"Accuracy of the model is: {accuracy}.")

          return accuracy


      if __name__ == "__main__":
          # Set our tracking server uri for logging
          mlflow.set_tracking_uri(uri="http://127.0.0.1:5000")

          # Create a new MLflow Experiment
          mlflow.set_experiment("MLflow Quickstart")

          # Define the model hyperparameters
          params = {
              "solver": "lbfgs",
              "max_iter": 1000,
              "multi_class": "auto",
              "random_state": 8888,
          }
          model_info = train_and_log_model(params)

          X_train, X_test, y_train, y_test = load_dataset()

          # Load the model back for predictions as a generic Python Function model
          loaded_model = mlflow.pyfunc.load_model(model_info.model_uri)
          accuracy = inference(loaded_model, X_test, y_test)
     ```

  3. Run your script and then navigate to the MLflow UI to visualize the logged experiments and compare different runs.

## Part 4: Integrate MLflow with Azure ML

### Task 1: Log Experiments in Azure

**Objective**: Configure MLflow to log experiments to Azure .

**Instructions**:

  1. **Set Up Azure ML Workspace**:
      - Create an Azure Machine Learning workspace in the Azure portal and set up your environment.

  2. **Configure MLflow to Use Azure ML**:
      - Make sure you are logged in with Azure CLI:

       ```bask
        az login
        ```

      - Install `azureml-core` and `azureml-mlflow`:

        ```bash
        pip install azureml-core azureml-mlflow
        ```

      - In your MLflow tracking script, set the tracking URI to your ML Workspace:

        ```python
        from azureml.core import Workspace

        workspace = Workspace.from_config("PATH_TO_YOUR_CONFIG")

        # Set the tracking server uri for logging
        mlflow.set_tracking_uri(workspace.get_mlflow_tracking_uri())
        ```

      - Run the python script and check your job in Azure ML Jobs:

### Task 2: Deploy an MLflow Model to Azure ML

**Objective**: Deploy a trained MLflow model to Azure Machine Learning.

**Instructions**:

   1. **Register a new model in ML Azure**:
      - Register a model from a job output (in Task 2 we have run the MLFlow job).

## Lab Wrap-Up: What We Learned

- **Experiment Tracking**: Learned how to use MLflow and Weights & Biases to log experiments, track parameters, metrics, and artifacts, and visualize results.
- **Data and Model Version Control**: Gained hands-on experience with DVC for version-controlling data and models, ensuring traceability and reproducibility.
- **Cloud Integration**: Learned to use cloud storage and cloud-based experiment tracking tools to manage datasets, models, and experiment logs in a scalable manner.
- **Azure ML Integration**: Implemented logging and deployment of models using Azure ML to enhance reproducibility and manageability.

## Bonus Material

### Best Practices and Useful Links

- **Experiment Tracking**: Always log your experiments with sufficient detail to ensure reproducibility and ease of comparison.
- **Model Checkpointing**: Save model checkpoints regularly during training to prevent data loss and facilitate model reuse.
- **Useful links**:
  - [MLflow Documentation](https://www.mlflow.org/docs/latest/index.html)
  - [DVC Documentation](https://dvc.org/doc)
  - [Setting up MLFlow on GCP](https://dlabs.ai/blog/a-step-by-step-guide-to-setting-up-mlflow-on-the-google-cloud-platform/)
  - [Setting up MLFlow on AWS](https://medium.com/ama-tech-blog/mlflow-on-aws-a-step-by-step-setup-guide-8601414dd6ea)
