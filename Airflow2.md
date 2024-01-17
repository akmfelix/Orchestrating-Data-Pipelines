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

## 7.3 XCOM
xcom: 'Cross Communication', allows to exchange SMALL amount of data.\
\
The X Come contains the information you want to share between your tasks and it is stored into the meta database of airflow.\
\
### 7.3.1 Share data between tasks
1. by a return value of python function
2. xcom_method

### 1 method
~~~
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

from datetime import datetime

def _t1():
    return 55

def _t2():
    None

with DAG(
    dag_id='xcom_dag',
    start_date=datetime(2024,1,1),
    schedule_interval='@daily',
    catchup=False
) as dag:
    
    t1 = PythonOperator(
        task_id='t1',
        python_callable=_t1
    )

    t2 = PythonOperator(
        task_id='t2',
        python_callable=_t2
    )

    t3 = BashOperator(
        task_id='t3',
        bash_command="echo ''"
    )

    t1 >> t2 >> t3
~~~

### 2 method
* use parameter **ti** - t is the task instance object of your task allowing you to access the method xcom_push.
* **ti.xcom_push** - to push the value that you want to store in the metadatabase that you want to share.
* next, specify key: **key='my_key'** and the value **value=42**
* the good thing with xcom_push - you can define the unique key
* to pull back to your code the xcom from the task t1 into t2 - **ti.xcom_pull(key='my_key', task_ids='t1')**
* By doing this you are able to push xcom from t1 one and get it back from t2.
~~~
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

from datetime import datetime

def _t1(ti):
    ti.xcom_push(key='my_key', value=55)

def _t2(ti):
    print(ti.xcom_pull(key='my_key', task_ids='t1'))

with DAG(
    dag_id='xcom_dag',
    start_date=datetime(2024,1,1),
    schedule_interval='@daily',
    catchup=False
) as dag:
    
    t1 = PythonOperator(
        task_id='t1',
        python_callable=_t1
    )

    t2 = PythonOperator(
        task_id='t2',
        python_callable=_t2
    )

    t3 = BashOperator(
        task_id='t3',
        bash_command="echo ''"
    )

    t1 >> t2 >> t3
~~~

## 7.4 Branch Operators
A very common use case in airflow is to choose one task or another according to a condition. And for that you have the branch operators.\
\
In this task, you have a condition and based on the output of that condition, you return a task ID or an order that corresponds to the next task to execute.\
\
So for example, you may have a condition that returns true, and if it is the case, then you want to execute task A. And if that condition returns false, then you want to execute task C.
![alt branch_operator](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/branch_operator.jpg)\
\
The branch operator has a condition based on the output of that condition. You return one task ID or another.
* **from airflow.operators.python import BranchPythonOperator** - allows you to execute a python function and in that python function you returns the task ID of the next task you want to execute based on your condition.
* Pushed by t1 to the variable value, we can make the condition. So if value is equal to 55, we want to execute the task t2, so we return the task id t2 and if not, then we want to execute the task t3.
* **if value==55:**
* put tasks accordingly **t1 >> branch >> [t2,t3]**
~~~
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.bash import BashOperator

from datetime import datetime

def _t1(ti):
    ti.xcom_push(key='my_key2', value=55)

def _t2(ti):
    ti.xcom_pull(key='my_key2', task_ids='t1')    

def _branch(ti):
    value = ti.xcom_pull(key='my_key2', task_ids='t1')
    if (value==55):
        return 't2'
    return 't3'

with DAG(
    dag_id='xcom2',
    start_date=datetime(2024,1,1),
    schedule_interval='@daily',
    catchup=False
) as DAG:
    
    t1=PythonOperator(
        task_id='t1',
        python_callable=_t1
    )

    branch = BranchPythonOperator(
        task_id='branch',
        python_callable=_branch
    )

    t2=PythonOperator(
        task_id='t2',
        python_callable=_t2
    )

    t3=BashOperator(
        task_id='t3',
        bash_command="echo ''"
    )

    t1 >> branch >> [t2,t3]
~~~
## 7.5 After BranchPythonOperator execution
To avoid skipping t4:
![alt after_branch_exe](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/after_branch_exe.jpg)\


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
