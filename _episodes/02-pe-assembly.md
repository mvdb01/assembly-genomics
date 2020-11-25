---
title: "de novo Short Read Paired-End Assembly"
teaching: 10
exercises: 0
questions:
- "How to do a de novo short read paired-end genome assembly?"
objectives:
- "Explain what is a contig"
- "Be able to calculate genome coverage"
- "Explain genome statistics"
keypoints:
- ""
---

# *De Novo* assembly (Paired End libary)

*De Novo* assembly is the process of merging short sequencing reads into contiguous sequences (contigs).

Now that we checked and trimmed the Paired End library we are ready to assemble it.

Go to the asm_workshop folder

~~~
$ cd ~/asm_workshop
~~~
{: .bash}

# SPAdes Genome Assembler

We will use the 'SPAdes Genome Assembler'. [SPAdes](https://cab.spbu.ru/software/spades/)

To run SPAdes from the command line, type:

~~~
$ spades.py -h
~~~
{: .bash}

Assemble the trimmed 600bp Paired End library with SPAdes and use as output folder "ecoli_pe"

Use -1 for the forward reads and -2 for the reverse reads.

~~~
$ spades.py -1 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_1.trim.fastq.gz \
            -2 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_2.trim.fastq.gz \
            -o ~/asm_workshop/results/ecoli_pe
~~~
{: .bash}

SPAdes created a new directory called "ecoli_pe". Give a listing of this directory.

~~~
$ ls -l results/ecoli_pe
~~~
{: .bash}

A log file (spades.log) is created by spades, outputing all the steps and results. Inspect the result:

~~~
$ less results/ecoli_pe/spades.log
~~~
{: .bash}

At the end of the log file (press shift-G) it shows the files that has been created. The contigs.fasta file contains the assembly. Inspect this file by using less. (Use Q to exit less)

~~~
$ less results/ecoli_pe/contigs.fasta
~~~
{: .bash}

The first contig is called `NODE_1` and had a certain length and coverage. We can count the number of contigs in this file by doing a search on a string that is in common in all contigs (NODE) and use the pipe command to pass the resuls to `wc -l` to count every line in the output

~~~
$ grep "NODE" results/ecoli_pe/contigs.fasta | wc -l
~~~
{: .bash}

Now run the python script assemblyStats.py on the contigs file to get some statistics.

~~~
$ assemblyStats.py results/ecoli_pe/contigs.fasta
~~~
{: .bash}

# Filter assembly

From the assembly statistics we find that there are 200 contigs. A lot of them smaller (56 bp) than the read length (100 bp). We therefor arbitrarily going to filter the assembly, keeping contigs > 500 bp

~~~
$ filterFasta_500bp.py -i results/ecoli_pe/contigs.fasta -o results/ecoli_pe/contigs_500bp.fasta
~~~
{: .bash}

Inspect the filtered assembly and compare it with the unfiltered.

~~~
$ assemblyStats.py results/ecoli_pe/contigs_500bp.fasta
~~~
{: .bash}

Have we assembled the complete genome of E. coli K12 substr. MG1655?

Compare the sum of all contigs with the genome size on NCBI: [https://www.ncbi.nlm.nih.gov/genome/167?project_id=57779](https://www.ncbi.nlm.nih.gov/genome/167?project_id=57779)

How many contigs represent 50% of the assembly? What is the N50?

Open the filtered contigs file and select randomly sequence from the contigs and BLAST it on NCBI. [https://blast.ncbi.nlm.nih.gov/Blast.cgi](https://blast.ncbi.nlm.nih.gov/Blast.cgi)

# Assembly with different kmers

> ## Exercise
> 
> Now make different assemblies with different kmer lengths to see the effect of the kmers. One with a small kmer (like 21) and the other with a higher kmer (like 77). change the output folder name to create new output folders.
> 
>> ## Solution
>> 
>> Use the `-k` option to set the kmer length. We will use here 21 as kmer length.
>> 
>> ~~~
>> $ spades.py -1 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_1.trim.fastq.gz \
>>             -2 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_2.trim.fastq.gz \
>>             -o ~/asm_workshop/results/ecoli_pe_k21
>>             -k 21
>> ~~~
>> {: .bash}
>> 
>> And for the second assembly we will use a kmer of 77.
>> 
>> ~~~
>> $ spades.py -1 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_1.trim.fastq.gz \
>>             -2 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_2.trim.fastq.gz \
>>             -o ~/asm_workshop/results/ecoli_pe_k77
>>             -k 77
>> ~~~
>> {: .bash}
>> 
> {: .solution}
{: .challenge}

# Evaluate with QUAST

