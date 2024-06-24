# Implementations / DAGs

## Standard

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


### How do Airflow's schedule intervals support the creation of incremental data pipelines, and what are the benefits of using these pipelines for large data sets?

<details>
  <summary>How Airflow's Schedule Intervals Support Incremental Data Pipelines and Their Benefits</summary>

#### How Airflow's Schedule Intervals Support Incremental Data Pipelines

1. **Discrete Time Intervals**: Airflow's schedule intervals allow you to divide time into discrete intervals (e.g., daily, weekly). This enables running a DAG for each specific interval, processing only the data associated with that time period.

2. **Last and Next Intervals**: The schedule intervals provide details about the last and expected next intervals, helping to identify the exact time range for the data to be processed in each DAG run.

3. **Incremental Processing**: By focusing on the data corresponding to each discrete time slot (the data's delta), incremental data pipelines process only new or changed data since the last run, rather than reprocessing the entire data set every time.

#### Benefits of Incremental Data Pipelines for Large Data Sets

1. **Time Efficiency**: Incremental data pipelines reduce the time required for processing by limiting the workload to only new or updated data, avoiding the need to reprocess unchanged data.

2. **Cost Efficiency**: They reduce computational costs and resource usage, as fewer resources are needed to process smaller data deltas compared to the entire data set.

3. **Scalability**: Incremental processing makes it easier to scale data pipelines to handle larger data volumes, as each run processes only a manageable portion of the data.

4. **Timely Updates**: These pipelines enable more frequent and timely updates to the data sets, as they can quickly process new data without the overhead of reprocessing everything.

5. **Backfilling and Reruns**: The combination of schedule intervals with backfilling allows for efficient historical data processing. If there are changes in the task code, clearing past results and rerunning tasks for specific intervals is straightforward, ensuring data consistency and accuracy.

Overall, Airflow's schedule intervals and incremental data pipelines offer significant advantages in managing and processing large data sets efficiently and cost-effectively.
</details>

### How long are deltas typically in Apache Airflow? What is the best practice? 

<details>
  <summary>Typical Delta Lengths and Best Practices in Apache Airflow</summary>

#### Typical Delta Lengths in Apache Airflow

1. **Common Intervals**: The typical delta lengths in Apache Airflow can vary widely depending on the specific use case and data processing requirements. Common intervals include:
   - **Hourly**: For time-sensitive applications requiring frequent updates.
   - **Daily**: Often used for end-of-day processing, reporting, and batch jobs.
   - **Weekly**: Suitable for less frequent, larger data aggregations and analyses.
   - **Monthly**: For periodic large-scale data processing tasks.

#### Best Practices for Delta Lengths

1. **Business Requirements**: Choose the delta length based on business requirements and the nature of the data. For example, real-time data processing might require hourly intervals, while financial reporting might be best suited to daily or monthly intervals.

2. **Data Volume and Velocity**: Consider the volume and velocity of incoming data. High-velocity data streams may necessitate shorter intervals (e.g., hourly), while low-volume data might be efficiently handled with daily intervals.

3. **Resource Availability**: Assess the computational resources available. Shorter intervals require more frequent processing and potentially more resources. Ensure that the chosen interval length aligns with the available infrastructure.

4. **Processing Time**: Ensure that the processing time for each interval fits within the interval length. For instance, if hourly processing takes 50 minutes, an hourly interval is appropriate, but if it takes 70 minutes, consider a longer interval.

5. **Data Dependencies**: Align the intervals with data dependencies. If downstream tasks rely on the completion of upstream tasks, ensure that the intervals are synchronized to avoid data inconsistencies.

6. **Error Handling and Reruns**: Choose intervals that allow for efficient error handling and reruns. Smaller intervals can isolate issues to specific time slots, making it easier to identify and correct errors.

7. **Incremental vs. Full Loads**: Balance between incremental and full data loads. Incremental processing with appropriate intervals minimizes reprocessing overhead, while full loads might be reserved for periodic comprehensive updates.

By carefully considering these factors, you can select the most appropriate delta lengths for your Apache Airflow workflows, ensuring efficient and reliable data processing.
</details>

### Explain the concept of backfilling in Airflow and describe how it can be used to handle historical data processing and task reprocessing.

<details>
  <summary>Backfilling in Airflow</summary>

#### Concept of Backfilling in Airflow

1. **Definition**: Backfilling in Airflow refers to the process of executing a DAG for historical time intervals that were missed or need to be reprocessed.

2. **Purpose**: It allows for the creation or updating of datasets for past periods, ensuring that all data intervals are processed even if the DAG was not initially run for those intervals.

#### Handling Historical Data Processing

1. **Historical Intervals**: By specifying a range of past intervals, backfilling runs the DAG for each of those periods, ensuring that historical data is processed accurately.

2. **Incremental Pipelines**: Each backfill run processes data for the specified past interval, making it possible to build or update datasets incrementally without reprocessing the entire dataset at once.

#### Task Reprocessing

1. **Rerunning Tasks**: If task code or logic changes, backfilling allows for rerunning tasks for historical intervals, updating results based on the new code.

2. **Clearing Results**: Clearing the results of past runs and using backfilling enables reprocessing of tasks, ensuring consistency and correctness of the data across all time intervals.

Backfilling is a powerful feature in Airflow that ensures comprehensive and accurate data processing, both for historical data and for updating results when changes occur.
</details>

### What is the difference between a Task and an Operator in Airflow? 

<details>
<summary>Details</summary>
In this context and throughout the Airflow documentation, we see the terms operator
and task used interchangeably. From a user’s perspective, they refer to the same
thing, and the two often substitute each other in discussions. Operators provide the
implementation of a piece of work. Airflow has a class called BaseOperator and many
subclasses inheriting from the BaseOperator, such as PythonOperator, EmailOperator,
and OracleOperator.

There is a difference, though. Tasks in Airflow manage the execution of an operator;
they can be thought of as a small wrapper or manager around an operator that
ensures the operator executes correctly. The user can focus on the work to be done
by using operators, while Airflow ensures correct execution of the work via tasks.

Tasks manage individual operator state and display state changes to the user, of an operator's execution.

</details>

### What is Airflow's syntax for schedule intervals? 

### How does the cross_downstream method work for declaring dependencies?

<details>
  <summary>How the cross_downstream Method Works</summary>

#### How the `cross_downstream` Method Works

1. **Purpose**: The `cross_downstream` method in Airflow sets up dependencies between all upstream tasks and all downstream tasks.

2. **Usage**: `cross_downstream(upstream_tasks, downstream_tasks)`

3. **Functionality**: 
   - **Upstream Tasks**: A list of tasks that should be completed before proceeding.
   - **Downstream Tasks**: A list of tasks that should start after all upstream tasks are completed.

4. **Dependencies**: Creates a "cross product" of dependencies, meaning each task in `upstream_tasks` is set to trigger every task in `downstream_tasks`.

This method simplifies the process of setting complex task dependencies where multiple upstream tasks need to trigger multiple downstream tasks.
</details>

### What is an example of the chain method to manage dependencies?

<details>
   <summary>Answer</summary>

```
chain(op1, [op2, op3], [op4, op5], op6)
```
</details>

### What are common variables avaliable at runtime within a Task?

<details>

In Apache Airflow, there are several predefined variables (also known as macros) that you can use within your tasks. Here are some of the most common ones:

- `ds`: the execution date as `YYYY-MM-DD`.
- `ds_nodash`: the execution date as `YYYYMMDD`.
- `ts`: the execution date and time as `YYYY-MM-DDTHH:MM:SS.ssssss`.
- `ts_nodash`: the execution date and time without dashes or colons.
- `yesterday_ds`: the day before the execution date as `YYYY-MM-DD`.
- `yesterday_ds_nodash`: the day before the execution date as `YYYYMMDD`.
- `tomorrow_ds`: the day after the execution date as `YYYY-MM-DD`.
- `tomorrow_ds_nodash`: the day after the execution date as `YYYYMMDD`.
- `END_DATE`: the end date of the interval as `YYYY-MM-DD`.
- `execution_date`: the execution date as a `datetime` object.
- `prev_execution_date`: the execution date of the previous DAG run as a `datetime` object.
- `next_execution_date`: the execution date of the next DAG run as a `datetime` object.
- `latest_date`: the latest execution date as `YYYY-MM-DD`.
- `macros`: a module containing all the above macros, plus some additional ones like `uuid`, `time`, `random`, etc.

You can use these variables in your tasks like so:

```python
from airflow.operators.python import PythonOperator

def print_execution_date(**context):
    print(context['ds'])

task = PythonOperator(
    task_id='print_execution_date',
    python_callable=print_execution_date,
    provide_context=True,
    dag=dag,
)
```

In this example, the `print_execution_date` function will print the execution date of the DAG run.
</details>

### How does branching operate in Airflow? 

### What are a list of the most common Task context variables? 

### How does accessing Task context variables differ in BashOperator vs PythonOperator?


###

###

###

## Jinja

### What are basics of Jinja templating?

### How do conditional statements work in Jinja?

<details>

Jinja is a templating language for Python, often used to generate HTML or XML documents. It also supports control structures like conditional statements and loops.

Conditional statements in Jinja work similarly to those in Python. You can use the `{% if %}`, `{% elif %}`, and `{% else %}` tags to create conditional statements. Here's a basic example:

```jinja
{% if user.is_admin %}
  <p>Welcome, admin!</p>
{% elif user.is_logged_in %}
  <p>Welcome, user!</p>
{% else %}
  <p>Welcome, guest!</p>
{% endif %}
```

In this example, if `user.is_admin` is `True`, it will display "Welcome, admin!". If `user.is_admin` is `False` but `user.is_logged_in` is `True`, it will display "Welcome, user!". If both are `False`, it will display "Welcome, guest!".

You can also use logical operators like `and`, `or`, and `not` in your conditions:

```jinja
{% if user.is_logged_in and not user.is_admin %}
  <p>You are a regular user.</p>
{% endif %}
```

In this example, the message "You are a regular user." will be displayed if `user.is_logged_in` is `True` and `user.is_admin` is `False`.

</details>

### What does the {:02} mean in this case in Jinja? `{{ '{:02}'.format(execution_date.hour) }}`

<details>

In Python and Jinja, {:02} is a format specification for an integer type. Here's what it means:

: is the separator between the field name and the format specification.
0 means that the output should be zero-padded.
2 specifies the width of the field. In this case, it's 2 characters wide.
So, {:02} will format an integer as a string of exactly 2 characters, padding it with zeros if necessary. For example, 1 would be formatted as '01', 10 would remain '10', and so on.

In your specific case, '{:02}'.format(execution_date.hour) will format the hour of the execution date as a zero-padded string of 2 characters. This is useful when you need to construct a string that includes a time, and you want to ensure that the hour is always represented by exactly 2 digits.
</details>