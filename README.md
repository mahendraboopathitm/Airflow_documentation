# Airflow_documentation

# Apache Airflow – Core Concepts Documentation

## Overview

Apache Airflow is an **open-source workflow orchestration platform** used to **schedule, monitor, and manage workflows**.  
Workflows are defined as **code (Python)**, which makes them dynamic, version-controlled, and scalable.

Airflow does **not process data itself**.  
Instead, it **coordinates and orchestrates** tasks that run in external systems such as databases, APIs, cloud services, or compute engines.

---

## 1. Introduction to Apache Airflow

Apache Airflow helps in managing complex workflows by:
- Defining task dependencies
- Scheduling task execution
- Handling retries and failures
- Providing monitoring and logging

Airflow is widely used in:
- Data engineering pipelines
- ETL / ELT workflows
- Batch processing systems
- Machine learning orchestration

---

## 1.1 Core Concepts

Airflow is built on four core concepts:
- DAGs
- Operators
- Tasks
- DAG-to-DAG triggers

Understanding these concepts is essential before building pipelines.

---

## 1.1.1 DAG Definition (Directed Acyclic Graph)

### What is a DAG?

A **DAG (Directed Acyclic Graph)** represents a workflow definition in Airflow.

- **Directed**: Tasks have a defined execution order
- **Acyclic**: No circular dependencies are allowed
- **Graph**: Tasks are nodes, dependencies are edges

A DAG defines **what should run and in what order**, but it does not execute tasks itself.

---

### Key Characteristics

- Written in Python
- Parsed continuously by the scheduler
- Represents workflow structure only
- Execution happens through DAG Runs

---

### Important DAG Parameters

| Parameter | Description |
|--------|------------|
| `dag_id` | Unique identifier for the DAG |
| `start_date` | Logical start date of the workflow |
| `schedule_interval` | Execution frequency |
| `catchup` | Whether to run missed schedules |
| `default_args` | Common task-level configurations |

---

### Conceptual Understanding

A DAG is like a **blueprint**:
- It describes the plan
- It does not perform the work
- Tasks are executed based on this plan

---

## 1.1.2 Operators

### What is an Operator?

An **Operator** defines **what kind of action** should be performed.

Operators act as **templates** for tasks.

Examples:
- Run a Python function
- Execute a shell command
- Send an email
- Trigger another DAG

---

### Operator vs Task

| Operator | Task |
|-------|------|
| Blueprint | Actual execution |
| Defines logic | Executes logic |
| Reusable | DAG-specific |

---

### Common Operator Types

- **Action Operators**: Perform actions (PythonOperator, BashOperator)
- **Sensor Operators**: Wait for conditions (FileSensor, ExternalTaskSensor)
- **Transfer Operators**: Move data between systems
- **Control Flow Operators**: Control execution path

---

## 1.1.3 Tasks

### What is a Task?

A **Task** is a **single execution unit** in a DAG.  
It is created when an operator is instantiated inside a DAG.

---

### Task Lifecycle

1. Task is created by the scheduler
2. Task waits for dependencies
3. Task is queued
4. Executor runs the task
5. Status is updated (success / failed / retry)

---

### Task Characteristics

- Atomic (does one job)
- Independent
- Retryable
- Stateless (recommended)

---

## 1.1.4 DAG to DAG Trigger

### What is DAG-to-DAG Triggering?

DAG-to-DAG triggering allows **one DAG to start another DAG**.

This is useful when workflows need to be:
- Modular
- Reusable
- Decoupled

---

### How It Works

- A task in DAG A triggers DAG B
- DAG B starts a new DAG Run
- DAGs operate independently after triggering

---

### Use Cases

- Breaking large workflows into smaller DAGs
- Event-based orchestration
- Multi-stage pipelines (ingestion → transformation → reporting)

---

## 1.2 Scheduler

### What is the Scheduler?

The **Scheduler** is the component responsible for **deciding when tasks should run**.

It continuously:
- Reads DAG files
- Creates DAG Runs
- Evaluates task dependencies
- Sends runnable tasks to the executor

---

### Responsibilities

- DAG parsing
- Scheduling DAG Runs
- Dependency resolution
- Task queuing

---

### Important Note

The scheduler:
- Does NOT execute tasks
- Does NOT run business logic
- Only makes scheduling decisions

---

## 1.3 Executor

### What is an Executor?

The **Executor** determines **how and where tasks are executed**.

It acts as a bridge between:
- Scheduler
- Workers

---

### Execution Flow


---

### Common Executor Types

| Executor | Description |
|--------|------------|
| SequentialExecutor | Single task at a time (testing) |
| LocalExecutor | Parallel tasks on one machine |
| CeleryExecutor | Distributed execution |
| KubernetesExecutor | Runs tasks as pods |
| CeleryKubernetesExecutor | Hybrid execution |

---

### Key Concept

Changing the executor **does not change DAG code**.  
Only execution behavior changes.

---

## 1.4 Operators (Detailed Categories)

### Action Operators

Used to perform direct actions.

Examples:
- PythonOperator
- BashOperator
- EmailOperator

---

### Sensor Operators

Used to **wait for conditions** before proceeding.

Examples:
- Waiting for a file
- Waiting for a table
- Waiting for another task

Sensors continuously check conditions until satisfied.

---

### Transfer Operators

Used to move data between systems.

Examples:
- Cloud storage to database
- Database to warehouse

---

### Control Flow Operators

Used to control execution paths.

Examples:
- Branching based on conditions
- Skipping tasks
- Short-circuiting workflows

---

## Summary

| Component | Purpose |
|--------|--------|
| DAG | Workflow definition |
| Operator | Type of action |
| Task | Unit of execution |
| Scheduler | Decides when tasks run |
| Executor | Decides how tasks run |
| DAG Trigger | Orchestrates multiple DAGs |

