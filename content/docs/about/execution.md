+++
title =  "Execution Overview"
description = "This page provides an overview of `Execution`, the smallest deployable object in the Kubegene object model."
weight = 10
aliases = ["/docs/execution/"]
[menu.main]
  parent = "What is Execution?"
  weight = 1
+++

## Understanding Execution

An *Execution* is the basic building block of Kubegene--the smallest and simplest unit in the Kubegene object model that you create or deploy. An Execution represents a group of running jobs on your kubernetes cluster.

You describe a batch of jobs in an Execution object to achieve a common goal(like Gene Sequencing), each job can complete part of the whole work. And all jobs may have some dependent relationship. The jobs can share outputs by pvc or other datastore. Kubegene would schedule each job by the dependents relation.

**Note:** You should not manage Jobs owned by an Execution. And currently we do not support modification of an existing Execution object.

See [Creating an Execution](## Creating an Execution) for more information on how to create an Execution.

## Creating an Execution

The following is an example of an Execution. It brings up four jobs with the below dependents relation:

          a
         / \
        /   \
       b     c
      /
     /
    d

```yaml
apiVersion: execution.kubegene.io/v1alpha1
kind: Execution
metadata:
  name: test-exec
  namespace: default
spec:
  tasks:
  - commands:
    - echo A
    dependents: null
    image: ubuntu
    name: a
    type: Job
  - commands:
    - echo B
    dependents:
    - target: a
      type: whole
    image: ubuntu
    name: b
    type: Job
  - commands:
    - echo C
    dependents:
    - target: a
      type: whole
    image: ubuntu
    name: c
    type: Job
  - commands:
    - echo D
    dependents:
    - target: b
      type: whole
    image: ubuntu
    name: d
    type: Job
```

In this example:

* An Execution named `test-exec` is created, indicated by the `.metadata.name` field.
* The Execution creates four groups of jobs, indicated by `commands` field.
* The `dependents` filed defines which jobs to depend on. Can define multi-dependents.
* The `image` field defines the image used by the group of jobs.

{{< note >}}
  **Note:** `tasks` is an array of tasks. Each one is consisted of one or multi similar jobs which use same image and require same resources.
{{< /note >}}

Next, run `kubectl get executions`. The output is similar to the following:

```shell
NAME        AGE
test-exec   8s
```

To see the jobs created by the execution, run `kubectl get jobs`:

```shell
NAME            COMPLETIONS   DURATION   AGE
test-exec.a.0   1/1           1s         16s
test-exec.b.0   1/1           2s         15s
test-exec.c.0   1/1           2s         15s
test-exec.d.0   1/1           2s         13s
```

And we can check the job `startTime` and `completionTime` to verify the jobs start up sequences.

Running `kubectl get pods` should now show the pods belonging to the jobs:

```shell
NAME                        READY   STATUS      RESTARTS   AGE
test-exec.a.0-jcqt5         0/1     Completed   0          20s
test-exec.b.0-nj7bt         0/1     Completed   0          19s
test-exec.c.0-xlrlp         0/1     Completed   0          19s
test-exec.d.0-nsccq         0/1     Completed   0          17s
```

## Execution status

Each job is a vertex in DAG. You can check the execution status running `kubectl get executions test-exec`. `.status.phase` shows the runing phase. And `.status.vertices` shows the jobs belonging to this execution and their dependency relation.

```shell
apiVersion: execution.kubegene.io/v1alpha1
kind: Execution
metadata:
  creationTimestamp: 2018-10-15T07:46:05Z
  generation: 1
  name: test-exec
  namespace: gene
  resourceVersion: "204944"
  selfLink: /apis/execution.kubegene.io/v1alpha1/namespaces/gene/executions/test-exec
  uid: 5f144885-d04e-11e8-ad51-286ed488dc10
spec:
  ...
status:
  finishedAt: 2018-10-15T07:47:52Z
  message: execution has run successfully
  phase: Succeeded
  startedAt: 2018-10-15T07:47:47Z
  vertices:
    test-exec.a.0:
      children:
      - test-exec.b.0
      - test-exec.c.0
      finishedAt: null
      id: test-exec.a.0
      message: success
      name: test-exec.a.0
      phase: Succeeded
      startedAt: null
      type: ""
    test-exec.b.0:
      children:
      - test-exec.d.0
      finishedAt: null
      id: test-exec.b.0
      message: success
      name: test-exec.b.0
      phase: Succeeded
      startedAt: null
      type: ""
    test-exec.c.0:
      finishedAt: null
      id: test-exec.c.0
      message: success
      name: test-exec.c.0
      phase: Succeeded
      startedAt: null
      type: ""
    test-exec.d.0:
      finishedAt: null
      id: test-exec.d.0
      message: success
      name: test-exec.d.0
      phase: Succeeded
      startedAt: null
      type: ""

```

## What's next

* Learn more about Execution behavior:
  * How to create complicated Executions
  * Delete an Execution and its owned resources.
  * Create an execution using GCS
