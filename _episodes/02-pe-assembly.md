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

`*De Novo*` assembly is the process of merging short sequencing reads into contiguous sequences (contigs).

Now that we checked and trimmed the Paired End library we are ready to assemble it.

Go to the asm_workshop folder

~~~
$ cd ~/asm_workshop
~~~
{: .bash}


# SPAdes Genome Assembler

We will use the 'SPAdes Genome Assembler'. [SPAdes](https://cab.spbu.ru/software/spades/)

SPAdes takes as input paired-end reads, mate-pairs and single (unpaired) reads in FASTA and FASTQ format.

In a first step SPAdes will do a read error corrction and use these in the iterative short-read genome assembly. 

To run SPAdes from the command line, type:

~~~
$ spades.py -h
~~~
{: .bash}

Assemble the trimmed `600bp` Paired End library with SPAdes and use as output folder `ecoli_pe`

Use `-1` for the forward reads and `-2` for the reverse reads.

~~~
$ spades.py -1 ~/asm_workshop/data/trimmed_fastq/PE_600bp_1.trim.fastq.gz \
            -2 ~/asm_workshop/data/trimmed_fastq/PE_600bp_2.trim.fastq.gz \
            -o ~/asm_workshop/results/spades_pe
~~~
{: .bash}

SPAdes created a new directory called `spades_pe`. Give a listing of this directory.

~~~
$ ls -l results/spades_pe
~~~
{: .bash}

A log file (spades.log) is created by spades, outputing all the steps and results. Inspect the result:

~~~
$ less results/spades_pe/spades.log
~~~
{: .bash}

At the end of the log file (press shift-G) it shows the files that has been created. The contigs.fasta file contains the assembly. Inspect this file by using less. (Use Q to exit less)

~~~
$ less results/spades_pe/contigs.fasta
~~~
{: .bash}

The first contig is called `NODE_1` and has a certain length and coverage. We can count the number of contigs in this file by doing a search on a string that is in common in all contigs (NODE) and use the pipe command to pass the resuls to `wc -l` to count every line in the output

~~~
$ grep "NODE" results/spades_pe/contigs.fasta | wc -l
~~~
{: .bash}


# Evaluate with QUAST

For evaluating the SPAdes assembly we can use the Genome assembly evaluation tool, QUAST [https://github.com/ablab/quast]([http://quast.sourceforge.net/quast](https://github.com/ablab/quast)).

We are going to compare the SPAdes generated contigs and scaffolds files in relation to a publicly available complete genome sequence of *Escherichia coli* K-12 which we find in the reference directory.

~~~
$ quast.py results/spades_pe/contigs.fasta \
            results/spades_pe/scaffolds.fasta \
            -R reference/Ecoli_K12_reference.fasta \
            -o results/quast_pe
~~~
{: .bash}


A summary table has been generated: `results/quast_pe/report.txt'.

~~~
$ less results/quast_pe/report.txt
~~~
{: .bash}

Compare the generated statistics for the two input files:
<ul>
<li>Number of large contigs/scaffolds (i.e., longer than 500 bp) and total length of them.</li>
<li>Length of the largest contig.</li>
<li>N50 and L50</li>
<li>Genome fraction %</li>
<li>Number of N's per 100 kbp</li>
</ul>


# Assembly alignment and visualization

We will use nucmer from the MUMmer package to align the contigs to the reference. [http://mummer.sourceforge.net/](http://mummer.sourceforge.net/)

Create a new folder called mummer in ~/asm_workshop/results/

~~~
$ mkdir ~/asm_workshop/results/mummer
~~~
{: .bash}

Move to this folder

~~~
$ cd results/mummer
~~~
{: .bash}

USAGE: nucmer [options] < reference > < Query >

Align the assembly (contigs.fasta) to the reference: (~/asm_workshop/reference/Ecoli_K12_reference.fasta)

~~~
$ nucmer --prefix spades_pe \
        ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
        ~/asm_workshop/results/spades_pe/contigs.fasta
~~~
{: .bash}

nucmer has aligned all contigs to the reference.

Run show-coords on the spades_pe.delta file:

~~~
$ show-coords spades_pe.delta
~~~
{: .bash}

This program parses the delta alignment output of nucmer This gives us the coordinates and displays the coordinates, and other useful information about the alignments.

Running 'show-coords -h' will provide the usage and explaines the output format:

~~~
$ show-tiling -h
~~~
{: .bash}

~~~
USAGE: show-coords  [options]  <deltafile>

-b          Merges overlapping alignments regardless of match dir
            or frame and does not display any idenitity information.
-B          Switch output to btab format
-c          Include percent coverage information in the output
-d          Display the alignment direction in the additional
            FRM columns (default for promer)
-g          Deprecated option. Please use 'delta-filter' instead
-h          Display help information
-H          Do not print the output header
-I float    Set minimum percent identity to display
-k          Knockout (do not display) alignments that overlap
            another alignment in a different frame by more than 50%
            of their length, AND have a smaller percent similarity
            or are less than 75% of the size of the other alignment
            (promer only)
-l          Include the sequence length information in the output
-L long     Set minimum alignment length to display
-o          Annotate maximal alignments between two sequences, i.e.
            overlaps between reference and query sequences
-q          Sort output lines by query IDs and coordinates
-r          Sort output lines by reference IDs and coordinates
-T          Switch output to tab-delimited format

  Input is the .delta output of either the "nucmer" or the
"promer" program passed on the command line.
  Output is to stdout, and consists of a list of coordinates,
percent identity, and other useful information regarding the
alignment data contained in the .delta file used as input.
  NOTE: No sorting is done by default, therefore the alignments
will be ordered as found in the <deltafile> input.
~~~
{: .output}

We will use mummerplot to plot the alignments:

~~~
$ mummerplot --png --layout --filter -p spades_pe \
        spades_pe.delta \
        -R ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
        -Q ~/asm_workshop/results/spades_pe/contigs_500bp.fasta
~~~
{: .bash}

A plot file 'spades_pe.png' has been created. Download the file to your local computer and inspect the file. 

In a new tab (local computer) in your terminal do:

~~~
$ mkdir ~/Desktop/mummer/
$ scp YOUR-NETID@vm0X-bt-edu.tnw.tudelft.nl:~/asm_workshop/results/mummer/spades_pe.png ~/Desktop/mummer/
~~~
{: .bash}

What does this tell us about the assembly?

# Visualise the assembly graphs with Bandage (Optional)

Bandage is a program for visualising de novo assembly graphs. By displaying connections which are not present in the contigs file.

Download Bandage from here: [http://rrwick.github.io/Bandage/](http://rrwick.github.io/Bandage/)
No installtion is necessary - just unzip and run.

When first opening Bandage on a Mac, you may receive a warning stating that Bandage 'can't be opened because it is from an unidentified developer.' Right click on the file and select 'Open' to override this warning.

download the assembly graph files to you local computer.
In a new tab (local computer) in your terminal do:

~~~
$ mkdir ~/Desktop/bandage/
$ scp YOUR-NETID@vm0X-bt-edu.tnw.tudelft.nl:~/asm_workshop/results/spades_pe/assembly_graph.fastg \
        ~/Desktop/bandage/spades_pe_graph.fastg
~~~
{: .bash}

start Bandage and load the file spades_pe_graph.fastg.

Click on "Draw graph" and save as image (current view)





