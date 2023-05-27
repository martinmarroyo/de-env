# Data Pipeline Environment

## Getting Started

> #### Prerequisites
> - [Docker & Docker Compose](https://docs.docker.com/get-docker/)
> - [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) (If you're on Windows)

In your terminal, cd to this project's root folder where the `docker-compose.yml` file lives. Then run `docker compose up -d`. That's it. It will likely take a couple of minutes to get the services started, but after that you should be good to go.

Read through the descriptions of the services defined in the `docker-compose.yml` file to get a sense of how to access each one. It's worth noting that all of the containers are in the same docker network, so they can communicate with one another. For example, if you are writing code to connect to the `warehouse` container and work with that data, you would use `warehouse` as the host/server URL in the connection parameters.  All container names are listed in the next section.

## Services

These are all of the containers/services defined in the `docker-compose.yml` file under the `services` section. In general, you can enter any one of these containers using `docker exec -it <container-name> bash`

> Example:
>
> ```bash docker exec -it ingest bash```

- `ingest`: A Python container used to control the ingestion process
    - Access the container's Jupyter Notebook server in your browser at http://localhost:8888/tree?token=letmein
- `plotly`: A Python container used to display a Plotly-Dash Dashboard
    - Access the container's Jupyter Notebook server in your browser at http://localhost:8877/tree?token=letmein
- `warehouse`: A Postgres container used to represent the data warehouse
    - The host name is the same as the container name (currently set to `warehouse`), which is used when connecting to the Postgres container from other services
    - An initial database called `mydb` is defined and configured in the config file for `warehouse`
    - You can use the credentials in the config file to connect to the Postgres container from other services like `dbeaver` and `viz`
    - You can manage the warehouse using `psql` also by using the following command: `docker exec -it warehouse psql -d mydb -U admin`
    - To enter the container itself, you can use `docker exec -it warehouse bash`
    - By default, the data is stored in a docker volume also called `warehouse` on your local machine. If you wanted to change the location of where the data is stored, you can go to the `docker-compose.yml` file under the `warehouse` service and change the left side of the `volume` mapping to point to your desired location (e.g. `./mylocalpgdata:/var/lib/postgresql/data`)
    ```
    Database Config
    Host: warehouse
    Port: 5432
    DB Name: mydb
    Username: admin
    Password: password
    ```
- `dbeaver`: A DBeaver container used as a UI to connect to and manage the data warehouse
    - You can access the DBeaver UI in your browser at http://localhost:8787
    - Use the configs defined in `warehouse/config` to connect to the data warehouse
- `viz`: A Metabase container that can be used to connect to the data warehouse and create dashboards/visualizations using the data stored there
    - You can access the Metabase UI in your browser at http://localhost:3000
- `datalake`: A MinIO container that can be used as a datalake. It is an object-store similar to AWS S3, and uses the same code as S3 to read/write data
    - You can access the MinIO UI in your browser at http://localhost:9991
    ```
    Login credentials:

    Username: admin
    Password: SuperSecurePassword!
    ```

## File Structure

The files in this repo are used to define container images and necessary environment variables to enable these services to run. Python source code that you want to run on a given container needs to be put in the `src` folder of either the `ingest` or `plotly` worker. The folders each represent a service defined in the `docker-compose.yml` file. The only service that does not have a folder is DBeaver since we don't define any necessary environment variables for it.

*root:*
- `docker-compose.yml`: Orchestrates container creation, sets ports and adds volumes for data storage.
- `ingest`: Container definition for the Python Ingestion Worker/Process
    -  `src`: All Python source code that you want to run on the container should be placed here.
    - `config`: Configuration variables for data connections (`datalake` and `warehouse`)
    - `Dockerfile`: The Dockerfile used to define the `ingest` image & container. You can modify this to adjust the environment as you see fit
    - `requirements.txt`: A requirements file to install dependencies for your projects from `pip`. Add/remove libraries from this file as you see fit
- `plotly`: Container definition for the Plotly-Dash Worker/Process
    - `src`: All Python source code that you want to run on the container should be placed here.
    - `config`: Configuration variables for data connections (`datalake` and `warehouse`)
    - `Dockerfile`: The Dockerfile used to define the `ingest` image & container. You can modify this to adjust the environment as you see fit
    - `requirements.txt`: A requirements file to install dependencies for your projects from `pip`. Add/remove libraries from this file as you see fit
- `datalake`: Holds config files for MinIO container | Represents the data lake (e.g. S3)
    - `config`: Configuration file with environment variables for the container to enable access to UI.
- `warehouse`: Holds config files for Postgres container | Represents the data warehouse (e.g. Redshift)
    - `config`: Configuration file with environment variables that define an initial database.
- `viz`: Holds config files for Metabase container | Represents a BI tool to use for visualizations that can connect directly to your warehouse
    - `config`: Configuration file with environment variables to define the storage location for visualization data