+++
title = "Generic Conditional Concurrency Requirement Overview and Design"
description = "This page provides an overview of `Generic Conditional Concurrency Requirement`"
weight = 10
toc = true
aliases = ["/docs/design/"]
[menu.main]
  parent = "Design"
  weight = 2 
+++

In the genome sequencing flow, sometimes the if-else process of the sequencing step or even the switch-case process needs to be implemented according to the situation. 
For example, step A is used to judge whether the data is qualified. If the data is qualified, step B is performed, and if it is unqualified, step C is performed.

Kubegene needs generic conditional branching in various scenarios. Kubegene need to dynamic generic conditional branching.

So Kubegene needs if/else conditional branch and switch/case generic conditional branch handling.

# **Generic Conditional Dynamic Concurrency**

The check_result based support is limited for just matching for equality, this generic condition will support the
different operators like In,Equal,NotIn,Exists etc so this feature is super set of check_result.

The so-called dynamic, that is, the process runs, and decide whether to perform the next step. This is the ability that many other existing processes (such as WDL) do not have.
Because it is necessary to judge whether the current condition value is true or false according to the "execution result" of another step, 
Kubegene uses the generic condition feature for dynamically conditional branch judging.


switch/case example:

In the following example, the result of job-a is testkey:testvalue, then job-b will execute, and job-c and job-d will not execute.

```yaml
Workflow:
  Job-a:
    tool: nginx:latest
    commands:
      - echo testkey:testvalue # <== The execution result is reflected in std_out
  Job-b:
    tool: nginx:latest
    genericcondition:         # <==Judge the result of job-a and decide whether the current step is executed.
      dependjobname: Job-a    # <==The given one of the matching rule should satisfy the job-a result which should be list of key:value  separated by ,
      matchrules:
        - key: testkey
          operator: In
          values:
           - testvalue
    commands:
      - echo run-job-b
  Job-c:
    cool: nginx:latest
    genericcondition: 
       dependjobname: Job-a
       matchrules:
        - key: testscenarioinv
          operator: Exists
    commands:
      - echo run-job-c
  Job-d:
    tool: nginx:latest
    genericcondition: 
       dependjobname: Job-a
       matchrules:
         - key: testkey
           operator: DoesNotExists
    commands:
      - echo run-job-d
      
```

## High Level changes required for this feature


* Validations,conversions etc for generic condition  in genectl 

* changes in the execution API to include the generic condition in the task

* Validations in the controller side to support directly in execution

* The below changes are related to client side in genectl

``` go

// job information.
type JobInfo struct {
    // generic conditional handling using the match requirements are ORed.
    GenericCondition *GenericCondition `json:"genericcondition,omitempty" yaml:"genericcondition,omitempty"`
}

// A match  operator is the set of operators that can be used in
// a MatchRequirement.
type MatchOperator string

const (
    MatchOperatorOpIn           MatchOperator = "In"
    MatchOperatorOpNotIn        MatchOperator = "NotIn"
    MatchOperatorOpExists       MatchOperator = "Exists"
    MatchOperatorOpDoesNotExist MatchOperator = "DoesNotExist"
    MatchOperatorOpGt           MatchOperator = "Gt"
    MatchOperatorOpLt           MatchOperator = "Lt"
    MatchOperatorOpEqual        MatchOperator = "="
    MatchOperatorOpNotEqual     MatchOperator = "!="
    MatchOperatorOpDoubleEqual  MatchOperator = "=="
)

// A matching rule is a requirement that contains values, a key, and an operator
// that relates the key and values.
type MatchRule struct {
    // The key that the requirement applies to.
    Key string `json:"key" yaml:"key"`
    // Represents a key's relationship to a set of values.
    // Valid operators are In, NotIn, Exists, DoesNotExist. Gt, and Lt.
    Operator MatchOperator `json:"operator" yaml:"operator"`
    // An array of string values. If the operator is In or NotIn,
    // the values array must be non-empty. If the operator is Exists or DoesNotExist,
    // the values array must be empty. If the operator is Gt or Lt, the values
    // array must have a single element, which will be interpreted as an integer.
    // +optional
    Values []string `json:"values,omitempty" yaml:"values,omitempty"`
}

// generic Conditional dynamic handling match rules are ORed.
type GenericCondition struct {
    DependJobName string      `json:"dependjobname" yaml:"dependjobname"`
    MatchRules    []MatchRule `json:"matchrules" yaml:"matchrules"`
}

```

* The below changes are related to controller side in kube-dag