---

# Apache Airflow – Detailed Concepts & Scenarios

This README provides **detailed explanations, reasons for usage, and practical examples** for core Apache Airflow operators, DAG concepts, and real-world scenarios. This document is intended for **learning + interview preparation**.

---

## 1. PythonOperator

### What it is

The `PythonOperator` is used to execute a **Python callable function** as a task in an Airflow DAG.

### Why we use it

* To run business logic written in Python
* To transform data, call APIs, validate data
* Most flexible operator in Airflow

### How it works

Airflow imports your Python function and executes it during task runtime.

### Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime


def print_message():
    print("Hello from PythonOperator")

with DAG(
    dag_id="python_operator_example",
    start_date=datetime(2024, 1, 1),
    schedule_interval=None
) as dag:

    task = PythonOperator(
        task_id="print_task",
        python_callable=print_message
    )
```

---

## 2. Sensor

### What it is

A Sensor **waits for a condition to be met** before allowing downstream tasks to run.

### Why we use it

* Wait for file arrival
* Wait for database record
* Wait for external system completion

### How it works

Sensors keep checking ("poking") at a defined interval until the condition is satisfied.

### Example – FileSensor

```python
from airflow.sensors.filesystem import FileSensor

file_sensor = FileSensor(
    task_id='wait_for_file',
    filepath='/data/input/data.csv',
    poke_interval=30,
    timeout=600
)
```

---

## 3. SubDagOperator (⚠ Deprecated – Conceptual Knowledge)

### What it is

Allows a DAG to be embedded inside another DAG.

### Why it was used

* Logical grouping of tasks
* Reusability

### Why NOT recommended now

* Scheduler performance issues
* Complex dependency management

### Modern Alternative

* TaskGroups

### Example (Conceptual)

```python
SubDagOperator(
    task_id='subdag_task',
    subdag=subdag_function()
)
```

---

## 4. TriggerDagRunOperator

### What it is

Used to **trigger another DAG** from the current DAG.

### Why we use it

* DAG-to-DAG orchestration
* Modular pipelines

### Example

```python
from airflow.operators.trigger_dagrun import TriggerDagRunOperator

trigger = TriggerDagRunOperator(
    task_id='trigger_target_dag',
    trigger_dag_id='target_dag'
)
```

---

## 5. BashOperator

### What it is

Executes **shell commands** inside a task.

### Why we use it

* Run scripts
* Execute CLI commands
* Lightweight tasks

### Example

```python
from airflow.operators.bash import BashOperator

bash_task = BashOperator(
    task_id='run_bash',
    bash_command='echo "Hello Airflow"'
)
```

---

## 6. Workflow Management

### What it is

Designing, scheduling, monitoring, and retrying tasks in a controlled way.

### Why it matters

* Ensures reliable data pipelines
* Handles failures and retries automatically

### Example

A DAG with retries, dependencies, and scheduling is workflow management.

---

## 7. Default Arguments

### What it is

Common parameters shared across tasks.

### Why we use it

* Avoid repetition
* Centralized control

### Example

```python
default_args = {
    'owner': 'airflow',
    'retries': 2,
    'retry_delay': timedelta(minutes=5)
}
```

---

## 8. Schedule Interval

### What it is

Defines **when and how often** a DAG runs.

### Examples

```python
schedule_interval='@daily'
schedule_interval='0 2 * * *'
schedule_interval=None  # Manual only
```

---

## 9. Catchup

### What it is

Whether Airflow should run **past missed DAG runs**.

### Why important

Avoid unexpected backfills.

### Example

```python
catchup=False
```

---

## 10. Task Dependencies

### What it is

Defines execution order of tasks.

### Example

```python
task1 >> task2
```

---

## 11. set_upstream() / set_downstream()

### What it is

Programmatic way to set dependencies.

### Example

```python
task2.set_upstream(task1)
task1.set_downstream(task2)
```

---

# 🔹 REAL-WORLD SCENARIOS

## 1. File-based DAG trigger (wait for file arrival)

### Use case

Run pipeline only when a file arrives.

### Example

```python
FileSensor(...)
```

---

## 2. DAG-to-DAG dependency handling

### Use case

One pipeline depends on another.

### Example

```python
TriggerDagRunOperator(...)
```

---

## 3. Conditional task execution (branching)

### Use case

Run different tasks based on conditions.

### Example

```python
from airflow.operators.branch import BranchPythonOperator
```

---

## 4. Sequential vs Parallel task execution

### Sequential

```python
t1 >> t2 >> t3
```

### Parallel

```python
t1 >> [t2, t3]
```

---

## 5. Task retry on transient failure

### Use case

Temporary API/network failures.

### Example

```python
retries=3
retry_delay=timedelta(minutes=2)
```

---

## 6. Partial DAG failure handling

### Use case

Some tasks fail, others succeed.

### How Airflow handles

Only failed tasks marked failed; downstream blocked.

---

## 7. Rerunning only failed tasks

### How

* UI → Clear failed tasks
* Scheduler reruns only failed ones

---

## 8. Backfilling historical data

### Use case

Load past data.

### Command

```bash
airflow dags backfill dag_id -s 2024-01-01 -e 2024-01-05
```

---

## 9. Preventing duplicate data on reruns

### Techniques

* Idempotent logic
* Merge instead of insert
* Use execution_date

---

## 10. Manual trigger vs scheduled run mismatch

### Problem

Manual runs don’t match scheduled dates.

### Solution

Use `{{ ds }}` instead of system date.

---

## 11. Handling long-running sensors

### Problem

Sensors occupy worker slots.

### Solution

Use **reschedule mode**.

### Example

```python
mode='reschedule'
```

---



