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

EXPOSE 8080

# Run the FastAPI application
CMD ["fastapi", "run", "app/main.py", "--port", "8080"]