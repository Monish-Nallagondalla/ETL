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
- **Apache Airflow**: Orchestrates the ETL pipeline using a DAG to manage task dependencies and scheduling.
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
- Docker and Docker Compose installed.
- An active NASA API key (obtain from [NASA API Portal](https://api.nasa.gov/)).
- DBeaver installed for database inspection.
- Astronomer CLI installed (`pip install astro-sdk`).

## Setup Instructions

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Monish-Nallagondalla/ETL.git
   cd ETL
   ```

2. **Set Up the NASA API Key in Airflow**:
   - Access the Airflow UI at `http://localhost:8080`.
   - Navigate to **Admin > Connections**.
   - Create a new connection for NASA API:
     - **Conn Id**: `nasa_api`
     - **Conn Type**: HTTP
     - **Host**: `https://api.nasa.gov`
     - **Extra**: `{"api_key": "<your-nasa-api-key>"}`
   - Create a connection for Postgres:
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
   - This uses the Astronomer CLI to initialize the Airflow environment and bring up services defined in `docker-compose.yml`.

4. **Access the Airflow UI**:
   - Open `http://localhost:8080` in your browser.
   - Log in with default credentials (username: `admin`, password: `admin`).
   - Enable the `nasa_apod_postgres` DAG to start the pipeline.

5. **Verify Data in DBeaver**:
   - Open DBeaver and create a new connection:
     - **Database Type**: PostgreSQL
     - **Host**: `localhost`
     - **Port**: `5432`
     - **Database**: `postgres`
     - **Username**: `postgres`
     - **Password**: `postgres`
   - Connect to the database and query the `apod_data` table to verify the data.

## Project Structure
```
├── dags/
│   └── etl.py              # Airflow DAG definition for the ETL pipeline
├── docker-compose.yml      # Docker Compose configuration for Airflow and Postgres
├── README.md               # Project documentation
└── ...                     # Other Astronomer-generated files
```

## DAG Details
The DAG (`nasa_apod_postgres`) is defined in `dags/etl.py` and includes:
- **create_table**: Creates the `apod_data` table in Postgres if it doesn't exist.
- **extract_apod**: Fetches data from the NASA APOD API using `SimpleHttpOperator`.
- **transform_apod_data**: Processes the API response to extract relevant fields.
- **load_data_to_postgres**: Inserts the transformed data into the Postgres table.

### Task Dependencies
```
create_table >> extract_apod >> transform_apod_data >> load_data_to_postgres
```

## Running the Pipeline
1. Ensure Docker containers are running (`docker ps`).
2. In the Airflow UI, enable the `nasa_apod_postgres` DAG.
3. Trigger the DAG manually by clicking the "Run" button or wait for the scheduled run (`@daily` with `catchup=False`).
4. Monitor task execution in the Airflow UI to confirm successful completion.

## Inspecting Data
- In DBeaver, connect to the Postgres database and run:
  ```sql
  SELECT * FROM apod_data;
  ```

## Troubleshooting
- **Airflow UI Not Accessible**: Verify Docker containers are running (`docker ps`) and the webserver is up.
- **API Key Issues**: Ensure the `nasa_api` connection in Airflow has the correct API key.
- **Database Connection Errors**: Confirm Postgres connection details in DBeaver and Airflow match `docker-compose.yml`.
- **DAG Not Running**: Check Airflow logs for errors and ensure the DAG is enabled.

## Future Improvements
- Add error handling for API failures or invalid responses.
- Implement data validation during transformation.
- Add logging or notifications for pipeline success/failure.
- Extend the pipeline to include additional API endpoints or data sources.

## License
This project is licensed under the GPL-2.0 license.

## Repository
The source code is available at: [https://github.com/Monish-Nallagondalla/ETL.git](https://github.com/Monish-Nallagondalla/ETL.git)
