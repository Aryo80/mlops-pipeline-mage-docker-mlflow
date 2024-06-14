# Mage AI Pipeline with Docker and MLflow






This repository contains a comprehensive walkthrough of setting up and running a data pipeline using Mage AI, Docker Compose, and MLflow. The pipeline involves data ingestion, preparation, model training with linear regression, and model registration using MLflow for reproducibility.

![image](https://github.com/Aryo80/mlops-zoomcamp-mage/assets/55058593/3750d9d1-0e06-45a2-91d1-106416d7dc5b)

## Overview

Mage AI is a powerful tool for building and deploying machine learning pipelines. In this project, we demonstrate:

- Setting up Mage AI with Docker Compose.
- Creating a project and pipeline within Mage AI.
- Ingesting and preparing data from March 2023 Yellow taxi trips.
- Training a linear regression model using `DictVectorizer`.
- Saving the model and artifacts with MLflow.

## Walkthrough Steps

1. **Setup Mage AI with Docker**
   - Install Docker and Docker Compose.
   - Clone this repository and navigate to the project directory.
   - Start Mage AI using Docker Compose:
   ## Setting up Mage AI with Docker

Start Mage AI using Docker Compose. Create a Dockerfile (`mlflow.dockerfile`) with the following content:

```dockerfile
FROM python:3.10-slim

RUN pip install mlflow==2.12.1

EXPOSE 5000

CMD [ \
    "mlflow", "server", \
    "--backend-store-uri", "sqlite:///home/mlflow/mlflow.db", \
    "--host", "0.0.0.0", \
    "--port", "5000" \
]
```

Create a Dockerfile (`Dockerfile.dockerfile`) with the following content:
```dockerfile
FROM mageai/mageai:alpha

ARG PROJECT_NAME=mlops
ARG MAGE_CODE_PATH=/home/src
ARG USER_CODE_PATH=${MAGE_CODE_PATH}/${PROJECT_NAME}

WORKDIR ${MAGE_CODE_PATH}

COPY ${PROJECT_NAME} ${PROJECT_NAME}

ENV USER_CODE_PATH=${USER_CODE_PATH}

# Install custom Python libraries and dependencies for your project.
RUN pip3 install -r ${USER_CODE_PATH}/requirements.txt

ENV PYTHONPATH="${PYTHONPATH}:${MAGE_CODE_PATH}/${PROJECT_NAME}"

# Installing necessary utilities and Terraform.
# Uncomment the following lines if you want to use Terraform in Docker.
# RUN apt-get update && \
#   apt-get install -y wget unzip && \
#   wget https://releases.hashicorp.com/terraform/1.8.3/terraform_1.8.3_linux_amd64.zip && \
#   unzip terraform_1.8.3_linux_amd64.zip -d /usr/local/bin/ && \
#   rm terraform_1.8.3_linux_amd64.zip

CMD ["/bin/sh", "-c", "/app/run_app.sh"]
```
 YAML file  (`docker-compose.yml`) with the following content:
```dockerfile
services:
  magic-platform:
    env_file:
      - .env.dev
    build:
      context: .
    command: /app/run_app.sh mage start $PROJECT_NAME
    ports:
      - 6789:6789
    volumes:
      # Mount your local codebase to the container.
      - .:/$MAGE_CODE_PATH
      # Store the data output on local machine to easily debug (optional).
      - ~/.mage_data:/$MAGE_CODE_PATH/mage_data
      # Initial credentials to create an IAM user with limited permissions for deployment.
      - ~/.aws:/root/.aws
      # Local machine’s SSH keys to pull and push to your GitHub repository.
      - ~/.ssh:/root/.ssh:ro
      # Local machine’s GitHub configs
      - ~/.gitconfig:/root/.gitconfig:ro
    restart: on-failure:5
    networks:
      - app-network
    depends_on:
      - magic-database
    stdin_open: true # used for interactive debugging
    tty: true # used for interactive debugging
  magic-database:
    image: pgvector/pgvector:0.6.0-pg16
    env_file:
      - .env.dev
    ports:
      - 5432:5432
    volumes:
      - ~/.postgres/data:/var/lib/postgresql/data
      # Custom database initialization scripts (optional).
      - ./scripts/database:/docker-entrypoint-initdb.d
    restart: always
    networks:
      - app-network
  mlflow:
    build:
      context: .
      dockerfile: mlflow.dockerfile
    ports:
      - "5000:5000"
    volumes:
      - "${PWD}/mlflow:/home/mlflow/"
    networks:
      - app-network
networks:
  app-network:
    driver: bridge
```
2. **Create a Project and Pipeline**
   - Access Mage AI UI (http://localhost:5000).
   - Create a new project named "homework_03".
   - Develop an ingestion code block for Yellow taxi trips data.

3. **Data Preparation**
   - Implement a transformer block to clean and prepare the dataset.
   - Ensure data quality and consistency for model training.

4. **Model Training**
   - Train a linear regression model using `DictVectorizer` for categorical features.
   - Use default parameters and separate pick-up/drop-off locations.

5. **MLflow Integration**
   - Save the trained model and `DictVectorizer` artifacts using MLflow.
   - Ensure model reproducibility and version control with MLflow tracking.
     ![image](https://github.com/Aryo80/mlops-zoomcamp-mage/assets/55058593/8f777a63-9d2c-45f9-abc0-ac4398376093)


## Technical Highlights

- **Technology Stack**: Mage AI, Docker, MLflow, Python, Pandas, Scikit-learn.
- **Artifact Management**: Utilize MLflow for logging and tracking model artifacts.
- **Data Processing**: Implement data transformations and cleaning for effective model training.
- **Scalability**: Docker Compose setup for scalable and reproducible environment management.

## Next Steps

- Explore advanced Mage AI functionalities for more complex pipelines.
- Experiment with different models and hyperparameters using Mage AI's flexible framework.
- Integrate additional data sources and improve pipeline robustness.



## Author

- [Aryo Honarvar](https://www.linkedin.com/posts/aryo-honarvar-5a349754_mageai-machinelearning-datascience-activity-7207159854644232195-xj67?utm_source=share&utm_medium=member_desktop)

Feel free to clone this repository and follow along with the walkthrough to enhance your understanding of building data pipelines with Mage AI, Docker, and MLflow!
