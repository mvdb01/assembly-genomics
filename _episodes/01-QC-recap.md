---
title: "Quality Control and Trimming Recap"
teaching: 10
exercises: 0
questions:
- "How to start a genome assembly?"
- "How can I describe the quality of my data?"
- "How can I get rid of sequence data that doesn’t meet my quality standards?"
objectives:
- "Explain how a FASTQ file encodes per-base quality scores."
- "Interpret a FastQC plot summarizing per-base quality across all reads."
- "Clean FASTQ reads using Trimmomatic."
keypoints:
- "The options you set for the command-line tools you use are important!"
- "Data cleaning is an essential step in a genomics workflow."
- "Quality encodings vary across sequencing platforms."
- ""
---

# *De Novo* assembly  workflows  

When working with high-throughput sequencing data, the raw reads you get off of the sequencer will need to pass
through a number of  different tools in order to generate your final desired output. The execution of this set of
tools in a specified order is commonly referred to as a *workflow* or a *pipeline*. 

An example of the workflow we will be using for our *De Novo* assembly is provided below with a brief
description of each step.

1. Quality control - Assessing quality using FastQC
2. Quality control - Trimming and/or filtering reads (if necessary)
3. Assemble the reads
4. Perform quality control of the assembly
5. Annotate the assembled genome

Like in the variant calling workflow, these workflows adopt a plug-and-play approach in that the output of one tool can be easily used as input to another tool without any extensive configuration. Having standards for data formats is what 
makes this feasible. Standards ensure that data is stored in a way that is generally accepted and agreed upon 
within the community. The tools that are used to analyze data at different stages of the workflow are therefore 
built under the assumption that the data will be provided in a specific format.

# Starting with Data

During this session we will work with sequencing reads coming from Illumina and Oxford Nanopore technology. Also here genomic DNA from *Escherichia coli* was used.

As a first step we will inspect sequence data from the illumina platform. The data structure and do some quality control and filtering. Before we can work with the data we first create a working directory and set the environment.

Login to one of the virtual machines (VMs) vmXX-bt-edu.tnw.tudelft.nl (XX= 01 - 08)

Load the environment

~~~
$ source /mnt/linapps/conda3loader
$ conda activate LM3601
~~~
{: .bash}

Use pwd (print working directory) to see in wich directory you are:

~~~
$ pwd
~~~
{: .bash}

You will get something like:

~~~
$ /home/nfs/YOUR-NETID
~~~
{: .output}

If you are not in your home folder type "cd" to go back to your home folder:

~~~
$ cd
~~~
{: .bash}

Copy the data that we are going to use for this session:

~~~
$ cp -r /mnt/linapps/share/LM3601/ .
~~~
{: .bash}

Move into the just created directorie "LM3601":

~~~
$ cd LM3601
~~~
{: .bash}

Check with "pwd" that you have something like:

~~~
$ /home/nfs/YOUR-NETID/LM3601
~~~
{: .output}


#FastQ format