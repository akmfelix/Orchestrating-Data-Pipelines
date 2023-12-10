# What is Airflow
* Apache Airflow is an open source platform to programmatically author, schedule and monitor workflows. The Workflows are you Data Pipelines.
* Airflow is an orchestrator, not a processing framework. Process your gigabytes of data outside of Airflow (i.e. You have a Spark cluster, you use an operator to execute a Spark job, and the data is processed in Spark).
* A DAG is a data pipeline, an Operator is a task.
* An Executor defines how your tasks are executed, whereas a worker is a process executing your task.
* The Scheduler schedules your tasks, the web server serves the UI, and the database stores the metadata of Airflow.

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














