# Airflow Questions
## Basics

### What are the differences between Apache Spark and Apache Airflow? 

### Give three example of use cases for Apache Airflow.
<details>

#### ETL Pipelines - Made-Up
- A daily ETL job runs that extracts sales data from an e-commerce's Postgres DB, cleans and aggregates the data, and loads it to a data warehouse.
#### Data Science - Made-Up
- A workflow runs every week that extracts and cleans data prior to model training. It then orchestrates the execution of model training scripts, distributing the load. Airflow then handles tasks for model evaluation, packaging, and deployment. 
#### Reporting - Made-Up
- A workflow runs daily that aggregates financial data from several sources, generates a financial summary report in a variety of formats, and executes tasks to distribute the reports by sending over email, uploading to NAS drives, publishing to websites, etc. 
</details>

### What is a Sensor in Apache Airflow, and how does it work?

<details>
A Sensor in Apache Airflow is a special type of operator that waits for a certain condition to be met before proceeding. This condition could be the presence of a file in a specific location, the arrival of a certain time, or the availability of a resource. Sensors are used to create dependencies based on external events or conditions.

Sensors continuously check the specified condition at regular intervals (configured by `poke_interval`). Once the condition is met, the sensor task is marked as successful, allowing downstream tasks to execute.

Example of a FileSensor in Airflow:

```python
from airflow import DAG
from airflow.operators.sensors import FileSensor
from airflow.operators.dummy_operator import DummyOperator
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
}

with DAG('file_sensor_example', default_args=default_args, schedule_interval='@daily') as dag:
    
    start = DummyOperator(task_id='start')
    
    wait_for_file = FileSensor(
        task_id='wait_for_file',
        filepath='/path/to/the/file.txt',
        poke_interval=30,  # Check every 30 seconds
        timeout=600,  # Timeout after 10 minutes
    )
    
    end = DummyOperator(task_id='end')
    
    start >> wait_for_file >> end
```
</details>

### What are the differences between `poke` and `reschedule` modes in Airflow Sensors?

<details>
Apache Airflow Sensors can operate in two modes: `poke` and `reschedule`.

1. **Poke Mode:**
   - The sensor task is running continuously and checks the condition at a regular interval (`poke_interval`).
   - It consumes a worker slot while waiting.
   - Suitable for short wait times and scenarios where the condition is likely to be met quickly.

2. **Reschedule Mode:**
   - The sensor task suspends itself after each check and is rescheduled after the `poke_interval`.
   - It frees up the worker slot between checks, making it more resource-efficient.
   - Suitable for long wait times or conditions that may take a significant amount of time to be met.

Example of a TimeDeltaSensor with reschedule mode:

```python
from airflow import DAG
from airflow.sensors.time_delta_sensor import TimeDeltaSensor
from airflow.operators.dummy_operator import DummyOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
}

with DAG('timedelta_sensor_reschedule_example', default_args=default_args, schedule_interval='@daily') as dag:
    
    start = DummyOperator(task_id='start')
    
    wait_for_timedelta = TimeDeltaSensor(
        task_id='wait_for_timedelta',
        delta=timedelta(minutes=5),
        mode='reschedule',
    )
    
    end = DummyOperator(task_id='end')
    
    start >> wait_for_timedelta >> end
```
</details>


### What are considerations for using Sensors for event-based, adhoc triggering of DAGs?

<details>

1. **Resource Utilization:**
   - **Poke Mode:** Continuous running can consume worker slots, potentially causing resource contention.
   - **Reschedule Mode:** More efficient as it frees worker slots between checks.

2. **Timeout Settings:**
   - Ensure sensors have appropriate timeout settings to avoid indefinitely blocking worker slots.

3. **Poke Interval:**
   - Set a reasonable `poke_interval` to balance between prompt detection and resource consumption.

4. **Concurrency and Parallelism:**
   - Excessive use of sensors can affect the concurrency limits and overall parallelism of the Airflow environment.

5. **Event Frequency:**
   - High-frequency events may benefit from a more efficient event handling mechanism (e.g., external triggers, message queues) instead of sensors.

6. **Error Handling:**
   - Implement robust error handling and retries for sensors to manage transient failures gracefully.

7. **DAG Dependencies:**
   - Properly manage DAG dependencies to avoid circular dependencies and ensure smooth DAG execution flow.

8. **Monitoring and Alerts:**
   - Set up monitoring and alerting for sensor tasks to detect and respond to issues promptly.

9. **Performance Impact:**
   - Evaluate the performance impact of sensors on the overall system, especially in high-load scenarios.

</details>

### What are Triggers in Airflow?

<details>
The Airflow scheduler monitors all tasks and all DAGs, and triggers the task instances whose dependencies have been met. Behind the scenes, it spins up a subprocess, which monitors and stays in sync with a folder for all DAG objects it may contain, and periodically (every minute or so) collects DAG parsing results and inspects active tasks to see whether they can be triggered.

Can run a trigger through the CLI, API, or webbooks. Triggers are allowed based on successful completion of upstream tasks. Triggers run external to your scheduler, and need to be installed separately depending on environment. 
</details>

### What is the difference between Triggers and Sensors?

<details>

**Triggers:**
- **Purpose:** Initiate the execution of a DAG or task based on predefined conditions.
- **Types:**
  - **Time-based:** Schedule DAGs using cron expressions or preset schedules.
  - **Manual/External:** Trigger DAGs via CLI, REST API, or webhooks.
  - **Task Dependencies:** Trigger tasks based on the completion of upstream tasks.
- **Usage:** Primarily used to start DAGs or tasks on a schedule or in response to external events.

**Sensors:**
- **Purpose:** Monitor and wait for a specific condition or event before allowing a task to proceed.
- **Operation Modes:**
  - **Poke Mode:** Continuously runs and checks for the condition at regular intervals.
  - **Reschedule Mode:** Suspends itself between checks to free resources.
- **Types:** FileSensor, TimeDeltaSensor, ExternalTaskSensor, etc.
- **Usage:** Used within a DAG to create dependencies on external conditions, ensuring tasks only execute when certain criteria are met.

**Summary:**
- **Triggers:** Start DAGs or tasks based on schedules or external commands.
- **Sensors:** Wait for conditions within a DAG to be met before proceeding with task execution.

</details>

###
