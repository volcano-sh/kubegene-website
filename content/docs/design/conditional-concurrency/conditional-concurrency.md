+++
title = "Conditional Concurrency Requirement Overview and Design"
description = "This page provides an overview of `Conditional Concurrency Requirement`"
weight = 10
toc = true
aliases = ["/docs/design/"]
[menu.main]
  parent = "Design"
  weight = 2 
+++

In the genome sequencing flow, sometimes the if-else process of the sequencing step or even the switch-case process needs to be implemented according to the situation. 
For example, step A is used to judge whether the data is qualified. If the data is qualified, step B is performed, and if it is unqualified, step C is performed.

Kubegene needs conditional branching in various scenarios. Kubegene need to support static conditional branching and dynamic conditional branching.

So Kubegene needs if/else conditional branch and switch/case conditional branch handling.

# **Static conditional concurrency**
The so-called static is to determine which steps of the entire process do not need to be executed before running. 
For example, if the complete process is A->B, when the process is executed, it is determined that the condition of the B step is false, 
then the B step does not need to be executed.

If/else example:

In the following example, if the condition value of job-a is false, the commands command will not be executed.
```yaml
Inputs:
  Bool-var:
    Default: true
    Description: Boolean input variable
    Type: bool
Workflow:
  Job-a:
    tool: nginx:latest
    condition: ${bool-var} # <==Through the variable, decide whether the step is executed. Equivalent to if(bool-var)
    commands:
      - echo run-job-a
```

# **Dynamic conditional concurrency**
The so-called dynamic, that is, the process runs, and decide whether to perform the next step. This is the ability that many other existing processes (such as WDL) do not have.
Because it is necessary to judge whether the current condition value is true or false according to the "execution result" of another step, 
Kubegene uses the check_result function for dynamically conditional branch judging.


switch/case example:

In the following example, the result of job-a is 123, then job-b will execute, and job-c and job-d will not execute.

```yaml
Workflow:
  Job-a:
    tool: nginx:latest
    commands:
      - echo 123 # <== The execution result is reflected in std_out
  Job-b:
    tool: nginx:latest
    condition: check_result(job-a, "123") # <==Judge the result of job-a and decide whether the current step is executed. Equivalent to case(123)
    commands:
      - echo run-job-b
  Job-c:
    cool: nginx:latest
    condition: check_result(job-a, "121") # <==Just determine the result of job-a and decide whether the current step is executed. Equivalent to case(121)
    commands:
      - echo run-job-c
  Job-d:
    tool: nginx:latest
    condition: check_result(job-a, "333") # <==Judge the result of job-a and decide whether the current step is executed. Equivalent to case (333)
    commands:
      - echo run-job-d
      
```
The second parameter of check_result can refer to the inputs variable, such as:

condition: check_result(job-a, ${my_var})
See the check_result function for details.


## Dependency transfer
If the flow is A->B->C, when step B does not need to be executed because the condition is not satisfied, step C does not need to be executed 
regardless of whether the condition is true or not due to the dependency.

which is:

A -> B (when condition is false ) -> C ( automatically does not need to be executed )

So there is no need to write a condition condition for each job, and KubeGene will automatically determine the dependency transfer.


# **checkresult**


check_result is used to get the standard output of the "specified step" and determine whether it is equal to the specified string. Mainly used to control the conditional branch execution of the process.

The maximum output length currently supported is 1K bytes. If the stdout of the target step is exceeded, the execution will be terminated when the execution is executed (of course, you can adjust the command after the failure and then trigger the execution).
check_result is only available in the condition field.


grammar
```yaml
Job-a:
  tool: nginx:latest
  commands:
    - echo 111 # <== The output of a step
Job-b:
  condition: check_result(job-target, expect) # <==== Note here, determine if the output is equal
```

Example
Suppose the function of a target step "job-a" is to judge whether the sample data is qualified or not. The standard output is:

Ok
Through the check_result function, the next step execution branch is determined.

```yaml
Job-a:
  tool: nginx:latest
  commands:
    - echo ok # <== output of the step
Job-b:
  condition: check_result(job-a, "ok") # <==== The result here is, conditon: true
Job-c
  condition: check_result(job-a, "not_ok") # <==== The result here is, conditon: false
```

## High Level changes required for this feature


* Validations,instantiations etc for check_result  function in genectl 

* changes in the execution API to include the check_result related job like add Condition in the task

``` go

// job information.
type JobInfo struct {

    // conditional branch handling
    Condition interface{} `json:"condition,omitempty" yaml:"vars,omitempty"`
	
}

// Condition  conditional branch handling  in Task
type Condition struct {
    Condition interface{} `json:"condition,omitempty"`
}

// update the Task as below 
type Task struct {
	
    // Specifies the condition for this task
    // +optional
    Condition *Condition `json:"condition",omitempty"`

}
 ```
* if the condition is just bool then if it is true the job is created otherwise not created
 
* if the condition is check_result based then get the result from the depend job and validate the result with the expected value, if validation is success then do create the job otherwise do not create the job 

* get the result by using the log API and response data size limitation is 1K

```go

    //size limit 1k bytes extra 100 bytes added
	var sizeLimit int64
	sizeLimit = 1024 + 100

	opt := v1.PodLogOptions{LimitBytes: &sizeLimit, SinceTime: &metav1.Time{}}

	res, err := e.kubeClient.CoreV1().Pods(job.Namespace).GetLogs(podList.Items[0].Name,
		&opt).Stream() 
		
```

* just handle the graph changes as required
