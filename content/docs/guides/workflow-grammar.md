+++
title = "Workflow Grammar"
description = "Information on how to use and write workflow yaml"
weight = 10
toc = true
bref= "command."
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 1
+++

We have defined a complete set of genome sequencing workflow grammars. It keeps the user's traditional usage habit as much as possible. It requires a very low learning cost to learn how to write and use the workflow. If you have other type workflow grammar files, you need to convert them to the gene container syntax first.

## **Template structure**

| field    | required | type   | description                                                                                                        |
|----------|----------|--------|--------------------------------------------------------------------------------------------------------------------|
| version  | Yes      | string | The version of workflow                                                                                            |
| inputs   | No       | Map    | The variable of the workflow. You can define multiple, set the actual values of these variables at execution time. |
| workflow | Yes      | Map    | Define the steps in the workflow and the dependencies between the steps.                                           |
| volumes  | No       | Map    | Define the used claim and mount path of the shared storage for the workflow steps.                                 |



## **workflow example**

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
    commands_iter:
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
    mount_path: /result
    mount_from:
      pvc: test-pvc # claimName
```

## **version**

The version of gene workflow, the current support is genecontainer_0_1.

## **inputs**

Optional. Used to define the variable of the genome sequencing process. Inputs consists of multiple variables, up to 60 variables can be defined, and each variable name is unique. If the variable name is repeated, the latter definition will override the previously defined one. Variables can be referenced in other parts of the workflow file, using the shell script format, ie: ${var}, where var is the variable name.


### inputs format

```
inputs:
  <var name>:
    type: <type>
    default: <default value>
    value: <value>
    description：<description>
```

### Field

| Field       | required | type        | description                                                                                                                                                                                                        |
|-------------|----------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| var name    | Yes      | String      | The var name consists of letters, numbers, and underscores "-" or underscores "_" with a length of [1, 20].                                                                                                        |
| type        | Yes      | String      | Parameter type, allowed type is as follows:<br> <ul><li>string</li> <li>number</li> <li>bool</li> <li>array</li> </ul>The default type is “string” if not specified.                                                                                              |
| default     | No       | interface{} | The default value of the parameter, fill in the corresponding value according to type. Note：the type of default and value must be consistent with the type field.                                                  |
| value       | No       | interface{} | Value is the literal value to use for the parameter. The value can be overwrite by an external input.Precedence: external input value > default. Note: one of external input, value and default must be specified. |
| description | No       | String      | Parameter description information, must be no more than 255 characters.                                                                                                                                         |

### Inputs example

```
inputs:
  sample:
    type: string
    default: /home/root
  split:
    default: 3
    type: number
description: parallel number
chromlist:
  default:
    - 'chr1:10000-103863906'
    - 'chr1:103913906-205922707'
    - 'chr1:206072707-249240621'
    - 'chr2:10000-87668206'
    - 'chr2:87718206-149690582'
    - 'chr2:149790582-243189373'
    - 'chr3:10000-90504854'
   type: array
flag:
  value: true
  type: bool
```

## **Workflow**

Required. Define the tasks involved in the process and the dependencies between the tasks. 
A workflow consists of multiple steps, each of which can perform a specific task.

### workflow format

```
workflow:
  <task name>:
    description: <task info>
    tool: <toolName:version>
    resources：<resources required by the workflow>
    commands：<commands>
    commands_iter: <commands with variable>
    depends：< dependent task>
