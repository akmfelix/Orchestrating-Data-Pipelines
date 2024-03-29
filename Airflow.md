https://github.com/teamclairvoyant/airflow-maintenance-dags

# What is Airflow
* Apache Airflow is an open source platform to programmatically author, schedule and monitor workflows. The Workflows are you Data Pipelines.
* Airflow is an orchestrator. It allows you to execute your tasks in the right way, in the right order at the right time.
* An Executor defines how your tasks are executed, whereas a worker is a process executing your task.
* The Scheduler schedules your tasks, the web server serves the UI, and the database stores the metadata of Airflow.

## Core Components of Airflow
* Web Server. The web server is a flask Python web server, that allows to access to user interface.
* Scheduler. Schedulling tasks and data pipelines.
* Metadatabase or Metastore. The metadatabase is nothing more than a database that is compatibale with SQL alchemy. For example Postgres, MySQL, Oracle, Sql server. In this database, you will have metadata related to your data, data pipelines, airflow users.
* Triggerer. Allows to run specific kind of tasks.

## What-is-not-Airflow
* Airflow is not a data streaming solution, neither a data processing framework.
* Airflow is an orchestrator, not a processing framework. Process your gigabytes of data outside of Airflow (i.e. You have a Spark cluster, you use an operator to execute a Spark job, and the data is processed in Spark).
* Instead, you should use Airflow as a way to trigger the tool that will process your data. For example, you have the SparkSubmitJobOperator, you could use that operator to process terabytes of data.

## Benefits
* Everything is coded in Python and everything is dynamic. 
* Scalabale. You can execute as many tasks as you want with Airflow.
* User Interface. Where you can monitor tasks and data pipelines.
* Extensibility. You can add own plugins, own functionalaties to Airflow.

## Why-do-we-need-airflow
Let's imagine that you have the following data pipeline with three tasks extract, load and transform, and it runs every day at 10:00 PM. Very simple. Obviously at every step you are going to interact with an external tool or an external system, for example, an API for extract. Snowflake for load, and DBT for transform. Now, what if the API is not available anymore? Or what if you have a error in Snowflake or what if you made a mistake in your transformations with DBT. As you can see at every step, you can end up with a failure and you need to have a tool that manages this. Also, what if instead of having one data pipeline, you have hundreds of data pipelines. As you can imagine, it's gonna be a nightmare for you, and this is why you need Airflow. With Airflow you are able to manage failures automatically, even if you have hundreds of data pipelines and millions of tasks.
![alt why-do-we-need-airflow](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/why-do-we-need-airflow.jpg)

#  1 Core Concepts. DAG. Operators. Task.
## 1.1 DAG. First DAG
A DAG means directed acyclic graph, and it's nothing more than a graph with nodes (tasks), directed edges (dependency) and no cycles.
 1. Create python file in a DAG folder.
 2. from airflow import DAG - this is how airflow knows, that it's a DAG file.
 3. Once you have this import, you are ready to instantiate a DAG object and the first parameter to define is the DAG ID, the unique identifier or the name of your DAG. This ID must be unique across all tags in your airflow instances.
 4. Then you need to define the start date. And the start date defines the date at which your DAG starts being scheduled.
 5. Once you have the start date, you need to define the schedule_interval, the frequency at which your DAG is triggered once every day, every 15 minutes, once a week and so on. Keep in mind that the schedule_interval is defined as a chronic expression. So behind the '@daily' you have a chron expression.
 6. Set 'catchup=False'. Because by default the catch parameter is set to true. And it means that if between now and the start date your DAG hasn't been triggered, then as soon as you start scheduling your data pipeline from the airflow, you are going to catchup.
~~~
from airflow import DAG
~~~
 
