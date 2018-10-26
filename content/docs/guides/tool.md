+++
title = "Tool overview"
description = "Information on how to use and write tool yaml"
weight = 10
toc = true
bref= "command."
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 1
+++

## Tool overview

Software tools which gene sequencing use. It is a mirrored package of bioinformatics software. The tools can be programmed into the workflow in series or independently. We will provide a number of public tools, and users can create custom tools. All the tools are stored in a sequencing tool repository.


## Tool format
```
name: <name>
version: <version>
image: <image>
cpu: <cpu>
memory: <memory>
description: <info>
```

## Tool field

| field       | required | type   | description                                                                                                                                                                                      |
|-------------|----------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name        | Yes      | string | Name of tool, it must consist of lower case alphanumeric characters or '-', start with an alphabetic character, and end with an alphanumeric character. The max length of name is 60 characters. |
| version     | Yes      | string | Version of tool.                                                                                                                                                                                 |
| image       | Yes      | string | Docker image name.                                                                                                                                                                               |
| cpu         | No       | string | cpu is the recommended cpu resources request for this tool to run. If we do not specify cpu request in the workflow, this value will be used. Format: "Number + Unit".<ul><li>The number can be decimals.</li><li>The unit is C or c.</li></ul>For example, if the cpu required is 4C, you can fill in "4c" or "4C" here.                                                   |
| memory      | No       | string | memory is the recommended memory resources request for this tool to run. If we do not specify memory request in the workflow, this value will be used. Format: "Number + Unit".<ul><li>The number can be decimals.</li><li>The unit is G or g.</li></ul>For example, if the memory size required is 4G, you can fill in "4G" or "4g" here.                                          |
| command     | No       | string | The recommended command to use this tool.                                                                                                                                                        |
| description | No       | string | Information about this tool and instructions info for use. The max length of name is 255 characters.   


## Tool instance
A tool instance looks like this:
```
name: gatk
version: 4.0.9.0
image: gatk:4.0.9.0
cpu: 2c
memory: 2G
description: gatk
```

## How to use tool
When you use genectl to submit gene sequencing workflow, you can specify what tool to use, the format is: "toolName:toolVersion", and when the workflow is executing, it will use the real image to perform tht task.  
For example, when use `sub job` command,
```
genectl sub job /kubegene/bwa_help.sh --memory 1g --cpu 1 --tool bwa:0.7.12 --pvc pvc-gene
```
When use `sub workflow` command, the workflow will be:  

```
...
  mergemappedbam:
    tool: 'bwa:0.7.12'
    resources:
      memory: 1G
    commands:
      - >-
        ls ${volume-path-tmp}/${sample}/${sample}.*.sort.bam >
        ${volume-path-tmp}/${sample}/mergelist.txt
    depends:
      - target: bwamapping
        type: whole
  samflagstat:
    tool: 'bwa:0.7.12'
    resources:
      memory: 1G
    commands:
      - >
        samtools merge -f -@ ${nthread} -b
        ${volume-path-tmp}/${sample}/mergelist.txt \

        ${volume-path-tmp}/${sample}/${sample}.sort.bam && \

        samtools flagstat ${volume-path-tmp}/${sample}/${sample}.sort.bam >
        ${volume-path-tmp}/${sample}/${sample}.sort.flagstat
    depends:
      - target: mergemappedbam
        type: whole
...
```

## Common tools

There are some common tools that you can use to perform gene sequencing easily.  

| Tool name 	| Tool description                                                                                                                                                                                                                                                                                             	|
|-----------	|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| gatk      	| GATK: The Genome Analysis Tool Kit is a set of mutation detection software developed by the Broad Institute jointly developed by Harvard University and the Massachusetts Institute of Technology. It can perform error analysis and correction of base quality, detection of mutation areas, and filtering. 	|
| zsplit    	| Zsplit, split the original fastq file into several subfiles.                                                                                                                                                                                                                                                 	|
| bwa       	| BWA, Burrows Wheeler Aligner and Samtools are classic genomic alignment applications that can be used to create reference alignments of reference genomes, then perform quick alignment of genomes based on indexes, and perform sam, bam format conversion, sequence ordering, etc. operating.              	|
