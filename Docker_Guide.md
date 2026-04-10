# ec2-airflow-data-pipeline-sk -- EC2 Name

# airflow_pipeline_alerts_sk -- SNS topic name

# mp-airflow-pipeline-raw-data-sk -- s3 name

# ec2-airflow-role-sk -- i am 'role' name

-- create i am role give full access of s3 and sns

-- call it in ec2 security i am role and select it

---

# dag inside of docker

cd ~/airflow-pipeline/dags
nano fmp_etl_pipeline.py -- dag name

---

# 📊 FMP Stock Data Pipeline with Airflow, Snowflake & AWS S3

A complete data pipeline that fetches stock data from Financial Modeling Prep (FMP) API, stores it in AWS S3, and loads it into Snowflake using Apache Airflow.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [EC2 Setup](#ec2-setup)
- [Docker Installation](#docker-installation)
- [Apache Airflow Setup](#apache-airflow-setup)
- [Custom Docker Configuration](#custom-docker-configuration)
- [Airflow Web Interface](#airflow-web-interface)
- [Snowflake Configuration](#snowflake-configuration)
- [Complete SQL Scripts](#complete-sql-scripts)
- [Troubleshooting](#troubleshooting)

---

## 🔧 Prerequisites

- AWS EC2 instance (Ubuntu 20.04+)
- Snowflake account
- FMP API key
- Basic knowledge of AWS S3, Snowflake, and Airflow

---

## 🚀 EC2 Setup

### Step 1: Connect to your EC2 Instance

```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip


# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io

# Start Docker service
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Add user to docker group (avoids using 'sudo' every time)
sudo usermod -aG docker ubuntu

# Apply group changes
newgrp docker

# Verify Docker is working
docker ps
# ✅ You should see several containers running

# Install Docker Compose:
sudo apt install -y docker-compose-v2

# Create a Project Folder:
mkdir airflow-pipeline && cd airflow-pipeline
mkdir -p ./dags ./logs ./plugins ./config

# Download the official Compose file:
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml'

# Set Airflow User ID:
echo -e "AIRFLOW_UID=$(id -u)" > .env

# Initialize the Database:
docker compose up airflow-init

# Start Airflow
docker compose up -d

# Professional tip: Validate YAML configuration
docker compose config

# Create Dockerfile
nano Dockerfile

# Use official Airflow image as base
file code below
FROM apache/airflow:3.2.0

USER root

# System dependencies for Snowflake and standard build tools
RUN apt-get update && apt-get install -y \
    build-essential \
    libssl-dev \
    libffi-dev \
    python3-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER airflow

# Install Airflow 3 compatible providers and libraries
RUN pip install --no-cache-dir \
    apache-airflow-providers-amazon \
    apache-airflow-providers-snowflake \
    boto3 \
    requests \
    snowflake-connector-python \
    snowflake-sqlalchemy

----------------------------

# Fix indentation and add build configuration
sed -i '/# image: ${AIRFLOW_IMAGE_NAME:-apache\/airflow:2.10.2}/a\  build: .' docker-compose.yaml

# Initialize and rebuild with custom Dockerfile
docker compose init
docker compose up -d --build

URL: http://your-ec2-public-ip:8080
Username: airflow
Password: airflow

# After Login, Configure Connections

1️⃣ AWS Connection
Conn Id: aws_default

Conn Type: Amazon Web Services

AWS Access Key ID: your-access-key

AWS Secret Access Key: your-secret-key

Region: your-region (e.g., us-east-1)

# 2️⃣ Snowflake Connection
Conn Id: snowflake_default # check name inside of your dag

Conn Type: Snowflake

Account: your-account-identifier

User: your-username

Password: your-password

Database: your data base name
Schema: your schema name

Warehouse: your-warehouse

# 3️⃣ Create Variables for API Endpoint
Key: fmp_api_endpoint

Val: https://financialmodelingprep.com/api/v3/quote/

Key: fmp_api_key

Val: your-fmp-api-key



# after creating snowflake connection create hand shake process and create a stage on snowflake

CREATE OR REPLACE STORAGE INTEGRATION S3_INTEGRATION
  TYPE = EXTERNAL_STAGE           -- Defines this as an external storage integration
  STORAGE_PROVIDER = 'S3'         -- Using AWS S3 as the storage provider
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::***********:role/ec2-airflow-sk'  -- IAM role with S3 access
  ENABLED = TRUE                  -- Enable this integration
  STORAGE_ALLOWED_LOCATIONS = ('s3://fmp-airflow-pipeline-raw-data-sk/');  -- Allowed S3 bucket paths

# stage for snow falke
  # Create or replace an external stage named my_s3_stage pointing to the S3 bucket
CREATE OR REPLACE STAGE my_s3_stage
  URL = 's3://fmp-airflow-pipeline-raw-data-sk/'  -- S3 bucket URL
  STORAGE_INTEGRATION = S3_INTEGRATION;           -- Use the storage integration for authentication

# Describe the storage integration to check its properties and status
DESC STORAGE INTEGRATION S3_INTEGRATION;


# List all files in the external stage (to verify connection and see available files)
LIST @my_s3_stage;




airflow-pipeline/
├── dags/                 # Airflow DAG files
├── logs/                 # Airflow logs
├── plugins/              # Custom plugins
├── config/               # Configuration files
├── Dockerfile            # Custom Docker configuration
├── docker-compose.yaml   # Airflow compose file
└── .env                  # Environment variables
```
