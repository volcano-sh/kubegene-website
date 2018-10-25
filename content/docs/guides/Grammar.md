+++
title = "Grammar"
description = "Grammar"
weight = 10
toc = true
bref= "workflow grammar."
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 1
+++

We have defined a complete set of gene sequencing workflow grammars. It keeps the user's traditional usage habit as much as possible. It requires a very low learning cost to learn how to write and use the workflow. If you have other type workflow grammar files, you need to convert them to the gene container syntax first.

### Template structure

| field    | required | type   | description                                                                                                        |
|----------|----------|--------|--------------------------------------------------------------------------------------------------------------------|
| version  | Yes      | string | The version of workflow                                                                                            |
| inputs   | No       | Map    | The variable of the workflow. You can define multiple, set the actual values of these variables at execution time. |
| workflow | Yes      | Map    | Define the steps in the workflow and the dependencies between the steps.                                           |
| volumes  | No       | Map    | Define the used claim and mount path of the shared storage for the workflow steps.                                 |



### Template example

```
version: genecontainer_0_1
inputs: # workflow variant
  sample: # variant name
    default: sample1
    type: string
    description: variant description
workflow:
  test-job-a:
    tool: busybox:latest
    resources:
      memory: 4G
      cpu: 4C
    commandsIter:
      command: sleep 10; touch /result/test-job/${sample}.${item}.${1}.txt 
      vars_iter:
        - [0,1]
  test-job-b:
    tool: busybox:latest
    resources:
      memory: 4G
      cpu: 4C
    commands:
      - sleep 10; touch /result/test-job/${sample}.job-b.txt
    depends:
      - target: test-job-a
volumes: 
  genobs:
    mountPath: /result
    mountFrom:
      pvc: test-pvc # claimName
```
