# 7 Implementing Advanced Concepts in Airflow
## 7.1 Group multiple Tasks in to one Task
\
![alt group_dag](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/group_dag.jpg)\
\
To group multiple tasks in to one we use SubDag.

Before
~~~
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.subdag import SubDagOperator

from datetime import datetime

with DAG(
    dag_id='group_dag',
    start_date=datetime(2024,1,1),
    schedule='daily',
    catchup=False
) as dag:
    
    download_a=BashOperator(
        task_id='download_a',
        bash_command='sleep 10'
    )

    download_b=BashOperator(
        task_id='download_b',
        bash_command='sleep 10'
    )

    download_c=BashOperator(
        task_id='download_c',
        bash_command='sleep 10'
    )

    check_files=BashOperator(
        task_id='check_files',
        bash_command='sleep 10'
    )

    transform_a=BashOperator(
        task_id='transform a'
        bash_command='sleep 10'
    )

    transform_b=BashOperator(
        task_id='transform b'
        bash_command='sleep 10'
    )

    transform_c=BashOperator(
        task_id='transform c'
        bash_command='sleep 10'
    )

    [download_a, download_b, download_c] >> check_files >> [transform_a, transform_b, transform_c]
~~~
\ After
~~~
## group_dag.py
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.subdag import SubDagOperator
from sub_dags.subdag_downloads import subdag_downloads

from datetime import datetime

with DAG(
    dag_id='group_dag',
    start_date=datetime(2024,1,1),
    schedule_interval='@daily',
    catchup=False
) as dag:
    
    args = {'start_date':dag.start_date, 'schedule_interval':dag.schedule_interval, 'catchup':dag.catchup}

    downloads=SubDagOperator(
        task_id='downloads',
        subdag=subdag_downloads(dag.dag_id, 'downloads', args)
    )

    check_files=BashOperator(
        task_id='check_files',
        bash_command='sleep 10'
    )

    transform_a=BashOperator(
        task_id='transform_a',
        bash_command='sleep 10'
    )

    transform_b=BashOperator(
        task_id='transform_b',
        bash_command='sleep 10'
    )

    transform_c=BashOperator(
        task_id='transform_c',
        bash_command='sleep 10'
    )

    downloads >> check_files >> [transform_a, transform_b, transform_c]
~~~
\
~~~
## subdag_downloads.py
from airflow import DAG
from airflow.operators.bash import BashOperator

def subdag_downloads(parent_dag_id, child_dag_id, args):

    with DAG(
        f"{parent_dag_id}.{child_dag_id}",
        start_date=args['start_date'],
        schedule_interval=args['schedule_interval'],
        catchup=args['catchup']
    ) as dag:
        download_a=BashOperator(
            task_id='download_a',
            bash_command='sleep 10'
        )

        download_b=BashOperator(
            task_id='download_b',
            bash_command='sleep 10'
        )

        download_c=BashOperator(
            task_id='download_c',
            bash_command='sleep 10'
        )
    return dag
~~~

## 7.2 TaskGroup
New feature of grouping tasks
~~~
# task_group_task.py
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime
from groups.group_downloads import download_tasks
from groups.group_transforms import transform_tasks

with DAG(
    dag_id='tasks_group_dag',
    start_date=datetime(2024,1,1),
    schedule_interval='@daily',
    catchup=False
) as dag:
    args = {'start_date':dag.start_date, 'schedule_interval':dag.schedule_interval, 'catchup':dag.catchup}

    downloads=download_tasks()

    check_files=BashOperator(
        task_id='check_files',
        bash_command='sleep 10'
    )

    transforms=transform_tasks()

    downloads>> check_files>> transforms
~~~
\
~~~
# group_downloads.py
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.utils.task_group import TaskGroup

def download_tasks():

    with TaskGroup(group_id='downloads', tooltip='Download tasks') as group:

        download_a=BashOperator(
            task_id='download_a',
            bash_command='sleep 10'
        )

        download_b=BashOperator(
            task_id='download_b',
            bash_command='sleep 10'
        )

        download_c=BashOperator(
            task_id='download_c',
            bash_command='sleep 10'
        )
    return group

~~~
\
~~~
# group_transforms
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.utils.task_group import TaskGroup

from datetime import datetime

def transform_tasks():

    with TaskGroup(group_id='transforms', tooltip='Transform tasks') as group:

        transform_a=BashOperator(
            task_id='transform_a',
            bash_command='sleep 10'
        )

        transform_b=BashOperator(
            task_id='transform_b',
            bash_command='sleep 10'
        )

        transform_c=BashOperator(
            task_id='transform_c',
            bash_command='sleep 10'
        )

    return group
~~~
================================================================
================================================================
# Airflow installation
## What-is-docker
Docker is a software development platform and a kind of virtualization technology that makes it easy for us to develop and deploy apps inside packaged, virtual containerized environments, meaning apps run the same no matter where they are on what machine they are running on.\
\
Your app runs in the Docker container, and you can think of a Docker container as a little microcomputer with its own job isolated CPU processes, memory and network resources. Because of these, they can be easily added, removed, stopped or started again without affecting each other or the host machine.\
\
If you open the file Docker compose the HTML, that file describes the airflow instance and or services Docker containers it needs to run.
~~~
# in cmd promd
docker-compose ps
~~~

### Prerequisites
First, make sure you have installed Docker Desktop and Visual Studio. If not, take a look at these links:
* Get Docker
* Get Visual Studio Code

Docker needs privilege rights to work, make sure you have them.\

### Install Apache Airflow with Docker
* Create a folder 'materials' in your Documents
* In this folder, download the following file: docker compose file
* If you right-click on the file and save it, you will end up with docker-compose.yaml.txt. Remove the .txt and keep docker-compose.yaml
* Open your terminal or CMD and go into Documents/materials
* Open Visual Studio Code by typing the command: "code ."
* Right click below docker-compose.yml and create a new file .env (don't forget the dot before env)
* In this file add the following lines and save the file:
~~~
AIRFLOW_IMAGE_NAME=apache/airflow:2.4.2
AIRFLOW_UID=50000
~~~
* Go at the top bar of Visual Studio Code -> Terminal -> New Terminal
* In your new terminal at the bottom of Visual Studio Code, type the command "docker-compose up -d" and hit ENTER
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

# To run individual task from terminal window
~~~
# in terminal window
docker-compose ps
# in order to run one-1 task from your DAG
# type scheduler name of container
# now you are inside of docker container 
docker exec -it docker-airflow-airflow-scheduler-1 /bin/bash
airflow -h
# dag_id, task_id
airflow tasks test user_processing create_table 2023-12-01
# to exit airflow container hit ctrl+d
~~~

### Airflow UI
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

## To see uploaded data into table in docker
~~~
docker-compose ps
# Type name of the worker container
docker exec -it docker-airflow-worker_1 /bin/bash
# in docker container type (you got processed_user.csv)
ls /tmp/

# container name of database
docker exec -it docker-airflow_postgres_1 /bin/bash/
# in a container type
psql -Uairflow
# then
select * from users;
~~~
