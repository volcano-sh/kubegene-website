+++
title = "KubeGene Overview"
description = "This page provides an overview of `KubeGene`"
weight = 10
toc = true
aliases = ["/docs/about/"]
[menu.main]
  parent = "What is KubeGene?"
  weight = 1
+++

## What is KubeGene

KubeGene is dedicated to making genome sequencing process simple, portable and scalable. It provides a complete solution for genome sequencing on the kubernetes and supports mainstream biological genome sequencing scenarios such as DNA, RNA, and liquid biopsy. KubeGene is based on lightweight container technology and official standard algorithms. You can make a flexible and customizable genome sequencing process by using KubeGene.

KubeGene which running on the kubernetes makes genome sequencing simple and easy. It has the following characteristics:

* **Universal workflow design grammar**. KubeGene provides a complete set of genome sequencing workflow grammars which decouples with specific analysis tools. It requires a very low learning cost to learn how to write and use the workflow. You can easily migrate the genome sequencing business to KubeGene.

* **Tailor-made workflow for the biosequencing industry**. The workflow grammar is designed by comparing different genome sequencing scenarios. It also keeps the user's traditional usage habit as much as possible and is closer to user scenarios.

* **More efficient resource usage**. KubeGene uses container to run the genome sequencing business. Compared to traditional genome sequencing solutions using virtual machines, KubeGene makes resource usage more efficient and avoiding resource idle .

* **Scaling based on demand**. kubernetes can automatically scale your cluster based on your genome sequencing workload by using KubeGene. Also you can easily scale the kubernetes cluster manually. It can save your production costs.

## Components of KubeGene

KubeGene has two main componments.

* **KubeDag**. KubeDag is a DAG workflow engine on Kubernetes platform. It is dedicated to making Gene Sequencing workflow execute in container easily. For more information, refer to [KubeDag](/docs/about/kubedag/).

* **genectl**. genectl is a command line interface for running commands against KubeGene. For how to use genectl, refer to [genectl](/docs/guides/genectl-command/).

## Concepts of KubeGene

### Workflow
Definition of the genome sequencing process. The genome sequencing workflow includes the execution sequence information and the tool required for the sequencing process and the input data. The workflow consists of at least one tool. Each subtask in the workflow forms a data stream by its sequential relationship, and the pre-order subtask provides input for the subsequent subtask. For how to write your workflow, refer to [workflow grammar](/docs/guides/workflow-grammar/).

### Execution
A certain execution process of the genome sequencing workflow. When you submit a workflow, KubeGene will create a execution to execute the genome sequencing job. For more information, refer to [execution](/docs/about/execution/).

### Tool
Software tools which genome sequencing use. It is a mirrored package of bioinformatics software. The tools can be programmed into the workflow in series or independently. For more information, refer to [Tool](/docs/guides/tool/)
