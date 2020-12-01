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
> Compare the error rate with what you would expect with Illumina error rates.
> Compare the read length distribution in respect to the Illumina libraries.
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
>> $ scp YOUR-NETID@student-linux.tudelft.nl:~/asm_workshop/results/fastqc_ont/DRR198814_50k-reads_fastqc.html \
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
            > ~/asm_workshop/results/asm_ont/ont_overlaps.paf
~~~
{: .bash}

The all-vs-all alignments (`ont_overlaps.paf`) will be used as input for `miniasm` that will build a assembly graph in GFA format.

Besides the all-vs-all `PAF` file we also have to provide the raw ONT reads.

~~~
$ miniasm -f ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            ~/asm_workshop/results/asm_ont/ont_overlaps.paf \
            > ~/asm_workshop/results/asm_ont/ont_assembly.gfa
~~~
{: .bash}

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
            ~/asm_workshop/results/asm_ont/ont_assembly.gfa \
            > ~/asm_workshop/results/asm_ont/ont_polished.gfa
~~~
{: .bash}

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
>> scp YOUR-NETID@student-linux.tudelft.nl:~/asm_workshop/results/asm_ont/ont_polished.gfa \
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


