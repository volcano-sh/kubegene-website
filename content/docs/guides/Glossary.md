+++
title = "Glossary"
description = "Glossary"
weight = 10
toc = true
bref= "workflow grammar."
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 1
+++



# Workflow
Definition of the gene sequencing process. The gene sequencing workflow includes the execution sequence information and the tools required for the sequencing process and the input data. The workflow consists of at least one tool. Each subtask in the workflow forms a data stream by its sequential relationship, and the pre-order subtask provides input for the subsequent subtask.

# Execution
A certain execution process of the gene sequencing workflow.

# Tool
Software tools which genetic sequencing use. It is a mirrored package of bioinformatics software. The tools can be programmed into the workflow in series or independently.

A tool instance may looks like this:
name: gatk
version: 4.0.9.0
image: gatk:4.0.9.0
cpu: 2c
memory: 2G
description: gatk

## tool field

| field       | required | type   | description                                                                                                                                                                                      |
|-------------|----------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name        | Yes      | string | Name of tool, it must consist of lower case alphanumeric characters or '-', start with an alphabetic character, and end with an alphanumeric character. The max length of name is 60 characters. |
| version     | Yes      | string | Version of tool.                                                                                                                                                                                 |
| image       | Yes      | string | Docker image name.                                                                                                                                                                               |
| cpu         | No       | string | cpu is the recommended cpu resources request for this tool to run. If we do not specify cpu request in the workflow, this value will be used.                                                    |
| memory      | No       | string | memory is the recommended memory resources request for this tool to run. If we do not specify memory request in the workflow, this value will be used.                                           |
| command     | No       | string | The recommended command to use this tool.                                                                                                                                                        |
| description | No       | string | Information about this tool and instructions info for use. The max length of name is 255 characters.   