## 1.2 Operators
* Operators. Think of operator as a task. There are 3 types of operator: 1) Action - Execute an action. For example, the PythonOperator executes a Python function, the BashOperator executes a bash command. 2) Transfer - Transfer data. Basically Transfer operator allows you to transfer data from a point A to point B. 3) Sensor - Wait for a condition to be met. Sensors are very useful because they allow you to wait for something to happen. For example, you are waiting for files you will use the FileSensor.
* A DAG is a data pipeline, an Operator is a task.
* Task / Task Instance. An Operator is a task and when you run an operator, you get a task instance.
* Finally, if you group all of those concepts together, you end up with the concept of workflow where you have your Operators, directed dependencies, and so a DAG, and that gives you the concept of workflow.
* Let's assume a DAG start_date to the 28/10/2021:10:00:00 PM UTC and the DAG is turned on at 10:30:00 PM UTC with a schedule_interval of */10 * * * * (After every 10 minutes). How many DagRuns are going to be executed? 2

## 1.3 Executor
Executor doesn't execute a task, but it defines how, and on which system your tasks are executed. In addition, you have a concept called executor and an executor defines how and on which support your tasks are executed. For example, if you have a Kubernetes cluster, you want to execute your tasks on this Kubernetes cluster, you will use the KubernetesExecutor. If you want to execute your tasks in a Celery cluster, Celery is a Python framework to execute multiple tasks on multiple machines, you will use the CeleryExecutor. Keep in mind that the executor doesn't execute any tasks. Now, if you use the CeleryExecutor for example, you will have two additional core components, a **queue**, and a **worker**.
* Queue. In a queue your tasks will be pushed in it in order to execute them in the right order.
* Worker. The worker is where your tasks are effectively executred.

# 2 Airflow Architecture
## 2.1 Single-Node-Architecture
![alt single-node-architecture](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/single-node-architecture.jpg)

## 2.2 Multi-Nodes-Architecture (Celery/ Kubernetes)
To run Airflow in production, you are not going to stay with a single node architecture. Indeed, you want to make sure that you don't have a single point of failure. You want to make sure that your architecture is highly available and you want to make sure that you're able to deal with the workload, with the number of tasks that you want to execute, and for that you need to use the multi nodes architecture. In this example, we use Celery, but that works with Kubernetes as well.\
\
Also keep in mind that you should have at least two Schedulers as well as two Web servers, maybe a Load balancer in front of your web servers to deal with the number of requests on the Airflow UI, as well as PGBouncer to deal with the number of connections that will be made to your meta database.
![alt multi-nodes-architecture](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/multi-nodes-architecture.jpg)

# 3 Airflow explanation

## 3.1 Execution-Flow
So you have one node with the components, the Web server, the Meta database, the Scheduler, the Executor, and the folder dags.\
\
First you create a new DAG, dag.py and you put that file into the folder DAGs. Next, the Scheduler parses this folder dags every five minutes by default to detect new DAGs. So you may need to wait up to five minutes before getting your DAG on the Airflow UI. Next, whenever you apply a modification to that DAG you may need to wait up to 30 seconds before getting your modification.\
\
Next, the Scheduler runs the DAG, and for that, it creates a DAG Run object with the state Running. Then it takes the first task to execute and that task becomes a task instance object. The task instance object has the state None and then Scheduled. After that the Scheduler sends the task instance object into the Queue of the Executor. Now the state of the task is Queued and the Executor creates a sub process to run the task, and now the task instance object has the state Running. Once the task is done, the state of the task is Success or Failed. It depends. And the Scheduler checks, if there is no tasks to execute.\
\
If the DAG is done in that case, the DAG Run has the state Success. And basically you can update the Airflow UI to check the states of both the DAG Run and the task instances of that DAG Run.

## 3.2 What should you keep in mind after what you've learned?
* Airflow is an orchestrator, not a processing framework. Process your gigabytes of data outside of Airflow (i.e. You have a Spark cluster, you use an operator to execute a Spark job, and the data is processed in Spark).
* A DAG is a data pipeline, an Operator is a task.
* An Executor defines how your tasks are executed, whereas a worker is a process executing your task
* The Scheduler schedules your tasks, the web server serves the UI, and the database stores the metadata of Airflow.

## 3.3 Questions
* What is a DAG: a collection of all the tasks you want to run, organised in a way that reflects their relationships and dependencies with no cycles.
* What is the meaning of the schedule_interval property for a DAG: It defines how often a DAG should be run from the start_date+schedule_time.
* What is an operator: an operator describes a single task in a workflow.
* What does a Sensor: it is a long running task waiting for an event to happen. A poke function is called every n seconds to check if the criteria are met.

