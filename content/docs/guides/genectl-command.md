+++
title = "Genectl Command"
description = "Information on how to use genectl command"
weight = 10
toc = true
bref= "command."
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 2
+++

genectl is a command line interface for running commands against KubeGene. You can use genectl to submit your workflow and query the status of workflow execution.

## **genectl sub**
submit genome sequencing workflow to kube-dag controller to execute.

It contains three subcommand:

```
genectl sub job
genectl sub repjob
genectl sub workflow
```
### Global flags

| Flags               | Required | Description                                                                                                       |
|---------------------|----------|-------------------------------------------------------------------------------------------------------------------|
| --kubeconfig string | No       | cpu resource required to run this job, default 1 (default "1")                                                    |
| --tool-repo string  | Yes      | directory or URL to tool repository, if it is a URL, it must point to tool file. (default "/root/kubegene/tools") |
| --dry-run           | No       | If true, display instantiate execution but do not submit                                                               |

## **genectl sub job**
sub job command submits a job which execute a single shell script when perform genome sequencing. You should upload the shell script and sample data to the volume used by this job in preparation stage.

### Usage
genectl sub job FILENAME [flags]
The args FILENAME is the absolute path of the shell script within the container.

### Flags
| Flags               | Required | Description                                                                                                                                         |
|---------------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| --cpu string        | No       | cpu resource required to run this job, default 1 (default "1")                                                                                      |
| --memory string     | No       | memory resource required to run this job, default 1G                                                                                                |
| --mount-path string | No       | path within the container at which the volume that pvc refer to should be mounted, default the same as the host dir of job script.                  |
| --pvc string        | Yes      | the name of pvc that used by the job, the backend storage that the pvc refer to should pre-populate the job script and sample data used by the job. |
| --shell string      | No       | linux shell used to execute the job script, default sh.                                                                                             |
| --tool string       | Yes      | tool used by the job, format: toolName:toolVersion                                                                                                  |

### example
```
genectl sub job /kubegene/bwa_help.sh --memory 1g --cpu 1 --tool bwa:0.7.12 --pvc pvc-gene
```
## **genectl sub repjob**
sub repjob command submits a group of job.
### Usage
genectl sub repjob FILENAME [flags]
The args[0] FILENAME is the absolute path of the shell script within the container. And every line in the shell script is a single job and it follow the format:
```
	bash/sh             scriptPath                args...
	  |                    |                        |
	linux shell  abs path within the container    shell args
```
for example:

```
genectl sub repjob /kubegene/bwa_mem_work.sh --memory 1g --cpu 1 --tool bwa:0.7.12 --pvc pvc-gene
```
/kubegene/bwa_mem_work.sh is the the absolute path of the shell script within the container.

the content of bwa_mem_work.sh:

```
	sh /kubegene/bwa_mem.sh obs/path/sample1.fastq.gz obs/path/hg19.fa >obs/path/sample1.sam
	sh /kubegene/bwa_mem.sh obs/path/sample2.fastq.gz obs/path/hg19.fa >obs/path/sample2.sam
```
And the script path in your host should keep consistent with the path within the container. You should upload all the shell script and sample data that will be used to the storage volume used by this job.

### Flags
| Flags               | Required | Description                                                                                                                                         |
|---------------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| --cpu string        | No       | cpu resource required to run this job, default 1                                                                                                    |
| --memory string     | No       | memory resource required to run this job, default 1G                                                                                                |
| --mount-path string | No       | path within the container at which the volume that pvc refer to should be mounted, default the same as the host dir of job script.                  |
| --pvc string        | Yes      | the name of pvc that used by the job, the backend storage that the pvc refer to should pre-populate the job script and sample data used by the job. |
| --shell string      | No       | linux shell used to execute the job script, default sh (default "sh")                                                                               |
| --tool string       | Yes      | tool used by the job, format: toolName:toolVersion                                                                                                  |

### example
```
genectl sub repjob /kubegene/bwa_mem_work.sh --memory 1g --cpu 1 --tool bwa:0.7.12 --pvc pvc-gene
```
## **genectl sub workflow**
Submit a workflow from a file with specify input json.
### Usage
genectl sub workflow FILENAME [flag]

| Flags          | Required | Description                                                                                                                                                                                                                                  |
|-----------|----------|-----------------|
| --input string | No       | the input json file path. If you do not want to use the input fields defined by the workflow YAML file, you can specify a json file to override this field here. When using a new sample, you can define a new sample name in the json file. |


### example
```
gcs sub workflow wf.yaml --input UserInputs.json
```

## **genectl describe**
Query the detail execution status of a workflow.
### Usage
genectl describe executionName [flags]
### Flags
| Flags              | Required | Description                               |
|--------------------|----------|-------------------------------------------|
| --namespace string | No       | namespace of execution(default "default") |

### example
```
genectl describe my-exec –n gene-system
```

## **genectl get**
Query one or a list of execution status of workflows.
### Usage
genectl get [flags]
### Flags
| Flags              | Required | Description                                                                                                                                               |
|--------------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| --namespace string | No       | namespace of execution(default "default")                                                                                                                 |
| --phase strings    | No       | A comma-separated list for workflow execution phase. Available values: [Running Succeeded Failed Error]. This flag unset means 'list all phase execution' |
| --all-namespaces   | No       | If present, list execution across all namespaces.                                                                                                         |

### example
List executions in default namespace and in default output format.
```
genectl get execution
```
List all executions in default output format.
```		
genectl get execution --all-namespaces
```
List all executions in yaml output format.
```		
genectl get execution --all-namespaces -o yaml
```
List executions that are running or succeeded in yaml output format.
```	
genectl get execution --all-namespaces –phase Running,Succeeded
```
List executions in exec-system namespace that are running or succeeded in yaml output format.
```		
genectl get execution -n exec-system –phase Running,Succeeded
```
## **genectl del**
delete an execution of workflow
### Usage
genectl del executionName [flags]
### Flags
| Flags              | Required | Description                               |
|--------------------|----------|-------------------------------------------|
| --namespace string | No       | namespace of execution(default "default") |

### example
```
genectl del my-exec –n gene-system
```