``` go
// update the Task as below 
type Task struct {
    // Specifies the generic condition for this task
    // The task will be executed only when any one of the rule of condition is  satisfied
    // +optional
    GenericCondition *GenericCondition `json:"genericcondition,omitempty"`
}

// A match  operator is the set of operators that can be used in
// a MatchRule.
type MatchOperator string

const (
    MatchOperatorOpIn           MatchOperator = "In"
    MatchOperatorOpNotIn        MatchOperator = "NotIn"
    MatchOperatorOpExists       MatchOperator = "Exists"
    MatchOperatorOpDoesNotExist MatchOperator = "DoesNotExist"
    MatchOperatorOpGt           MatchOperator = "Gt"
    MatchOperatorOpLt           MatchOperator = "Lt"
    MatchOperatorOpEqual        MatchOperator = "="
    MatchOperatorOpNotEqual     MatchOperator = "!="
    MatchOperatorOpDoubleEqual  MatchOperator = "=="
)

// A matching rules is a requirement that contains values, a key, and an operator
// that relates the key and values.
type MatchRule struct {
    // The key that the requirement applies to.
    Key string `json:"key"`
    // Represents a key's relationship to a set of values.
    // Valid operators are In, NotIn, Exists, DoesNotExist. Gt, and Lt.
    Operator MatchOperator `json:"operator"`
    // An array of string values. If the operator is In or NotIn,
    // the values array must be non-empty. If the operator is Exists or DoesNotExist,
    // the values array must be empty. If the operator is Gt or Lt, the values
    // array must have a single element, which will be interpreted as an integer.
    // +optional
    Values []string `json:"values,omitempty"`
}

// generic Conditional dynamic handling match rules are ORed.
type GenericCondition struct {
    DependJobName string      `json:"dependjobname"`
    MatchRules    []MatchRule `json:"matchrules"`
}

 ```

* get the result by using the log API and response data size limitation is 1K

```go
    // size limit 1k bytes extra 100 bytes added
    var sizeLimit int64
    sizeLimit = 1024 + 100
    opt := v1.PodLogOptions{LimitBytes: &sizeLimit, SinceTime: &metav1.Time{}}
    res, err := e.kubeClient.CoreV1().Pods(job.Namespace).GetLogs(podList.Items[0].Name,
        &opt).Stream()        
```

* the Match Rules matching the key ,value list of dependency job result as below with different operators

```go

func RuleSatisfied(r genev1alpha1.MatchRule, kv map[string]string) bool {

    switch r.Operator {
    case genev1alpha1.MatchOperatorOpIn, genev1alpha1.MatchOperatorOpEqual, 
    genev1alpha1.MatchOperatorOpDoubleEqual:
        val, ok := kv[r.Key]
        if !ok {
            return false
        }
        return hasValue(r, val)
    case genev1alpha1.MatchOperatorOpNotIn, genev1alpha1.MatchOperatorOpNotEqual:
        val, ok := kv[r.Key]
        if !ok {
            return true
        }
        return !hasValue(r, val)
    case genev1alpha1.MatchOperatorOpExists:
        _, ok := kv[r.Key]
        return ok
    case genev1alpha1.MatchOperatorOpDoesNotExist:
        _, ok := kv[r.Key]
        return !ok
    case genev1alpha1.MatchOperatorOpGt, genev1alpha1.MatchOperatorOpLt:
        val, ok := kv[r.Key]
        if !ok {
            return false
        }
        lsValue, err := strconv.ParseInt(val, 10, 64)
        if err != nil {
            glog.V(2).Infof("ParseInt failed for value %+v in key &val %+v, %+v", val, kv, err)
            return false
        }

        // There should be only one strValue in r.Values, and can be converted to a integer.
        if len(r.Values) != 1 {
            glog.V(2).Infof("Invalid values count %+v of match rule %#v, for 'Gt', 'Lt' operators, exactly one value is required", len(r.Values), r)
            return false
        }

        var rValue int64
        for i := range r.Values {
            rValue, err = strconv.ParseInt(r.Values[i], 10, 64)
            if err != nil {
                glog.V(2).Infof("ParseInt failed for value %+v in matchrule %#v, for 'Gt', 'Lt' operators, the value must be an integer", r.Values[i], r)
                return false
                }
        }
        return (r.Operator == genev1alpha1.MatchOperatorOpGt && lsValue > rValue) || 
        (r.Operator == genev1alpha1.MatchOperatorOpLt && lsValue < rValue)
    default:
        return false
    }
}

```
* just handle the graph changes as required
