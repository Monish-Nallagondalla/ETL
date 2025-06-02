
# NASA APOD ETL Pipeline with Apache Airflow and Postgres

This project implements an ETL (Extract, Transform, Load) pipeline using Apache Airflow to fetch data from NASA's Astronomy Picture of the Day (APOD) API, transform it, and load it into a PostgreSQL database. The pipeline is orchestrated using Airflow, running in a Dockerized environment set up with Astronomer. The data is stored in a Postgres database and can be inspected using DBeaver.

## Project Overview

### Objectives
- **Extract**: Retrieve daily astronomy data from NASA's APOD API.
- **Transform**: Process the JSON response to extract relevant fields (`title`, `explanation`, `url`, `date`, `media_type`).
- **Load**: Store the transformed data in a PostgreSQL database.
- **Orchestration**: Use Apache Airflow to schedule and manage the ETL workflow.
- **Visualization**: Inspect the loaded data using DBeaver.

### Key Components
- **Apache Airflow**: Orchestrates the ETL pipeline using a DAG (Directed Acyclic Graph) to manage task dependencies and scheduling.
- **PostgreSQL**: Stores the extracted and transformed data in a table named `apod_data`.
- **NASA APOD API**: Provides astronomy-related data, including the title, explanation, image URL, date, and media type.
- **Docker**: Runs Airflow and Postgres in isolated containers for a reproducible environment.
- **DBeaver**: Used to connect to the Postgres database and verify the loaded data.

### Architecture
The ETL pipeline consists of the following tasks:
1. **Create Table**: Creates the `apod_data` table in Postgres if it doesn't exist.
2. **Extract**: Uses `SimpleHttpOperator` to fetch data from the NASA APOD API.
3. **Transform**: Processes the JSON response to extract relevant fields using Airflow's TaskFlow API.
4. **Load**: Inserts the transformed data into the Postgres database using `PostgresHook`.

## Prerequisites
- Docker and Docker Compose installed on your system.
- An active NASA API key (freely available from [NASA API Portal](https://api.nasa.gov/)).
- DBeaver installed for database inspection.
- Astronomer CLI installed (`pip install astro-sdk`).

## Setup Instructions

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Set Up the NASA API Key in Airflow**:
   - Access the Airflow UI (default: `http://localhost:8080`).
   - Navigate to **Admin > Connections** in the Airflow UI.
   - Create a new connection:
     - **Conn Id**: `nasa_api`
     - **Conn Type**: HTTP
     - **Host**: `https://api.nasa.gov`
     - **Extra**: `{"api_key": "<your-nasa-api-key>"}`
   - Create another connection for Postgres:
     - **Conn Id**: `my_postgres_connection`
     - **Conn Type**: Postgres
     - **Host**: `postgres`
     - **Schema**: `postgres`
     - **Login**: `postgres`
     - **Password**: `postgres`
     - **Port**: `5432`

3. **Start the Docker Services**:
   - Run the following command to start Airflow and Postgres:
     ```bash
     astro dev start
     ```
   - This command uses the Astronomer CLI to initialize the Airflow environment and bring up the services defined in `docker-compose.yml`.

4. **Access the Airflow UI**:
   - Open your browser and navigate to `http://localhost:8080`.
   - Log in with the default credentials (username: `admin`, password: `admin`).
   - Enable the `nasa_apod_postgres` DAG in the Airflow UI to start the pipeline.

5. **Verify Data in DBeaver**:
   - Open DBeaver and create a new connection:
     - **Database Type**: PostgreSQL
     - **Host**: `localhost`
     - **Port**: `5432`
     - **Database**: `postgres`
     - **Username**: `postgres`
     - **Password**: `postgres`
   - Connect to the database and query the `apod_data` table to verify the loaded data.

## Project Structure
```
├── dags/
│   └── etl.py              # Airflow DAG definition for the ETL pipeline
├── docker-compose.yml      # Docker Compose configuration for Airflow and Postgres
├── README.md               # This file
└── ...                     # Other Astronomer-generated files
```

## DAG Details
The DAG (`nasa_apod_postgres`) is defined in `dags/etl.py` and includes the following tasks:
- **create_table**: Creates the `apod_data` table in Postgres if it doesn't exist.
- **extract_apod**: Fetches data from the NASA APOD API using `SimpleHttpOperator`.
- **transform_apod_data**: Processes the API response to extract relevant fields.
- **load_data_to_postgres**: Inserts the transformed data into the Postgres table.

### Task Dependencies
```
create_table >> extract_apod >> transform_apod_data >> load_data_to_postgres
```

## Running the Pipeline
- The DAG is scheduled to run daily (`@daily`) with `catchup=False`.
- Trigger the DAG manually from the Airflow UI or wait for the scheduled run.
- Monitor the task execution in the Airflow UI to ensure all tasks complete successfully.

## Inspecting Data
- Use DBeaver to connect to the Postgres database and run the following query to view the data:
  ```sql
  SELECT * FROM apod_data;
  ```

## Troubleshooting
- **Airflow UI Not Accessible**: Ensure Docker containers are running (`docker ps`) and the Airflow webserver is up.
- **API Key Issues**: Verify the `nasa_api` connection in the Airflow UI has the correct API key.
- **Database Connection Errors**: Confirm the Postgres connection details in DBeaver and Airflow match the `docker-compose.yml` configuration.
- **DAG Not Running**: Check the Airflow logs for errors and ensure the DAG is enabled in the UI.

## Future Improvements
- Add error handling for API failures or invalid responses.
- Implement data validation during the transformation step.
- Add logging or notifications for pipeline success/failure.
- Extend the pipeline to handle multiple API endpoints or additional data sources.

## License
This project is licensed under the .

