# inputs

Optional. Used to define the variable of the gene sequencing process. Inputs consists of multiple variables, up to 60 variables can be defined, and each variable name is unique. If the variable name is repeated, the latter definition will override the previously defined one. Variables can be referenced in other parts of the workflow file, using the shell script format, ie: ${var}, where var is the variable name.


### inputs format

```
inputs:
  <var name>:
    type: <type>
    default: <default value>
    value: <value>
    description：<description>
```

## Field

| Field       | required | type        | description                                                                                                                                                                                                        |
|-------------|----------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| var name    | Yes      | String      | The var name consists of letters, numbers, and underscores "-" or underscores "_" with a length of [1, 20].                                                                                                        |
| type        | Yes      | String      | Parameter type, allowed type is as follows:<br> <ul><li>string</li> <li>number</li> <li>bool</li> <li>array</li> </ul>The default type is “string” if not specified.                                                                                              |
| default     | No       | interface{} | The default value of the parameter, fill in the corresponding value according to type. Note：the type of default and value must be consistent with the type field.                                                  |
| value       | No       | interface{} | Value is the literal value to use for the parameter. The value can be overwrite by an external input.Precedence: external input value > default. Note: one of external input, value and default must be specified. |
| description | No       | String      | Parameter description information, must be no more than 255 characters.                                                                                                                                         |

## Inputs example

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