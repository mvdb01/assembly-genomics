---
title: "de novo Long Read Assembly"
teaching: 10
exercises: 0
questions:
- "How to do a de novo long-read genome assembly?"
objectives:
- "Long read vs short read characteristics "
- "Long read error rate vs short read error rate"
keypoints:
- ""
---

# *De Novo* assembly (Nanopore)

Recent new single molecule technologies like PacBio and Oxford Nanopore are promising. Here we will assemble data from NCBI Bioproject [https://www.ncbi.nlm.nih.gov/bioproject/PRJDB9013](https://www.ncbi.nlm.nih.gov/bioproject/PRJDB9013), which was used for `Benchmarks of de novo assemblers for bacterial genomes`. The data we will use is *E. coli* str. K12 substr. MG1655 with identifier `DRR198814` (https://www.ncbi.nlm.nih.gov/sra/DRX189231[accn]) which was sequenced on a MinION from Oxford Nanopore. [https://nanoporetech.com/products/minion](https://nanoporetech.com/products/minion)

# Quality Control

Like we have done for the short reads we are going to do quality control on the raw long read data.

## FastQC

> ## Exercise
> 
>  Assess the quality of the long reads from Oxford Nanopre technology (ONT) located in: `~/asm_workshop/data/ont/DRR198814_44x.fastq.gz`. These are single reads with different read lengths. We will use a subset of the data, in this case 44x coverage.  
>
>
> Use as output folder `~/asm_workshop/results/fastqc_ont`  
>
>
> Compare the error rate with those from the Illumina error rates.  
> Compare the read length distribution in respect to the Illumina libraries.  
>
>
> (Hint: Use `fastqc` and `scp` to download the created `html` files.)
>
>> ## Solution
>>  
>> Create an output folder for the result files.
>>
>> ~~~
>> $ mkdir -p ~/asm_workshop/results/fastqc_ont
>> ~~~
>> {: .bash}
>>
>> Run fastqc on the ONT library.
>>
>> ~~~
>> $ fastqc ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz -o ~/asm_workshop/results/fastqc_ont
>> ~~~
>> {: .bash}
>>
>> In a new tab (local computer) in your terminal do:
>>
>> ~~~
>> $ mkdir ~/Desktop/fastqc_html/
>> $ scp YOUR-NETID@student-linux.tudelft.nl:~/asm_workshop/results/fastqc_ont/DRR198814_44x_fastqc.html \
>>        ~/Desktop/fastqc_html/
>> ~~~
>> {: .bash}
>> 
>> Then take a look at the html files in your browser.
>>
> {: .solution}
{: .challenge}


# Miniasm assembler




Miniasm is a very fast OLC-based de novo assembler for noisy long reads like PacBio and Oxford Nanopore data. [https://github.com/lh3/miniasm](https://github.com/lh3/miniasm)

Miniasm needs an all-vs-all read self-mapping (typically by minimap2) as input.

Before we can start we first we will move to the asm_workshop folder

~~~
$ cd ~/asm_workshop
~~~
{: .bash}

And create an output folder for the Nanopore assembly results.

~~~
$ mkdir ~/asm_workshop/results/ecoli_ont
~~~
{: .bash}

## minimap2

The first step is to run minimap for the all-vs-all mapping.

Run minimap2 without options for command usage.

~~~
$  minimap2
~~~
{: .bash}

For all-vs-all mapping we use the raw ont reads as target `and` as query. As preset setting we have to use `-x ava-ont` for ONT read overlap and redirect the output to a `PAF` file (Pairwise mApping Format), which is a text format describing the approximate mapping positions between two sets of sequences.


~~~
$  minimap2 -x ava-ont \
            ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            > ~/asm_workshop/results/ecoli_ont/ont_overlaps.paf
~~~
{: .bash}

## miniasm

The all-vs-all alignments (`ont_overlaps.paf`) will be used as input for `miniasm` that will build a assembly graph in GFA format.

Besides the all-vs-all `PAF` file we also have to provide the raw ONT reads.

~~~
$ miniasm -f ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            ~/asm_workshop/results/ecoli_ont/ont_overlaps.paf \
            > ~/asm_workshop/results/ecoli_ont/ont_assembly.gfa
~~~
{: .bash}

## Consensus sequence

Now we have to extract the consence sequence from the `assembly graph` and for that we have to use `awk`, which is a linux tool that does pattern scanning and processing in text files.

The assembly graph contains the sequence that we need but we have to convert it to fasta format. AWK is searching for lines that start with `S` (`/^S/`) and starts printing the sequence identifiers `$2` with the fasta header symbol `>` in front of it. On the next line `\n` the actual sequence will be printed `$3`. 

~~~
$ awk '/^S/{print ">"$2"\n"$3}' \
        ~/asm_workshop/results/ecoli_ont/ont_assembly.gfa \
        > ~/asm_workshop/results/ecoli_ont/ont_assembly.fasta
~~~
{: .bash}

## Assembly Error rate

Since we are using noisy reads to do a De Novo assembly we would like to know how this will workout for the assembly.

We could align the assembly to the reference sequence we have to inspect the sequence similarity.

For this we will use the tool `dnadiff` from the `MUMmer` package. [https://github.com/garviz/MUMmer](https://github.com/garviz/MUMmer)

on how to use dnadiff:

~~~
$ dnadiff -h
~~~
{: .bash}

USAGE: dnadiff  [options]  <reference>  <query>

        <reference>       Set the input reference multi-FASTA filename
        <query>           Set the input query multi-FASTA filename

We will use as a reference: `~/asm_workshop/reference/Ecoli_K12_reference.fasta`

And the ONT assembly as query: `~/asm_workshop/results/ecoli_ont/ont/assmbly.fasta`

First we have to make an output folder:

~~~
$ mkdir ~/asm_workshop/results/mummer
~~~
{: .bash}

Run dnadiff with the -p option to control the output file name.

~~~
$ dnadiff -p ~/asm_workshop/results/mummer/ecoli_ont \
            ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
            ~/asm_workshop/results/ecoli_ont/ont_assmbly.fasta
~~~
{: .bash}

Now open the report file that was generated by dnadiff and inspect the average sequence identity `(AvgIdentity)` under the `1-to-1 alignments`.

~~~
$ less ~/asm_workshop/results/mummer/ecoli_ont.report
~~~
{: .bash}

> ## Exercise
> 
> We have looked into the sequence similarity of Noisy long read assembly but not for Illumina based assemblies.
>
>
> Apply `dnadiff` on the `paired-end` SPAdes assembly we did: `~/asm_workshop/results/ecoli_pe/contigs.fasta `.
>
> Use the same `mummer` output folder and call the output file ecoli_pe: `~/asm_workshop/results/mummer/ecoli_pe`
>
> Compare the average sequence identity from the `ecoli_pe` assembly with the `ecoli_ont` sequence identity.
> 
>
>> ## Solution
>>
>> run dnadiff with: 
>>
>> ~~~
>> dnadiff -p ~/asm_workshop/results/mummer/ecoli_pe \
>>            ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
>>            ~/asm_workshop/results/ecoli_pe/contigs.fasta
>> ~~~
>> {: .bash}
>> 
>> Open the report `ecoli_pe.report`and compare the average sequence identity `(AvgIdentity)` under the `1-to-1 alignments` with the from the ont assembly report: `ecoli_ont.report`
>> 
>> ~~~
>> $ less ~/asm_workshop/results/mummer/ecoli_pe.report
>> ~~~
>> {: .bash}
>>
> {: .solution}
{: .challenge}




# Error correcting (polishing) long read based assemblies

Contigs genereated from erroneous long read based *De Novo* assemblies needs to be error corrected to improve the consensus sequence. Here we will use Minipolish, which will use Racon for polishing, to error correct in three steps the miniasm assembly. 

## Step1: initial Racon polish with constituent reads

Miniasm's assembled contigs are made up of pieces of long reads and therefore has a high error rate – probably around 90% or so, depending on the input reads.

The miniasm GFA file indicates specifically which reads contributed to each contig.

Therefore, the first thing Minipolish does is to run Racon on each contig independently, only using the reads which were used to create that contig. This step typically quite fast because it does not involve high read depths, and it can bring the percent identity up to the high 90s.

## Step 2: full Racon polish rounds

Now that the assembly is in better shape, Minipolish does full Racon-polishing rounds – aligning the full read set to the whole assembly and getting a Racon consensus.

## Step 3: contig read depth

Minipolish finishes by doing one more read-to-assembly alignment, this time not to polish but to calculate read depths. These depths are added to the GFA line for each contig (e.g. dp:f:77.179) and they will be recognised if the graph is loaded in Bandage.

## Run Minipolish

To be able to run Minipolish we need to provide it with the raw reads and the assembly graph from the miniasm output and redirect this to a new now pollished assembly graph:

~~~
$ minipolish ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            ~/asm_workshop/results/ecoli_ont/ont_assembly.gfa \
            > ~/asm_workshop/results/ecoli_ont/ont_polished.gfa
~~~
{: .bash}

## Polished consensus sequence

Now we have to extract the consence sequence from the polished assembly graph and for that we have to use `awk`, which is a linux tool that does pattern scanning and processing in text files.

The assembly graph contains the sequence that we need but we have to convert it to fasta format. AWK is searching for lines that start with `S` (`/^S/`) and starts printing the sequence identifiers `$2` with the fasta header symbol `>` in front of it. On the next line `\n` the actual sequence will be printed `$3`. 

~~~
$ awk '/^S/{print ">"$2"\n"$3}' \
        ~/asm_workshop/results/ecoli_ont/ont_polished.gfa \
        > ~/asm_workshop/results/ecoli_ont/ont_polished.fasta
~~~
{: .bash}

> ## Exercise
> 
>
> Apply `dnadiff` on the `minipolished` assembly: `~/asm_workshop/results/ecoli_ont/ont_polished.fasta `.
>
> Use the same `mummer` output folder and call the output file ecoli_pe: `~/asm_workshop/results/mummer/ecoli_ont_minipolish`
>
> Compare the average sequence identity from the `ecoli_ont_minipolish` assembly with the `ecoli_ont` sequence identity.
> 
>
>> ## Solution
>>
>> run dnadiff with: 
>>
>> ~~~
>> dnadiff -p ~/asm_workshop/results/mummer/ecoli_ont_minipolish \
>>            ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
>>            ~/asm_workshop/results/ecoli_ont/ont_polished.fasta
>> ~~~
>> {: .bash}
>> 
>> Open the report `ecoli_ont_minipolish.report`and compare the average sequence identity `(AvgIdentity)` under the `1-to-1 alignments` with the from the ont assembly report: `ecoli_ont.report`
>> 
>> ~~~
>> $ less ~/asm_workshop/results/mummer/ecoli_ont_minipolish.report
>> ~~~
>> {: .bash}
>>
> {: .solution}
{: .challenge}


# Polishing with Pilon

Now that we have seen that we need to error correct (polish) assemblies that are based on noisy long reads. Minipolish used the noisy long reads to align these to the raw assembly and based on the sequence depth it was able to correct the sequence of the assembly.

Still the error rate is to high to do for instance an annotation. The errors would result in highly fragmented genes. 

An approach would be to use the Illumina reads from which we know it has high read accuracy. A polishing tool that can be used for this is Pilon.

Pilon requires as input a fasta file of the draft assembly and the aligned Illumina reads to this draft assembly in BAM format as we did during the Variant Calling pipeline.

> ## Exercise
> 
> Align the Illumina reads from `~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_1.trim.fastq.gz` and `~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_2.trim.fastq.gz` to the minipolished assembly: `~/asm_workshop/results/ecoli_ont/ont_polished.fasta` as we have done during the Variant Calling sessions.
> 
>
> Use `bwa mem ` to do the mapping and create an `SAM` file
>
> Use `samtools` to `sort` and convert the `SAM` file to a sorted `BAM` file.
>
>  
>
>> ## Solution
>>
>> First index the polished assembly: 
>>
>> ~~~
>> $ bwa index ~/asm_workshop/results/ecoli_ont/ont_polished.fasta
>> ~~~
>> {: .bash}
>>
>> Map the paired-end Illumina reads to the indexed polished assembly:
>> ~~~
>> $ bwa mem ~/asm_workshop/results/ecoli_ont/ont_polished.fasta \
>>          ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_1.trim.fastq.gz \
>>          ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_2.trim.fastq.gz \
>>          > ~/asm_workshop/results/ecoli_ont/pe_polished.sam
>> ~~~
>> {: .bash}
>>
>> Sort and convert the SAM file
>>
>> ~~~
>>  samtools sort -O bam \
>>            -o ~/asm_workshop/results/ecoli_ont/pe_polished.sorted.bam \
>>            ~/asm_workshop/results/ecoli_ont/pe_polished.sam
>> ~~~
>> {: .bash}
>> 
>>
> {: .solution}
{: .challenge}

## Pilon

Before we can apply `pilon` we first have to `index` the alignment file.

~~~
$ samtools index ~/asm_workshop/results/ecoli_ont/pe_polished.sorted.bam
~~~
{: .bash}

Now we are ready to run pilon:

~~~
$ java -Xmx2G -jar /mnt/linapps/share/java/pilon-1.18.jar \
        --threads 4 \
        --genome ~/asm_workshop/results/ecoli_ont/ont_polished.fasta \
        --frags ~/asm_workshop/results/ecoli_ont/pe_polished.sorted.bam \
        --output ~/asm_workshop/results/ecoli_ont/ont_pilon_polished \
        --changes --fix all
~~~
{: .bash}




> ## Exercise
> 
>
> Apply `dnadiff` on the `pilon polished` assembly: `~/asm_workshop/results/ecoli_ont/ont_pilon_polished.fasta `.
>
> Use the same `mummer` output folder and call the output file ecoli_pe: `~/asm_workshop/results/mummer/ecoli_ont_pilon`
>
> Compare the average sequence identity from the `ecoli_ont_pilon` assembly with the `ecoli_ont` and `ecoli_ont_minipolish` sequence identity.
> 
>
>> ## Solution
>>
>> run dnadiff with: 
>>
>> ~~~
>> dnadiff -p ~/asm_workshop/results/mummer/ecoli_ont_pilon \
>>            ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
>>            ~/asm_workshop/results/ecoli_ont/ont_pilon_polished.fasta
>> ~~~
>> {: .bash}
>> 
>> Open the report `ecoli_ont_pilon.report`and compare the average sequence identity `(AvgIdentity)` under the `1-to-1 alignments` with the from the ont assembly report: `ecoli_ont_minipolish.report`
>> 
>> ~~~
>> $ less ~/asm_workshop/results/mummer/ecoli_ont_pilon.report
>> ~~~
>> {: .bash}
>>
> {: .solution}
{: .challenge}



# QUAST: compare ONT, PE and PE-MP assemblies

> ## Exercise
> 
> Compare the `pilon polished ONT` assembly with the illumina assemblies `PE` and `PE-MP` by using `QUAST`.
> 
>
> What are the major differences between the two assemblies?
>
>
>
> Use `quast_pe_mp_ont` as output folder.
>
>
>
>> ## Solution
>> 
>> ~~~
>> $ quast.py \
>>      ~/asm_workshop/results/ecoli_pe/contigs.fasta \
>>      ~/asm_workshop/results/ecoli_pe_mp/contigs.fasta \
>>      ~/asm_workshop/results/ecoli_ont/ont_pilon_polished.fasta \
>>      -o ~/asm_workshop/results/quast_pe_mp_ont
>> ~~~
>> {: .bash}
>>
>> In a new tab (local computer) in your terminal do:
>>
>> ~~~
>> $ scp YOUR-NETID@student-linux.tudelft.nl:~/asm_workshop/results/quast_pe_mp_ont/report.html \
        ~/Desktop/quast/report_pe_mp_ont.html
>> ~~~
>> {: .bash}
>> 
> {: .solution}
{: .challenge}


# Visualise the assembly graphs with Bandage (Optional)

> ## Exercise
> 
> Download the `polished assembly graph` of the `ONT` assembly and compare it with the `assembly graph` of the `PE` and `PE-MP` assemblies.
> 
>
>> ## Solution
>>
>> In a new tab (local computer) in your terminal do:
>>
>> ~~~
>> scp YOUR-NETID@student-linux.tudelft.nl:~/asm_workshop/results/ecoli_ont/ont_polished.gfa \
>>        ~/Desktop/bandage/
>> ~~~
>> {: .bash}
>> 
>>
>> start Bandage and load the file ont_polished.gfa.
>>
>> Click on "Draw graph" and save as image (current view)
>>
>> compare with the `PE` `PE-MP` assembly graphs.
>>
> {: .solution}
{: .challenge}




# Assembly alignment

> ## Exercise
>
>
> Align the polished ONT assembly to the reference: (`~/asm_workshop/reference/Ecoli_K12_reference.fasta`).
> Use as a prefix: `ecoli_ont`.
>
> Make sure you are working in the mummer folder: `~/asm_workshop/results/mummer`
>
> Inspect the resulting `ecoli_ont.png` plot.
> Has the long reads improved the assembly?
>
>> ## Solution
>> 
>> Use as working directory: `~/asm_workshop/results/mummer`
>> ~~~
>> $ cd ~/asm_workshop/results/mummer
>> ~~~
>> {: .bash}
>> 
>> Run nucmer
>> 
>> ~~~
>> $ nucmer --prefix ecoli_ont \
>>          ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
>>          ~/asm_workshop/results/asm_ont/ont_polished.fasta
>> ~~~
>> {: .bash}
>>
>> nucmer has aligned the assembly to the reference.
>> 
>> Use mummerplot to plot the alignments:
>>
>> ~~~
>> $ mummerplot --png --layout --filter --prefix ecoli_ont \
>>          ~/asm_workshop/results/mummer/ecoli_ont.delta \
>>          -R ~/asm_workshop/reference/Ecoli_K12_reference.fasta \
>>          -Q ~/asm_workshop/results/asm_ont/ont_polished.fasta
>> ~~~
>> {: .bash}
>>
>> A plot file 'ecoli_ont.png' has been created. Download the file to your local computer and inspect the file. 
>> 
>> In a new tab (local computer) in your terminal do:
>> 
>> ~~~
>> $ scp YOUR-NETID@student-linux.tudelft.nl:~/asm_workshop/results/mummer/ecoli_ont.png ~/Desktop/mummer/
>> ~~~
>> {: .bash}
>>
> {: .solution}
{: .challenge}

