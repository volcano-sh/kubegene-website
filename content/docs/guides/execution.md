+++
title = "Execution Creation"
description = "This page shows how to create complicated executions"
weight = 10
toc = true
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 4
+++

## Before you begin

If you haven't already done so, setup KubeGene by the installation guide.

## Jobs with depend type `whole`

```yaml
apiVersion: execution.kubegene.io/v1alpha1
kind: Execution
metadata:
  name: exec-whole
  namespace: default
spec:
  tasks:
  - commands:
    - echo a1
    - echo a2
    dependents: null
    image: ubuntu
    name: a
    type: Job
  - commands:
    - echo b1
    - echo b2
    dependents:
    - target: a
      type: whole
    image: ubuntu
    name: b
    type: Job
```

Running `kubectl get job`, refer to kube-dag run four jobs:

```shell
NAME             COMPLETIONS   DURATION   AGE
exec-whole.a.0   1/1           2s         18s
exec-whole.a.1   1/1           2s         18s
exec-whole.b.0   1/1           2s         16s
exec-whole.b.1   1/1           1s         16s
```
 `exec-whole.a.0` and `exec-whole.a.1` belongs to group *a* defined by task name `.spec.tasks[0].name`
 `exec-whole.b.0` and `exec-whole.b.1` belongs to group *b*.

 **Note:** Job names are generated from Execution name and task name.

Running `kubectl get execution exec-whole -oyaml`, and refer to the depend relation from `{{.status.vertices}}`:
```shell
apiVersion: execution.kubegene.io/v1alpha1
kind: Execution
metadata:
  creationTimestamp: 2018-10-16T08:18:57Z
  generation: 1
  name: exec-whole
  namespace: gene
  resourceVersion: "289698"
  selfLink: /apis/execution.kubegene.io/v1alpha1/namespaces/gene/executions/exec-whole
  uid: 20f5b454-d11c-11e8-ad51-286ed488dc10
spec:
  tasks:
  - commands:
    - echo a1
    - echo a2
    dependents: null
    image: ubuntu
    name: a
    type: Job
  - commands:
    - echo b1
    - echo b2
    dependents:
    - target: a
      type: whole
    image: ubuntu
    name: b
    type: Job
status:
  finishedAt: 2018-10-16T08:19:01Z
  message: execution has run successfully
  phase: Succeeded
  startedAt: 2018-10-16T08:18:57Z
  vertices:
    exec-whole.a.0:
      children:
      - exec-whole.b.0
      - exec-whole.b.1
      finishedAt: null
      id: exec-whole.a.0
      message: success
      name: exec-whole.a.0
      phase: Succeeded
      startedAt: null
      type: ""
    exec-whole.a.1:
      children:
      - exec-whole.b.0
      - exec-whole.b.1
      finishedAt: null
      id: exec-whole.a.1
      message: success
      name: exec-whole.a.1
      phase: Succeeded
      startedAt: null
      type: ""
    exec-whole.b.0:
      finishedAt: null
      id: exec-whole.b.0
      message: success
      name: exec-whole.b.0
      phase: Succeeded
      startedAt: null
      type: ""
    exec-whole.b.1:
      finishedAt: null
      id: exec-whole.b.1
      message: success
      name: exec-whole.b.1
      phase: Succeeded
      startedAt: null
      type: ""
```
Jobs of task `b` depend on jobs of task `a`, and the result verifies that kube-dag guaranteed the running sequence.
The dependent graph is :

     a0  a1
     | \/ |
     | /\ |
     b0   b1

## Jobs with depend type `iterate`

Note that if task `b` depends on task `a` with `iterate` depend type, then they must have same number of jobs. That is to say the length of task's `commands` must be equal. Otherwise, it will fail in validation.

Following is an example:

```yaml
apiVersion: execution.kubegene.io/v1alpha1
kind: Execution
metadata:
  name: exec-iterate
  namespace: default
spec:
  tasks:
  - commands:
    - echo a1
    - echo a2
    dependents: null
    image: ubuntu
    name: a
    type: Job
  - commands:
    - echo b1
    - echo b2
    dependents:
    - target: a
      type: iterate
    image: ubuntu
    name: b
    type: Job
```

Running `kubectl get job`, refer to kube-dag run four jobs:

