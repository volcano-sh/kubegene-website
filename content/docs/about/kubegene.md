+++
title = "kubegene Overview"
description = "This page provides an overview of `kubegene`"
weight = 10
toc = true
aliases = ["/docs/about/"]
[menu.main]
  parent = "What is Kubegene?"
  weight = 1
+++

## What is kubegene

The Kubegene is dedicated to making gene sequencing process simple, portable and scalable. It provides a complete solution for gene sequencing on the kubernetes and supports mainstream biological gene sequencing scenarios such as DNA, RNA, and liquid biopsy. Kubegene is based on lightweight container technology and official standard algorithms. You can make a flexible and customizable gene sequencing process by using Kubegene.

Kubegene which running on the kubernetes makes gene sequencing simple and easy. It has the following characteristics:

* **Universal workflow design grammar**. Kubegene provides a complete set of gene sequencing workflow grammars which decouples with specific analysis tools. It requires a very low learning cost to learn how to write and use the workflow. You can easily migrate the gene sequencing business to kubegene.

* **More efficient resource usage**. Kubegene uses container to run the gene sequencing business. Compared to traditional gene sequencing solutions using virtual machines, kubegene makes resource usage more efficient and avoiding resource idle . 

* **Scaling based on demand**. kubernetes can automatically scale your cluster based on your gene sequencing workload by using Kubegene. Also you can easily scale the kubernetes cluster manually. It can save your production costs.

* **Tailor-made workflow for the biosequencing industry**. The workflow grammar is designed by comparing different genetic sequencing scenarios. It also keeps the user's traditional usage habit as much as possible and is closer to user scenarios.


## Components of Kubegene

Kugene has two main componments. Kbuedag is a workflow engine on Kubernetes platform.

* **Kubedag**. Kubedag is a DAG workflow engine on Kubernetes platform. It is dedicated to making Gene Sequencing workflow execute in container easily. For more information, you can see [Kugedag](https://kubegene.netlify.com/docs/about/kubedag/).

* **genectl**. genectl is a command line interface for running commands against Kubegene. For how to use genectl, you can see [genectl](https://kubegene.netlify.com/docs/guides/genectl-command/).

## Concepts of Kubegene

### Workflow
Definition of the gene sequencing process. The gene sequencing workflow includes the execution sequence information and the tool required for the sequencing process and the input data. The workflow consists of at least one tool. Each subtask in the workflow forms a data stream by its sequential relationship, and the pre-order subtask provides input for the subsequent subtask. For how to write your workflow, you can see [workflow grammar](https://kubegene.netlify.com/docs/guides/workflow-grammar/).

### Execution
A certain execution process of the gene sequencing workflow. When you submit a workflow, Kubegene will create a execution to execute the gene sequencing job. For more information, you can see [execution](https://kubegene.netlify.com/docs/about/execution/).

### Tool
Software tools which genetic sequencing use. It is a mirrored package of bioinformatics software. The tools can be programmed into the workflow in series or independently.
