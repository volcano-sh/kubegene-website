+++
title = "KubeDag Design and Architecture"
description = "KubeDag design and architecture"
weight = 10
toc = true
aliases = ["/docs/about/"]
[menu.main]
  parent = "What is KubeGene?"
  weight = 2
+++

## Overview

KubeDag is a workflow engine on Kubernetes platform. It is dedicated to making Gene Sequencing workflow execute in container easily. Anywhere you are running Kubernetes, you should be able to run KubeGene.

## Goals

The project is committed to the following design ideals:
* Easy, portable deployments on various kubernetes env. Currently we support kubernetes 1.7+
* Easy to use. Provide an easy to use template for different Gene Sequencing scenes. You can define a complex workflow easily.
* High availability. Provide fail over, even if an instance down.
* Flexible control. You can limit any reousrce requirement, concurrently running jobs, and select nodes to run on.


## Architecture

KubeDag implements the workflow as [Execution](/docs/about/execution) using [Kubernetes CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions).

KubeDag consists of three main components:
* Execution controller
* Job controller
* Status updater

### Execution controller

1. *Execution controller* watches `executions` resource from kubernetes. When a new `Execution` created, it will validate the execution and create a graph for it. The graph is used later by *Job controller* to run real kubernetes `Job`.

1. Then *Execution controller* notifies `Job controller` asynchronously by sending an event through work queue. The event includes the execution info and a type.

1. Watches `jobs` from kubernetes. And filter out that donot belong to any execution. Then call *Status updater* to mark execution status.

### Job controller

1. *Job controller* reads execution from the work queue. And run jobs accoding the type. There are currently two types: one for new added execution, the other for `Job` completed.

1. New added *Execution*
*Job controller* gets all vertices of the graph which In-degress equals to 0. And create these jobs in kubernetes according to the parallelism limit. That is to say running all jobs which have no dependents first.

1. A `Job` of *Execution* completed
When job `A` is completed, *Job controller* would find other suitable Jobs to run. Following is how it is done:

* It iterates *Execution* graph and find the Children jobs.
* And then filter out the jobs whose dependents are not all completed. The left ones are ready to run.

### Status updater

*Status updater* is used to update Execution status according to the children jobs status.