![alt what-is-dag](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/what-is-dag.jpg)
![alt what-is-workflow](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/what-is-workflow.jpg)

## 3.4 Sensors
A sense of wait for something to happen before moving to the next task.
* poke_interval. which is defined to 60 seconds by default. So every 60 seconds sensors checks if the condition is true or not before executing the next task.
* timeout. which is defined to 7 days by default. It tells in seconds when your sensor times out and fails.
 
### 3.4 Sensors example
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

## 3.5 Hook
With a airflow, you can interact with many tools and to make sure that it is easy for you to interact with the tool, there is the concept of Hook.\
\
Let's imagine that you have the operator and you want to execute a SQL request to a PostgreSQL database. With the PostgresOperator, you can execute a SQL request, but behind the scene a postgres HOOK is used and the goal of the PostgresHook is to obstruct all the complexity of interacting with a Postgres database.\
\
So keep in mind, whenever you interact with an external tool or an external service, you have a Hook behind the scene that abstracts the complexity of interacting with that tool or service. You have the AWS hook, you have the Postgres hook, you have the MySQL hook and the list goes on.\
\
I strongly advise you to always take a look at the hook as you may have access to some methods that you don't have access to from the operator.\
\
A hook allows you to easily interact with an external tool or an external service.\
Keep in mind the method copy_expert doesn't exist from the PostgreOperator. You need to use the PostgresHook for that and that's why I always recommend you to take a look at the hook in order to see what methods you can use.

## 3.6 DAG scheduling
* start_date: defines the date at which your DAG starts being scheduled.
* schedule_interval: how often a DAG runs.
* end_date: the timestamp from which a DAG ends.
* a DAG is triggered AFTER tje start_date/last_run + the schedule_interval
* Your DAG is effectively triggered after the start date or the last time your dag was triggered, plus the schedule_interval.
![alt dag_schedule](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/dag_schedule.jpg)

## 3.7 Catchup/ Backfill mechanisms
* The catchup mechanism allows you to automatically run non trigger dag runs between the last time your DAG was triggered and the date of now.
* The backfilling mechanism allows to run historical DAG runs.

# 4 Task
create_table -> is_api_available -> extract_user -> process_user -> store_user
* Define a Data Pipeline
* Execute a SQL request with the PostgresOperator
* Execute a Python function with the PythonOperator
* Execute an HTTP request against an API
* Wait for something to happen with Sensors
* Use Hooks to access secret methods
* Exchange data between tasks
## 4.1 Instantiate DAG object
with DAG
~~~
from airflow import DAG
from datetime import datetime

with DAG(
    dag_id='user_processing',
    start_date=datetime(2023, 12, 20),
    schedule_interval='@daily',
    catchup=False
) as dag:
    None
~~~