```

### Field

#### workflow field

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>task name</td>
      <td>Yes</td>
      <td>String   </td>
      <td>The name of task. It must consist of lower case alphanumeric characters or '-', 
      and must start and end with an alphanumeric character. And must be no more than 
      40 characters.</td>
   </tr>
   <tr>
      <td>description</td>
      <td>No</td>
      <td>String   </td>
      <td>Information about this task, must be no more than 255 characters.</td>
   </tr>
   <tr>
      <td>tool</td>
      <td>Yes</td>
      <td>String   </td>
      <td>The tool and version of the current task required, format "toolName:toolVersion". 
      For example, the bwa of version 0.7.12 is set to "tool: bwa:0.7.12".</td>
   </tr>
   <tr>
      <td>resources</td>
      <td>No</td>
      <td>struct</td>
      <td>Compute Resources required by the task.</td>
   </tr>
   <tr>
      <td>commands</td>
      <td>No</td>
      <td>array[string]</td>
      <td>The command executed in the container. The length of the array indicates 
      the number of concurrent. Each member represents a command executed in a 
      container.In the following example, if there are four lines in the command, 
      the number of concurrent containers is 4, and each container executes a 
      different command.
         <pre>
commands:
  - sh /obs/shell/run-xxx/run.sh 1 a 
  - sh /obs/shell/run-xxx/run.sh 2 a 
  - sh /obs/shell/run-xxx/run.sh 1 b 
  - sh /obs/shell/run-xxx/run.sh 2 b </pre>
  Note 1: One of field commands and commands_iter must be set. Use commands_iter if the commands 
  need to pass variables, otherwise use commands.
      </td>
   </tr>
   <tr>
      <td>commands_iter</td>
      <td>No</td>
      <td>Struct</td>
      <td>The command executed in the container, and the difference between command and commandIter 
      is that the commandIter supports shell scripts with variables.</td>
   </tr>
   <tr>
      <td>depends</td>
      <td>No</td>
      <td>array[Struct]</td>
      <td>Specify other tasks that the current task depends on.</td>
   </tr>
</table>

#### resource field description

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>memory</td>
      <td>No</td>
      <td>String</td>
      <td>
         The amount of memory resources required, in G. Format: "Number + Unit".
         <ul>
            <li>The number can be decimals.</li>
            <li>The unit is G or g.</li>
         </ul>
         For example, if the memory size required is 4G, you can fill in "4G" or "4g" here.
      </td>
   </tr>
   <tr>
      <td>cpu</td>
      <td>No</td>
      <td>String</td>
      <td>The amount of memory resources required, in C. Format: "Number 
      + Unit".
      <ul>
      <li>The number can be decimals.</li>
      <li>The unit is C or c.</li></td>
   </tr>
</table>


#### commands_iter field description

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>command</td>
      <td>Yes</td>
      <td>String</td>
      <td>A shell script with variables, for example:
      <pre>echo ${1} ${2} ${item} </pre>
There are two ways to define variables: 
<ul>
<li>
Variable parameters defined in commands_iter, including vars and vars_iter parameters,
 you can set one of them. The format is "${n}", n is a positive integer starting at 1. 
 And it will be replaced by the nth element of vars. The vars needs to list all 
 combinations of parameter, and the vars_iter is an automatic traversal combination of parameter.
</li>
<li>Built-in variable "${item}". Represent the order number of all the possible parameter combinations.
</li>
</ul>
For example:
<pre>
    commands_iter: 
      command: echo ${1} ${item} 
        vars:
          - a
          - b 
          - c </pre>
 
Then the final command will be:
<pre>
    - echo a 0 
    - echo b 1 
    - echo c 2
</pre>
</td>
   </tr>
   <tr>
      <td>vars</td>
      <td>No</td>
      <td>array[array] </td>
      <td>A two-dimensional array, will be used to replace the command variable, represent all the possible parameter combinations. 
      <ul>
      <li>
      In the two-dimensional array, the members of each row represent the variables 
      <code>${1}</code>, <code>${2}</code>, <code>${3}</code> in the command. <code>${1}</code> represents the first member of each line. <code>${2}</code> represents the second member of each line. And <code>${3}</code> represents the third member of each line.
      </li>
      <li>
      The length of the two-dimensional array indicates that how many times the command 
      will be executed with different parameters. Each line of the array is used to 
      instantiate the command.The number of rows in the array is the number of k8s job that will run.
      </li>
      </ul>
      For example, the vars has four lines.
      <pre>
command: echo ${1} ${2} ${item} 
vars: 
  - [0, 0] # 0 -> ${1}; 0 -> ${2}; 0 -> ${item}
  - [0, 1] # 0 -> ${1}; 1 -> ${2}; 1 -> ${item} 
  - [1, 0] # 1 -> ${1}; 0 -> ${2}; 2 -> ${item} 
  - [1, 1] # 1 -> ${1}; 1 -> ${2}; 3 -> ${item} </pre>

4 k8s jobs will run to execute the commands.<br>
For the 1st job, the command is:
<pre>
echo 0 0 0 
</pre>
For the 2nd job, the command is: 
<pre>
echo 0 1 1 
</pre>
For the 3rd job, the command is:
<pre>
echo 1 0 2 
</pre>
For the 4th job, the command is: 
<pre>
echo 1 1 3
</pre>
</td>
   </tr>
   <tr>
      <td>vars_iter</td>
      <td>No</td>
      <td>array[array]</td>
      <td>A two-dimensional array. vars_iter list all the possible parameters for every position 
      in the command line. And we will use algorithm Of Full Permutation to generate all the 
      permutation and combinations for these parameter that will be used to replace the <code>${n}</code> variable.
      The first row member of the array replace the variable <code>${1}</code> in the command,
      the second row member replace the variable <code>${2}</code> in the command, and so on.
      For example,
<pre>
commands_iter:
  command: sh /tmp/step1.splitfq.sh ${1} ${2} ${3}
  vars_iter: - ["sample1", "sample2"]
            - [0, 1]
            - [25]
</pre>
then the final command will be:
<pre>
sh /tmp/scripts/step1.splitfq.sh sample1 0 25 
sh /tmp/scripts/step1.splitfq.sh sample2 0 25 
sh /tmp/scripts/step1.splitfq.sh sample1 1 25 
sh /tmp/scripts/step1.splitfq.sh sample2 1 25 
</pre>
If there are many array members per line, you can use the range function.
The format of range function:
<pre>range(start, end, step)</pre>
 Start and end are all integer. And step can only be positive integer. 
 If you do not specify step, the default is 1.

Range(1, 4) represents array [1,2,3]
Range(1, 10, 2) represents array [1, 3, 5, 7, 9] 
<pre>
vars_iter: - range(0, 4)
</pre>the same as: 
<pre>
vars_iter: - [0, 1, 2, 3]
</pre>
</td>
   </tr>
</table>

#### Depends field description

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>target</td>
      <td>Yes</td>
      <td>String</td>
      <td>The name of the task relying on, make sure that the specified task name must exist in the workflow.</td>
   </tr>
   <tr>
      <td>type</td>
      <td>No</td>
      <td>String</td>
      <td>Dependency type, the value can be:
      <ul>
      <li>
      whole, overall dependencies, this is the default.
      </li>
      <li>
      iterate, iterative dependency.
      </li>
      </ul>
      For example, if both task A and task B are executed concurrently 100.
    <ul>
    <li>
     Setting “whole” indicates that task B can start execution after all 100 steps of task A finished.
     </li>
     <li>
     Setting “iterate” means that the 1st step of task A is completed, then the 1st step of the task B 
     can start execution. Iterative execution can improve the overall concurrency efficiency.
     </li>
     </ul>
     </td>
   </tr>
</table>


### Workflow example

```
workflow:
  job-a:
    tool: nginx:latest
    resources:
      memory: 2G
      cpu: 1c
    commands:
      - sleep `expr 3 \* ${wait-base}`; echo ${output-prefix}job-a | tee -a ${obs}/${output}/${result};
  job-b:
    tool: nginx:latest
    commands_iter:
      command: sleep `expr ${1} \* ${wait-base}`; echo ${output-prefix}job-b-${item} | tee -a ${obs}/${output}/${result};
      vars_iter:
        - range(0, 3)
    depends:
      - target: job-a
        type: whole
  job-c:
    tool: nginx:latest
    type: GCS.Job
    resources:
      memory: 8G
      cpu: 2c
    commands_iter:
      command: sleep `expr ${1} \* ${wait-base}`; echo ${output-prefix}job-c-${item} | tee -a ${obs}/${output}/${result};
      vars_iter:
        - [3, 20]
    depends:
      - target: job-a
        type: iterate
      - target: job-b
```

## **volumes**

Optional, information about volume that required genome sequencing process required, such as mount path in container, pvc name and so on. You can specify multiple volumes.

### Volumes format

```
volumes: 
  <volume name>:
    mount_path: <mount_path>
    mount_from: <pvc info>
```

### field

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>mount_path</td>
      <td>Yes</td>
      <td>string</td>
      <td>Path within the container at which the volume should be mounted. Must not contain ':'.</td>
   </tr>
   <tr>
      <td>mount_from</td>
      <td>Yes</td>
      <td>struct</td>
      <td>Detail info about volume</td>
   </tr>
</table>


mount_from field description

<table>
   <tr>
      <td>field</td>
      <td>required</td>
      <td>type</td>
      <td>description</td>
   </tr>
   <tr>
      <td>pvc</td>
      <td>Yes</td>
      <td>string</td>
      <td>The pvc name. Note: the specified pvc must exist in the cluster.</td>
   </tr>
</table>


### volumes example
```
volumes:
  genref:
    mount_path: ${volume-path-ref}
    mount_from:
      pvc: ${my_k8s_pvc}
  genobs:
    mount_path: /volume-path-obs
    mount_from:
      pvc: sample-data-pvc
```
