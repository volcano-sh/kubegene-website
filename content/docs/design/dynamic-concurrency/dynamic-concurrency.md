+++
title = "Dynamic Concurrency Requirement Overview and Design"
description = "This page provides an overview of `Dynamic Concurrency Requirement`"
weight = 10
toc = true
aliases = ["/docs/design/"]
[menu.main]
  parent = "Design"
  weight = 2 
+++

# **Dynamic Concurrency feature in KubeGene**

In the process of genome sequencing, it is often necessary to read the contents of a file to control concurrent tasks, or to obtain the "output results" of another tasks. 
For example, after splitting the sample file by a fixed size, you need to get all the set of split file names. Or the previous step is distributed processing, and you need to get the sum of the results.


current KubeGene supports the static concurrency as shown below

```yaml
Job-2:
    command: echo ${1} ${2}
    vars_iter:
      - [A, B, C] # <==== Note here, the number of concurrent representations (range) for ${1}
      - [0, 1]

```
In the above KubeGene syntax, [A, B, C] indicates that the number of concurrency is three. The dynamic concurrency is that the value in the array [] is the result of the previous step. as follows:

```yaml
Job-2:
    command: echo ${1} ${2}
    vars_iter:
      - get_result(job-1) # <==== Note that the array result is dynamically "calculated" based on the stdout of the specified task(job-1).
      - [0, 1]

```

The result that is actually obtained is the standard output of the specified step.
For example, the stdout of job-1 is "1 2 3 4", then the result of the previous step is

    vars_iter:
      - ["1 2 3 4"]
Note that there is only one member in the array. That is, there is only one concurrency.
If we want, each one as a concurrency, you can add a "segment" to split . See the get_result function for the specific syntax .

    vars_iter:
      - get_result( job-1, " ")
Will get:

    vars_iter:
      - ["1", "2", "3", "4"] 
This gives four concurrencies, each variable being "1", "2", "3", "4". Among them, the separator is an arbitrary string.
After having the get_result function, if you need to implement the task concurrently and dynamically according to the result of the previous step



Example:

```yaml
Job-list:
    tool: nginx:new-latest
    commands: #   <== (1) List files in the directory
    - |
      For i in `ls ${sfs}/${output-folder}`; do
        Echo ${i}
      Done
Job-a:
 commands_iter:
    tool: nginx:new-latest
    command: echo ${output-prefix}job-c-${item} >> ${sfs}/${output-folder}/${1}; # <== (3) Iterative concurrency, replace variable $ {1}
    vars_iter:
      - get_result(job-list, '\n') # <== (2) Output the job-list, one by one, and split into concurrent arrays
  depends:
    - target: job-list
      type: whole

```


# **get_result  function** 

* get_result is only available in the vars_iter field.
* get_result (Job-target,Split)

Examples:

get_result( job-1)

get_result( job-1, "\n")  

get_result(job-target, ${input})


### Parameter Description

| Parameter           | Required |Type	  |	 Description                                                                                       |
|---------------------|----------|--------|--------------------------------------------------------------------------------------------------------|
| Job-target          | Yes      | string |	Specifies the target job name for the result   			                                   |
| Split               | No       | string |	The separator used to split the string.The Separator can be a single character or a string  or can be double quotes to indicate, such as "\n"  or can be variables such as ${input} |


Example
Suppose the standard output of a target step "job-1" is:

List-1.txt
List-2.txt
List-3.txt
List-4.txt
The iterative execution command is obtained by the get_result function.
```yaml
Job-a:
  commands_iter:
    command: echo ${1} ${2}
    vars_iter:
      - [A, B, C]
      - get_result( job-1, "\n") # <==== Note here, the number of concurrent representations (range) for ${2}
Then when you perform the job-a step, you will actually get the following results:

Job-a:
  commands_iter:
    command: echo ${1} ${2}
    vars_iter:
      - [A, B, C]
      - ["list-1.txt", "list-2.txt", "list-3.txt", "list-4.txt"] # <==== Note here, the concurrency of ${2} Quantity (range)
```

## High Level changes required for this feature


* Validations,instantiations etc for get_result  function in genectl 

* changes in the execution API to include the get_result related job like add command_iter in the task

``` go
// update the Task as below 
type Task struct {
	
	// CommandsIter defines batch command for workflows job.
	CommandsIter CommandsIter `json:"commands_iter,omitempty"`

}
 ```
* After getting the result from the depend job then need to update the  execution tasks by creating dynamic job 

* get the result by using the log API and response data size limitation is 1K

```go
res, err := e.kubeClient.CoreV1().Pods(job.Namespace).GetLogs(podList.Items[0].Name, nil).Param("limitBytes", SIZELIMIT).DoRaw() 
```

* after updating the  execution tasks need to update/recreate the graph

```go

// ExecutionUpdater is an interface used to update the ExecutionSpec associated with a Execution.
type ExecutionUpdater interface {
	UpdateExecution(modified *genev1alpha1.Execution, original *genev1alpha1.Execution) error
}

// NewExecutionUpdater returns a ExecutionUpdater that updates the Spec of a Execution,
// using the supplied client.
func NewExecutionUpdater(client geneclientset.ExecutionsGetter) ExecutionUpdater {
	return &executionUpdater{client}
}

type executionUpdater struct {
	execClient geneclientset.ExecutionsGetter
}
```

* need to update execution  with the latest changes in execution tasks

* may required to change the execution status
 