## 4.2 Create Table. SQL
We are going to use the postgres operator in order to execute a SQL request against a database and create a table.\
Whenever you want to use an operator, you need to import the corresponding operator and for the Postgres operator it is **from airflow.providers.postgres.operators.postgres import PostgresOperator**
~~~
from airflow.providers.postgres.operators.postgres import PostgresOperator
...
    create_table=PostgresOperator(
        task_id='create_table',
        postgres_conn_id='postgres',
        sql='''
            CREATE TABLE IF NOT EXISTS user (
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

## 4.3 Sensors. HttpSensor
In this case, we want to verify if the API is available or not and for that we use the HTTP sensor.
~~~
from airflow.providers.http.sensors.http import HttpSensor
...
    is_api_available=HttpSensor(
        task_id='is_api_available',
        http_conn_id='user_api',
        endpoint='api/'
    )
~~~

## 4.4 Extract data from API. SimpleHttpOperator
It's time to extract the data from API and for this we are going to use the SimpleHttpOperator.\
method='GET' to request data.\
response_filter=lambda response: json.loads(response.text) to extract data and transform it in json format. And for this we can define lambda, python function in a way that loads the response, the text into a json.\
log_response=True - to log the response so that you will be able to see the response that you get in the logs on the user interface.
~~~
from airflow.providers.http.operators.http import SimpleHttpOperator
import json
...
    extract_user=SimpleHttpOperator(
        task_id='extract_user',
        http_conn_id='user_api',
        endpoint='api/',
        method='GET',
        response_filter = lambda response: json.loads(response.text)
    )
~~~

## 4.5 Process user. PythonOperator
PythonOperator allows to use python operators
~~~
from airflow.operators.python import PythonOperator
from pandas import json_normalize
...
def _process_user(ti):
    user = ti.xcom_pull(task_ids='extract_user')
    user = user['results'][0]
    processed_user = json_normalize({
        'firstname':user['name']['first'],
        'lastname':user['name']['last'],
        'country':user['location']['country'],
        'username':user['login']['username'],
        'password':user['login']['password'],
        'email':user['email'] }
    )
    processed_user.to_csv('myfolder/processed_user.csv', index=None, header=False)
...
    process_user=PythonOperator(
        task_id='process_user',
        python_callable=_process_user
    )
~~~

## 4.6 Store procced user. PostgresHook
A hook allows you to easily interact with an external tool or an external service.
~~~
from airflow.providers.postgres.hooks.postgres import PostgresHook
...
def _store_user():
    hook = PostgresHook(postgres_conn_id='postgres')
    hook.copy_expert(
        sql="COPY users FROM stdin WITH DELIMITER as ','",
        filename='/tmp/processed_user.csv'
    )
...
    store_user = PythonOperator(
        task_id='store_user',
        python_callable=_store_user
    )
~~~

## 4.7 The whole code
~~~
from airflow import DAG
from datetime import datetime

from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.http.sensors.http import HttpSensor
from airflow.providers.http.operators.http import SimpleHttpOperator
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook

import json
from pandas import json_normalize

# The task to run
def _store_user():
    hook = PostgresHook(postgres_conn_id='postgres')
    hook.copy_expert(
        sql="COPY users FROM stdin WITH DELIMITER as ','",
        filename='/tmp/processed_user.csv'
    )

def _process_user(ti):
    user = ti.xcom_pull(task_ids="extract_user")
    user = user['results'][0]
    processed_user = json_normalize({
        'firstname': user['name']['first'],
        'lastname': user['name']['last'],
        'country': user['location']['country'],
        'username': user['login']['username'],
        'password': user['login']['password'],
        'email': user['email'] })
    processed_user.to_csv('/tmp/processed_user.csv', index=None, header=False)  
    
with DAG(
    dag_id="user_processing",
    start_date=datetime(2023, 1, 1),
    schedule_interval="@daily",
    catchup=False
) as dag:
    
    create_table = PostgresOperator(
        task_id='create_table',
        postgres_conn_id='postgres',
        sql='''
            CREATE TABLE IF NOT EXISTS users (
                firstname TEXT NOT NULL,
                lastname TEXT NOT NULL,
                country TEXT NOT NULL,
                username TEXT NOT NULL,
                password TEXT NOT NULL,
                email TEXT NOT NULL
            );
        ''')
        
    is_api_available = HttpSensor(
        task_id='is_api_available',
        http_conn_id='user_api',
        endpoint='api/'
    )
    
    extract_user = SimpleHttpOperator(
        task_id='extract_user',
        http_conn_id='user_api',
        endpoint='api/',
        method='GET',
        response_filter=lambda response: json.loads(response.text),
        log_response=True
    )
    
    process_user = PythonOperator(
        task_id='process_user',
        python_callable=_process_user
    )
    
    store_user = PythonOperator(
        task_id='store_user',
        python_callable=_store_user
    )
    
    extract_user >> process_user
~~~

# 5 New AIRFLOW feature of scheduling
## 5.1 New way of Scheduling
Before Airflow ver 2 'TriggerDag operator or the external tasks sensor, the TriggerDag one operator allows you to trigger knows or DAG from a DAG, whereas the external task sensor allows you to wait for a task in another DAG before moving forward. That being said, those two operators are pretty complex to use.'\
After Airflow ver 2. With a new way of scheduling your DAGs, it's much easier to wait for data to exist in order to trigger another DAG.\
When you make an update to a dataset to some data from a DAG that triggers another DAG. 
* You can trigger your DAGs based on data updates.
![alt ex1](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/ex1.jpg)
~~~
# Schedule before 2.4
with DAG(schedule_interval='@daily')
OR
With DAG(timetable=MyTimeTable)

# since 2.4
with DAG(schedule=mydataset)
# in a schedule= you can put either a timetable, a cron expression, a time delta object or a data set.
~~~

## 5.2 Dataset
Dataset has two properties and the first one is the URI is like the path to the data, and the extra parameter is a decent dictionary that you can define.
![alt dataset](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/dataset.jpg)
~~~
from airflow import Dataset
my_file = Dataset(
    's3://dataset/file.csv',
    extra={'owner': 'james'}
    )
~~~

## 5.3 @task decorators. Dataset update
As we want to add task to data pipeline, we can use the task decorator type **from airflow.decorators import task**, the task decorator allows you to create python operator tasks in a much faster way.\
To indicate airflow that this task update_dataset updates the dataset my_file.txt **@task(outlets=[my_file])**. It means that as soon as this task succeeds, well, the DAG that depends on this dataset my_file.txt will be automatically triggered.
~~~
# producer.py
from airflow import DAG, Dataset
from airflow.decorators import task

from datetime import datetime

my_file=Dataset('tmp/my_file.txt')

with DAG(
    dag_id='producer',
    start_date=datetime(2023,12,1),
    schedule='@daily',
    catchup=False
):
    @task(outlets=[my_file])
    def update_dataset():
        with open('my_fil.uri', 'a+') as f:
            f.write('producer updated')

    update_dataset()
~~~

## 5.4 Triggered DAG
**schedule=[my_file]** it means that as soon as the task in producer.py DAG update_dataset updates the Dataset my_file.txt that triggers the DAG consumer.py. So now your DAG is not scheduled based on time, but it is based on data updates.
~~~
from airflow import DAG, Dataset
from airflow.decorators import task

from datetime import datetime

my_file=Dataset('/tmp/my_file.txt')

with DAG(
    dag_id='consumer',
    schedule=[my_file],
    start_date=datetime(2023,12,1),
    catchup=False
):
    @task
    def read_dataset():
        with open(my_file.uri, 'r') as f:
            print(f.read())
    
    read_dataset()
~~~

## 5.5 Many datasets
Trigger consumer.py after producer.py
~~~
# producer.py
from airflow import DAG, Dataset
from airflow.decorators import task

from datetime import datetime

my_dataset = Dataset('/tmp/my_file.txt')
my_dataset_2 = Dataset('/tmp/my_file_2.txt')

with DAG(
    dag_id='producer',
    schedule='@daily',
    start_date=datetime(2023,12,1),
    catchup=False
):
    @task(outlets=[my_dataset]) # to indicate airflow that this task updates dataset
    def update_dataset():
        with open(my_dataset.uri, 'a+') as f:
            f.write('producer updated')

    @task(outlets=[my_dataset_2])
    def update_dataset_2():
        with open(my_dataset_2.uri, 'a+') as f:
            f.write('producer updated')

    update_dataset()
    update_dataset_2()

---------------------------------------
# consumer.py
from airflow import DAG, Dataset
from airflow.decorators import task

from datetime import datetime

my_dataset=Dataset('/tmp/my_file.txt')
my_dataset_2=Dataset('/tmp/my_file_2.txt')
with DAG(
    dag_id='consumer',
    schedule=[my_dataset,my_dataset_2],
    start_date=datetime(2023, 12, 1),
    catchup=False
):
    @task
    def read_dataset():
        with open(my_dataset.uri, 'r') as f:
            print(f.read())
    read_dataset()
~~~
![alt file-trigger](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/file_trigger.jpg)

## 5.6 Dataset trigger limitations
Datasets are amazing, but they have limitations as well:
* DAGs can only use Datasets in the same Airflow instance. A DAG cannot wait for a Dataset defined in another Airflow instance.
* Consumer DAGs are triggered every time a task that updates datasets completes successfully. Airflow doesn't check whether the data has been effectively updated.
* You can't combine different schedules like datasets with cron expressions.
* If two tasks update the same dataset, as soon as one is done, that triggers the Consumer DAG immediately without waiting for the second task to complete.
* Airflow monitors datasets only within the context of DAGs and Tasks. If an external tool updates the actual data represented by a Dataset, Airflow has no way of knowing that.

# 6 Databases and Executors

## 6.1 Executor
Executor defines how to run your tasks on which system. And basically you have many different executors that you can use. They are local executors and remote executors.\
\
For example:
* You have the local executor to run multiple tasks on a single machine.
* You have the sequential executor to run one task at a time on a single machine.
* And you have the remote executors like the self executor to execute your tasks on the celery cluster on multiple machines.
* And you have the communities executor to run your tasks on a Kubernetes cluster, same thing on multiple machines in multiple pods.

The only thing that you need to change is the executor parameter in the configuration file of airflow.\
\
To copy the configuration file of airflow from the container to the host to your machine.
~~~
docker cp docker-airflow-airflow-scheduler-1:/opt/airflow/airflow.cfg .
~~~

To restart airflow
~~~
docker-compose down && docker-compose up -d
~~~

We have this environment variable with the CeleryExecutor, this overrides the value of the SequentialExecutor corresponding to the parameter executor.
~~~
in airflow.cfg and docker-compose.yaml
AIRFLOW__CORE__EXECUTOR: CeleryExecutor
executor = SequentialExecutor
~~~

## 6.2 SequentialExecutor
The sequential executor is the executor by default when you install airflow manually. \
\
You have a web server, a scheduler and a database, a SQLite database. And if you want to run the following, DAG, the scheduler runs one task at a time. You are not able to run multiple tasks at the same time.\
\
So for example, here it runs T1 one, then once T2 is completed, it runs to T3. Then once it is completed, it runs to four.\
\
To configure this executor, you just need to modify the executor setting with the sequential executor value.
![alt SequentialExecutor](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/SequentialExecutor.jpg)

## 6.3 The Local executor
The local executor is one step further than the sequential executor, as it allows you to execute multiple tasks at the same time, but on a single machine. \
\
It means that you end up with the same airflow instance, but with a different database. In this time we are going to use either PostgreSQL, my SQL, Oracle DB or whatever you want, but not the SQL database. And by doing so you are able to execute multiple tasks at the same time.\
\
So for example, the scheduler runs T1 and once it is completed, T2 and T3 run at the same time. Once they are completed, it runs to four and you are done to configure this executable. To define:
~~~
executor=LocalExecutor
sql_alchemy_conn = postgresql+psycopg2://<user>:<password>@<host>/<db>
~~~
* What is the main limitation of SQLite? It can accept only one writer at a time.

## 6.4 Celery executor
The Celery executor is nice to start sketching out the number of tasks that you can execute at the same time.\
\
How? By using a Celery cluster in order to execute your tasks on multiple machines.\
\
So first, you still have the web server, the scheduler and the metadata database of airflow with Postgres. But as you can see, you have additional components. The first one is the workers. Indeed, you have airflow workers, which are nothing more than machines in charge of executing your tasks.\
\
So in this case, you have three workers, so three machines to execute your tasks. If you need more resources to execute more tasks, you just need to add a new airflow worker and that's it.\
\
Now the Celery Queue is composed of two things the Result Back End, where the airflow workers store the status of the tasks that have been executed and the Broker, which is nothing more than a queue where the scheduler sends the task to execute and the workers pull the tasks out of that queue to execute them.\
\
![alt celery](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/celery.jpg)

You need to install the Celery queue which may be redis or rabbit in queue.
~~~
executor = CeleryExecutor
sql_alchemy_conn=postgresql+psycopg2://<user>:<password>@<host>/<db>
celery_result_backend=postgresql+psycopg2://<user>:<password>@<host>/<db>
celery_broker_url=redis://:@redis:6379/0

    AIRFLOW__CORE__EXECUTOR: CeleryExecutor
    AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    # For backward compatibility, with Airflow <2.3
    AIRFLOW__CORE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
    AIRFLOW__CELERY__BROKER_URL: redis://:@redis:6379/0

  redis:
    image: redis:latest
    expose:
      - 6379
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 30s
      retries: 50
    restart: always

  airflow-worker:
    <<: *airflow-common
    command: celery worker
    healthcheck:
      test:
        - "CMD-SHELL"
        - 'celery --app airflow.executors.celery_executor.app inspect ping -d "celery@$${HOSTNAME}"'
      interval: 10s
      timeout: 10s
      retries: 5
    environment:
      <<: *airflow-common-env
      # Required to handle warm shutdown of the celery workers properly
      # See https://airflow.apache.org/docs/docker-stack/entrypoint.html#signal-propagation
      DUMB_INIT_SETSID: "0"
    restart: always
    depends_on:
      <<: *airflow-common-depends-on
      airflow-init:
        condition: service_completed_successfully
    
~~~

## 6.5 parallel_dag.py
~~~
    from airflow import DAG
    from airflow.operators.bash import BashOperator
     
    from datetime import datetime
     
    with DAG('parallel_dag', start_date=datetime(2022, 1, 1), 
        schedule_interval='@daily', catchup=False) as dag:
     
        extract_a = BashOperator(
            task_id='extract_a',
            bash_command='sleep 1'
        )
     
        extract_b = BashOperator(
            task_id='extract_b',
            bash_command='sleep 1'
        )
     
        load_a = BashOperator(
            task_id='load_a',
            bash_command='sleep 1'
        )
     
        load_b = BashOperator(
            task_id='load_b',
            bash_command='sleep 1'
        )
     
        transform = BashOperator(
            task_id='transform',
            bash_command='sleep 1'
        )
     
        extract_a >> load_a
        extract_b >> load_b
        [load_a, load_b] >> transform
~~~

## 6.6 Flower
To monitor tasks on dashboard. 
~~~
localhost:5555
docker-compose --profile flower up -d
~~~

## 6.6 Remove DAG examples
* Open the file docker-compose.yaml
* Replace the value 'true' by 'false' for the AIRFLOW__CORE__LOAD_EXAMPLES environment variables
* Restart Airflow by typing **docker-compose down** && **docker-compose up -d**
* Once it's done, go back to localhost:8080

## 6.6 What is a ques
With queues you are able to distribute your tasks among multiple machines according to the specificities of your tasks and your machines.
![alt ques](https://github.com/akmfelix/Orchestrating-Data-Pipelines/blob/main/img/ques.jpg)

## 6.7 Creating a new queue
Creating a queue in airflow is very simple. You just need to specify to the command cell.\
For exxample: celery worker -q for_example

~~~
  airflow-worker-2:
    <<: *airflow-common
    command: celery worker -q high_cpu
  ...
~~~

## 6.8 Send a task to specific queue
Sending tasks to Multiple workers.\
\
Creating a queue in airflow: need to specify to the **command: celery worker -q ml_model**. By doing this you will be able to execute that task ml_model only on this specipic worker.\
\
Other tasks will be directed to default worker.

Add paramater to a task queue='for_example'
~~~
...
    transform = BashOperator(
        task_id='transform',
        queue='ml_model',
        bash_command='sleep 1'
    )
...
~~~
You are able to add a new celery worker, you are able to create a queue and attach that queue to a specific worker, which is very useful if you have a resource consuming task that you want to send to a specific worker with more resources than the others.

## 6.9 Concurrency
Concurrency defines the number of tasks and DAG Runs that you can execute at the same time (in parallel)\
\
Starting from the configuration settings/
**parallelism / AIRFLOW__CORE__PARALELISM**/
This defines the maximum number of task instances that can run in Airflow per scheduler. By default, you can execute up to 32 tasks at the same time. If you have 2 schedulers: 2 x 32 = 64 tasks.What value to define here depends on the resources you have and the number of schedulers running.\
\
**max_active_tasks_per_dag / AIRFLOW__CORE__MAX_ACTIVE_TASKS_PER_DAG**
This defines the maximum number of task instances allowed to run concurrently in each DAG. By default, you can execute up to 16 tasks at the same time for a given DAG across all DAG Runs./
\
**max_active_runs_per_dag / AIRFLOW__CORE__MAX_ACTIVE_RUNS_PER_DAG**
This defines the maximum number of active DAG runs per DAG. By default, you can have up to 16 DAG runs per DAG running at the same time.

