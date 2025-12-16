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