```shell
NAME               COMPLETIONS   DURATION   AGE
exec-iterate.a.0   1/1           2s         19s
exec-iterate.a.1   1/1           2s         19s
exec-iterate.b.0   1/1           2s         17s
exec-iterate.b.1   1/1           2s         17s
```

Running `kubectl get execution exec-iterate -oyaml`, and refer to the depend relation from `{{.status.vertices}}`:

```shell
apiVersion: execution.kubegene.io/v1alpha1
kind: Execution
metadata:
  creationTimestamp: 2018-10-16T07:01:06Z
  generation: 1
  name: exec-iterate
  namespace: gene
  resourceVersion: "284961"
  selfLink: /apis/execution.kubegene.io/v1alpha1/namespaces/gene/executions/exec-whole
  uid: 40acb318-d111-11e8-ad51-286ed488dc10
spec:
  tasks:
  - commands:
    - echo a1
    - echo a2
    dependents: null
    image: ubuntu
    name: a
    type: Job
  - commands:
    - echo b1
    - echo b2
    dependents:
    - target: a
      type: iterate
    image: ubuntu
    name: b
    type: Job
status:
  finishedAt: 2018-10-16T07:01:10Z
  message: execution has run successfully
  phase: Succeeded
  startedAt: 2018-10-16T07:01:06Z
  vertices:
    exec-iterate.a.0:
      children:
      - exec-iterate.b.0
      finishedAt: null
      id: exec-iterate.a.0
      message: success
      name: exec-iterate.a.0
      phase: Succeeded
      startedAt: null
      type: ""
    exec-iterate.a.1:
      children:
      - exec-iterate.b.1
      finishedAt: null
      id: exec-iterate.a.1
      message: success
      name: exec-whole.a.1
      phase: Succeeded
      startedAt: null
      type: ""
    exec-iterate.b.0:
      finishedAt: null
      id: exec-iterate.b.0
      message: success
      name: exec-iterate.b.0
      phase: Succeeded
      startedAt: null
      type: ""
    exec-iterate.b.1:
      finishedAt: null
      id: exec-iterate.b.1
      message: success
      name: exec-iterate.b.1
      phase: Succeeded
      startedAt: null
      type: ""
```

The dependent graph is :

     a0   a1
     |     |
     |     |
     b0   b1

## Jobs with persistent output

In some cases, it is necessary to record the outputs of jobs. Since *Job* in kubernetes can not achieve this. But kubernetes provides [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/), which have a lifecycle independent of any individual pod that uses the PV.

Note if you have a multiple nodes kubernetes cluster, the HostPath type pv may not work. You should try to create other type pv according [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).

Create a HostPath type pv:

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: exec-pv
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp
    type: Directory
```

Create a pvc to claim the pv:

```shell
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
    name: exec-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

Check the pvc `kubectl get pvc exec-pvc --template={{.status.phase}}` and make sure it is in phase `Bound`.

Create the following execution, the jobs all output a message to the same file.

```yaml
apiVersion: execution.kubegene.io/v1alpha1
kind: Execution
metadata:
  name: exec-volume
spec:
  tasks:
  - commands:
    - echo a1 >> /tmp/hostvolume/output
    - echo a2 >> /tmp/hostvolume/output
    dependents: null
    image: ubuntu
    name: a
    type: Job
    volumes:
      volumec:
        mountFrom:
          pvc: exec-pvc
        mountPath: /tmp/hostvolume
  - commands:
    - echo b1 >> /tmp/hostvolume/output
    - echo b2 >> /tmp/hostvolume/output
    dependents:
    - target: a
      type: whole
    image: ubuntu
    name: b
    type: Job
    volumes:
      volumec:
        mountFrom:
          pvc: exec-pvc
        mountPath: /tmp/hostvolume
```

Check file content `/tmp/output` on your node.

```shell
$ cat /tmp/output
a2
a1
b2
b1
```

## Cleanup

When you have finished expeimenting with these 3 sample, cleanup using the follwing instructions.

```
$ kubectl delete execution exec-whole exec-iterate exec-volume
```

Confirm the resources all deleted:

```
$ kubectl get executions   #-- there should be no executions
$ kubectl get jobs         #-- there should be no jobs
$ kubectl get pods         #-- there should be no pods
```

## What's next

* For other advanced features, please look into Execution Api definition.