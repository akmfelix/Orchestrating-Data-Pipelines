# What is Airflow
* Apache Airflow is an open source platform to programmatically author, schedule and monitor workflows. The Workflows are you Data Pipelines.
* Airflow is an orchestrator, not a processing framework. Process your gigabytes of data outside of Airflow (i.e. You have a Spark cluster, you use an operator to execute a Spark job, and the data is processed in Spark).
* A DAG is a data pipeline, an Operator is a task.
* An Executor defines how your tasks are executed, whereas a worker is a process executing your task.
* The Scheduler schedules your tasks, the web server serves the UI, and the database stores the metadata of Airflow.

## Why-do-we-need-airflow
Let's imagine that you have the following data pipeline with three tasks extract, load and transform, and it runs every day at 10:00 PM. Very simple. Obviously at every step you are going to interact with an external tool or an external system, for example, an API for extract. Snowflake for load, and DBT for transform. Now, what if the API is not available anymore? Or what if you have a error in Snowflake or what if you made a mistake in your transformations with DBT. As you can see at every step, you can end up with a failure and you need to have a tool that manages this. Also, what if instead of having one data pipeline, you have hundreds of data pipelines. As you can imagine, it's gonna be a nightmare for you, and this is why you need Airflow. With Airflow you are able to manage failures automatically, even if you have hundreds of data pipelines and millions of tasks.
![alt why-do-we-need-airflow](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/why-do-we-need-airflow.jpg)


# Benefits
* Everything is coded in Python and everrything is dynamic. 
* Scalabale. You can execute as many tasks as you want with Airflow.
* User Interface. Which is nice to have to monitor tasks and data pipelines.
* Extensibility. You can add own plugins, own functionalaties to Airflow.

# Core Components
* Web Server. The web server is a flask Python web server, that allows to access to user interface.
* Scheduler. Schedulling tasks and data pipelines.
* Metadatabase or Metastore. The metadatabase is nothing more than a database that is compatibale with SQL alchemy. For example Postgres, MySQL, Oracle, Sql server. In this database, you will have metadata related to your data, data pipelines, airflow users.
* Triggerer. Allows to run specific kind of tasks.

## Executor
* Queue. In a queue your tasks will be pushed in it in order to execute them in the right order
* Worker. The worker is where your tasks are effectively executred.

#  Core Concepts
* DAG. A directed acyclic graph. A graph that represents a data pipeline with tasks and directed dependicies.
* Operators: 1) Action - Execute an action 2) Transfer - Transfer data 3) Sensor - Wait for a condition to be met.
* Task / Task Instance. When a DAG runs, the scheduler creates a DAG Run for that specific run.

# Sensors
A sense of wait for something to happen before moving to the next task.
* poke_interval. which is defined to 60 seconds by default. So every 60 seconds sensors checks if the condition is true or not before executing the next task.
* timeout. which is defined to 7 days by default. It tells in seconds when your sensor times out and fails.

In this example, we want to verify if the API is available or not. And for that we use the HTTP sensor.\
1. As usual, you need to create a new variable. In this case, is_api_available and you add the HTTP sensor in order to check if URL active or not.\
2. Then you specify task_id, always specify the task_id as 'is_api_available' as well.\
3. The HTTP connection ID as you interact with an external service, in this case a URL, you need to define a connection as well as the website that you want to check.\
4. Then last but not least, you have the endpoint API slash. So that's the path from the website that you want to check.\
5. Finally add import airflow providers HTTP sensors.
~~~
    is_api_available = HttpSensor (
        task_id = 'is_api_available',
        http_conn_id = 'user_api',
        endpoint = 'api/'
    )
~~~


### Airflow not a data streaming solution neither a processeing framework.

# Airflow installation
### Prerequisites
First, make sure you have installed Docker Desktop and Visual Studio. If not, take a look at these links:
* Get Docker
* Get Visual Studio Code

Docker needs privilege rights to work, make sure you have them.\

### Install Apache Airflow with Docker
* Create a folder materials in your Documents
* In this folder, download the following file: docker compose file
* If you right-click on the file and save it, you will end up with docker-compose.yaml.txt. Remove the .txt and keep docker-compose.yaml
* Open your terminal or CMD and go into Documents/materials
* Open Visual Studio Code by typing the command: "code ."
* Right click below docker-compose.yml and create a new file .env (don't forget the dot before env)
* In this file add the following lines:
~~~
AIRFLOW_IMAGE_NAME=apache/airflow:2.4.2
AIRFLOW_UID=50000
~~~
and save the file
* Go at the top bar of Visual Studio Code -> Terminal -> New Terminal
* n your new terminal at the bottom of Visual Studio Code, type the command "docker-compose up -d" and hit ENTER
* You will see many lines scrolled, wait until it's done. Docker is downloading Airflow to run it. It can take up to 5 mins depending on your connection. If Docker raises an error saying it can't download the docker image, ensure you are not behind a proxy/vpn or corporate network. You may need to use your personal connection to make it work.
* Open your web browser and go to "localhost:8080"

### Troubleshoots
-> If you don't see this page, make sure you have nothing already running on the port 8080\
\
Also, go back to your terminal on Visual Studio Code and check your application with docker-compose ps\
\
All of your "containers" should be healthy.\
\
If a container is not healthy. You can check the logs with 
~~~
docker logs materials_name_of_the_container
~~~
Try to spot the error; once you fix it, restart Airflow with
~~~
docker-compose down
# then 
docker-compose up -d
~~~
and wait until your container states move from starting to healthy.
-> If you see this error
remove your volumes with 
~~~
docker volume prune 
# and run 
docker-compose up -d 
# again
~~~
-> If you see that airflow-init docker container has exited, that's normal :)

# What is a docker
Software development platform and a kind of virtualization technology that makes easy to develop and deploy apps inside packaged virtual containerized environments.

## Docker compose file, docker containerz

What is the best view to check the dependencies of your DAG?
* Graph view
What is the best view to monitoring the time it takes for your tasks to complete over many DAG Runs?
* Landing Times
What's the most useful view to detect bottlenecks in your DAG?
* Gantt
What view can you use to check if a modification you made is applied on your DAG or not?
* Code
What view is best to get the history of the states of your DAG Runs and Tasks?
* Grid

# Task
create_table -> is_api_available -> extract_user -> process_user -> store_user

~~~
from airflow import DAG
from datetime import datetime
from airflow.providers.postgres.operators.postgres import PostgresOperator

with DAG(
    'user_processing', start_date = datetime(2023,12,1), schedule_interval='@daily', catchup=False
) as dag:
    create_table = PostgresOperator (
        task_id = 'create_table',
        postgres_conn_id = 'postgres',
        sql = '''
            CREATE TABLE IF NOT EXISTS users (
                firstname TEXT NOT NULL,
                lastname TEXT NOT NULL,
                country TEXT NOT NULL,
                username TEXT NOT NULL,
                password TEXT NOT NULL,
                email TEXT NOT NULL
            );
        '''
    )

~~~
~~~
# in terminal window
docker-compose ps
# in order to run one-1 task from your DAG
docker exec -it docker-airflow-airflow-scheduler-1 /bin/bash
airflow -h
airflow tasks test user_processing create_table 2023-01-01
~~~











