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

During this session we will work with sequencing reads coming from a study of Trivedi *et all* [doi](https://dx.doi.org/10.3389%2Ffgene.2014.00111) were they used benchmark datasets generated from control samples across a range of genome sizes to illustrate that QC inferences made using draft assemblies are broadly equivalent to those made using a well-established reference. Multiple different organismes were sequenced but here we will use only a 600bp paired-end and a 2.5 kb mate pair library from *Escherichia coli* from illumina platform.

As a first step we will inspect the paired-end library. The data structure and do some quality control and filtering. Before we can work with the data we first create a working directory and set the environment.

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


# Quality Control

We learned about fastq files and how to do quality control in the variant calling sessions. [fastq and quality control](https://mvdb01.github.io/wrangling-genomics/02-quality-control/index.html)



> ## Exercise
> 
>  Asses the quality of the paired-end library called PE_600bp_50x. PE stands for Paired-end, 600bp is the insert-size of the sequenced fragment and we will use a subset of the data, in this case 50x coverage. 
> (Hint: Use `fastqc` and `scp` to download the created `html` files.)
>
>> ## Solution
>>  
>> Create an output folder for the result files.
>>
>> ~~~
>> $ mkdir -p results/fastqc_untrimmed_reads
>> ~~~
>> {: .bash}
>>
>> Run fastqc on the paired-end library
>>
>> ~~~
>> $ fastqc data/PE_600bp_50x_* -o results/fastqc_untrimmed_reads
>> ~~~
>> {: .bash}
>>
>> In a new tab (local computer) in your terminal do:
>>
>> ~~~
>> $ mkdir ~/Desktop/fastqc_html/
>> $ scp YOUR-NETID@student-linux.tudelft.nl:~/LM3601/results/fastqc_untrimmed_reads/*.html ~/Desktop/fastqc_html/
>> ~~~
>> {: .bash}
>> 
>> Then take a look at the html files in your browser.
>>
> {: .solution}
{: .challenge}


# Trimming

Trimmomatic performs a variety of useful trimming tasks, like removal of bad quality data and adapters, for illumina paired-end and single ended data. For a recap look at the trimming lesson. [trimmomatic](https://mvdb01.github.io/wrangling-genomics/03-trimming/index.html)


> ## Exercise
> 
> Apply Trimmomatic on the Paired End 600bp frags library using:
>
> 1. No adapter trimming.
> 2. Remove leading low quality or N bases (below quality 3) (LEADING:3)
> 3. Remove trailing low quality or N bases (below quality 3) (TRAILING:3)
> 4. Scan the read with a 4-base wide sliding window, cutting when the average quality per base drops below 15 (SLIDINGWINDOW:4:15)
> 5. Drop reads below the 100 bases long (MINLEN:100)
>
> How many reads were kept and how many removed?
>
>> ## Solution
>>  
>> Create an output folder trimmed_fastq in folder data.
>>
>> ~~~
>> $ mkdir -p data/trimmed_fastq
>> ~~~
>> {: .bash}
>>
>> Move into trimmed_fastq
>>
>> ~~~
>> $ cd data/trimmed_fastq
>> ~~~
>> {: .bash}
>>
>> Run trimmomatic on the paired end library:
>>
>> ~~~
>> $ trimmomatic PE \
        ~/LM3601/data/untrimmed_fastq/PE_600bp_50x_1.fastq.gz ~/LM3601/data/untrimmed_fastq/PE_600bp_50x_2.fastq.gz \
        PE_600bp_50x_1.trim.fastq.gz PE_600bp_50x_1un.trim.fastq.gz \
        PE_600bp_50x_2.trim.fastq.gz PE_600bp_50x_2un.trim.fastq.gz \
        LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:100 
>> ~~~
>> {: .bash}
> {: .solution}
{: .challenge}

> ## Exercise
> 
>  Asses the quality of the trimmed paired-end library.
>
>> ## Solution
>>  
>> Create an output folder for the result files.
>>
>> ~~~
>> $ mkdir -p ~/LM3601/results/fastqc_trimmed_reads
>> ~~~
>> {: .bash}
>>
>> Run fastqc on the paired-end library
>>
>> ~~~
>> $ fastqc ~/LM3601/data/trimmed_fastq/PE_600bp_50x_* -o ~/LM3601/results/fastqc_trimmed_reads
>> ~~~
>> {: .bash}
>>
>> In a new tab (local computer) in your terminal do:
>>
>> ~~~
>> $ mkdir ~/Desktop/fastqc_html/
>> $ scp YOUR-NETID@student-linux.tudelft.nl:~/LM3601/results/fastqc_trimmed_reads/*.html ~/Desktop/fastqc_html/
>> ~~~
>> {: .bash}
>> 
>> Then take a look at the html files in your browser.
>>
> {: .solution}
{: .challenge}