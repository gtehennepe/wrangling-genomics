---
title: "Automating a Variant Calling Workflow"
teaching: 30
exercises: 15
questions:
- "How can I make my workflow more efficient and less error-prone?"
objectives:
- "Write a shell script with multiple variables."
- "Incorporate a `for` loop into a shell script."
- ""
keypoints:
- "We can combine multiple commands into a shell script to automate a workflow."
- "Use `echo` statements within your scripts to get an automated progress update."
---
# What is a shell script?

You wrote a simple shell script in a [previous lesson](http://www.datacarpentry.org/shell-genomics/05-writing-scripts/) that we used to extract bad reads from our
FASTQ files and put them into a new file. 

Here's the script you wrote:

~~~
grep -B1 -A2 NNNNNNNNNN *.fastq > scripted_bad_reads.txt

echo "Script finished!"
~~~
{: .bash}

That script was only two lines long, but shell scripts can be much more complicated
than that and can be used to perform a large number of operations on one or many 
files. This saves you the effort of having to type each of those commands over for
each of your data files and makes your work less error-prone and more reproducible. 
For example, the variant calling workflow we just carried out had about eight steps
where we had to type a command into our terminal. Most of these commands were pretty 
long. If we wanted to do this for all six of our data files, that would be forty-eight
steps. If we had 50 samples (a more realistic number), it would be 400 steps! You can
see why we want to automate this.

We've also used `for` loops in previous lessons to iterate one or two commands over multiple input files. 
In these `for` loops you used variables to enable you to run the loop on multiple files. We will be using variable 
assignments like this in our new shell scripts.

Here's the `for` loop you wrote for unzipping `.zip` files: 

~~~
$ for filename in *.zip
> do
> unzip $filename
> done
~~~
{: .bash}

And here's the one you wrote for running Trimmomatic on all of our `.fastq` sample files.

~~~
$ for infile in *.fastq
> do
> outfile=$infile\_trim.fastq
> java -jar ~/Trimmomatic-0.32/trimmomatic-0.32.jar SE $infile $outfile SLIDINGWINDOW:4:20 MINLEN:20
> done
~~~
{: .bash}

In this lesson, we will create two shell scripts. The first will run our FastQC analysis, 
including creating our summary file. To do this, we'll take each of the commands we entered to run FastQC and 
process the output files and put them into a single file with a `.sh` extension. The `.sh` is not essential, but
serves as a reminder to ourselves and to the computer that this is a shell script.

# Analyzing Quality with FastQC

We will use the command `touch` to create a new file where we will write our shell script. We will create this script in a new
directory called `scripts/`. Previously, we used
`nano` to create and open a new file. The command `touch` allows us to create a new file without opening that file.

~~~
$ cd ~/dc_workshop
$ mkdir scripts
$ cd scripts
$ touch read_qc.sh
$ ls 
~~~
{: .bash}

~~~
read_qc.sh
~~~
{: .output}

We now have an empty file called `read_qc.sh` in our `scripts/` directory. We will now open this file in `nano` and start
building our script.

~~~
$ nano read_qc.sh
~~~
{: .bash}

Enter the following pieces of code into your shell script (not into your terminal prompt).

Our first line will move us into the `untrimmed_fastq/` directory when we run our script.

~~~
cd ~/dc_workshop/data/untrimmed_fastq/
~~~
{: .output}

These next two lines will give us a status message to tell us that we are currently running FastQC, then will run FastQC
on all of the files in our current directory with a `.fastq` extension. 

~~~
echo "Running FastQC ..."
~/FastQC/fastqc *.fastq
~~~
{: .output}

Our next line will create a new directory to hold our FastQC output files. Here we are using the `-p` option for `mkdir`. This 
option forces `mkdir` to create the new directory, even if one of the parent directories doesn't already exist. It is a good
idea to use this option in your shell scripts to avoid running into errors if you don't have the directory structure you think
you do.'

~~~
mkdir -p ~/dc_workshop/results/fastqc_untrimmed_reads
~~~
{: .output}

Our next three lines first give us a status message to tell us we are saving the results from FastQC, then moves all of the files
with a `.zip` or a `.html` extension to the directory we just created for storing our FastQC results. 

~~~
echo "Saving FastQC results..."
mv *.zip ~/dc_workshop/results/fastqc_untrimmed_reads/
mv *.html ~/dc_workshop/results/fastqc_untrimmed_reads/
~~~
{: .output}

The next line moves us to the results directory where we've stored our output.

~~~
cd ~/dc_workshop/results/fastqc_untrimmed_reads/
~~~
{: .output}

The next five lines should look very familiar. First we give ourselves a status message to tell us that we're unzipping our ZIP
files. Then we run our for loop to unzip all of the `.zip` files in this directory.

~~~
echo "Unzipping..."
for filename in *.zip
    do
    unzip $filename
    done
~~~
{: .output}

Next we concatenate all of our summary files into a single output file, with a status message to remind ourselves that this is 
what we're doing.

~~~
echo "Saving summary..."
cat */summary.txt > ~/dc_workshop/docs/fastqc_summaries.txt
~~~
{: .output}

> ## Using echo statements
> 
> Add a note here about why it's a good idea to use status reminders in your code.
>
>
{: .callout}

Your full shell script should now look like this:

~~~
cd ~/dc_workshop/data/untrimmed_fastq/

echo "Running FastQC ..."
~/FastQC/fastqc *.fastq

mkdir -p ~/dc_workshop/results/fastqc_untrimmed_reads

echo "Saving FastQC results..."
mv *.zip ~/dc_workshop/results/fastqc_untrimmed_reads/
mv *.html ~/dc_workshop/results/fastqc_untrimmed_reads/

cd ~/dc_workshop/results/fastqc_untrimmed_reads/

echo "Unzipping..."
for filename in *.zip
    do
    unzip $filename
    done

echo "Saving summary..."
cat */summary.txt > ~/dc_workshop/docs/fastqc_summaries.txt
~~~
{: .output}

Save your file and exit `nano`. We can now run our script:

~~~
$ bash read_qc.sh
~~~
{: .bash}

~~~
Running FastQC ...
Started analysis of SRR097977.fastq
Approx 5% complete for SRR097977.fastq
Approx 10% complete for SRR097977.fastq
Approx 15% complete for SRR097977.fastq
Approx 20% complete for SRR097977.fastq
Approx 25% complete for SRR097977.fastq
. 
. 
. 
~~~
{: .output}

For each of your sample files, FastQC will ask if you want to replace the existing version with a new version. This is 
because we have already run FastQC on this samples files and generated all of the outputs. We are now doing this again using
our scripts. Go ahead and select `A` each time this message appears. It will appear once per sample file (six times total).

~~~
replace SRR097977_fastqc/Icons/fastqc_icon.png? [y]es, [n]o, [A]ll, [N]one, [r]ename:
~~~
{: .output}

> ## Exercise
> 
>
>
{: .challenge}


# Automating the Rest of our Variant Calling Workflow

Now we will create a second shell script to complete the other steps of our variant calling
workflow. To do this, we will take all of the individual commands that we wrote before, put them into a single file, 
add variables so that
the script knows to iterate through our input files and do a few other formatting that
we'll explain as we go. This is very similar to what we did with our `read_qc.sh` script, but will be a bit more complex.

First we need to set up a directory structure to store our output files. We will also bring our input files (including our trimmed samples
and our reference genome) into this new directory structure. To make things easier for us, our directory already contains the 
files and subdirectories that we want to use. These files are compressed into a compressed file called a tarball and ends with the
`.tar.gz` extension. This is another type of compressed file, similar to a `.zip` file. We need to decompress this file to 
create our directory structure. We use the command `tar` to decompress our tarball, with the options `-zxvf`. 

~~~
$ tar -zxvf variant_calling.tar.gz
~~~
{: .bash}

This will create a directory tree that contains some input data (reference genome and fastq files)
and a shell script that details the series of commands used to run the variant calling workflow (`run_variant_calling.sh`).

Our variant calling workflow will do the following steps

1. Index the reference genome for use by bwa and samtools
2. Align reads to reference genome
3. Convert the format of the alignment to sorted BAM, with some intermediate steps.
4. Calculate the read coverage of positions in the genome
5. Detect the single nucleotide polymorphisms (SNPs)
6. Filter and report the SNP variants in VCF (variant calling format)


We will be recreating this script together. The version provided is for your reference. First, we will move our 
`variant_calling/` directory to our `dc_workshop/` directory.

~~~
mv ~/dc_sample_data/variant_calling ~/dc_workshop/
~~~
{: .bash}

Next, we will create a new script in our `scripts/` directory using `touch`. 

~~~
$ cd ~/dc_workshop/scripts
$ touch run_variant_calling_2.sh
$ ls 
~~~
{: .bash}

~~~
read_qc.sh  run_variant_calling_2.sh
~~~
{: .output}

We now have a new empty file called `run_variant_calling_2.sh` in our `scripts/` directory. We will open this file in `nano` and start
building our script, like we did before.

~~~
$ nano run_variant_calling_2.sh
~~~
{: .bash}

Enter the following pieces of code into your shell script (not into your terminal prompt).

The first command is to change to our working directory
so the script can find all the files it expects.

~~~
cd ~/dc_workshop/variant_calling
~~~
{: .output}

Next we tell our script where to find the reference genome by assigning the `genome` variable to 
the path to our reference genome: 

~~~
genome=data/ref_genome/ecoli_rel606.fasta
~~~
{: .output}

> ## Creating Variables
> Within the Bash shell you can create variables at any time (as we did
> above, and during the 'for' loop lesson). Assign any name and the 
> value using the assignment operator: '='. You can check the current
> definition of your variable by typing into your script: echo $variable_name. 
{: .callout}

Next we index our reference genome for BWA.

~~~
bwa index $genome
~~~
{: .output}

And create the directory structure to store our results in: 

~~~
mkdir -p results/sai results/sam results/bam results/bcf results/vcf
~~~
{: .output}

We will now use a loop to run the variant calling workflow on each of our FASTQ files. The full list of commands
within the loop will be executed once for each of the FASTQ files in the `data/trimmed_fastq/` directory. 
We will include a few `echo` statements to give us status updates on our progress.

The first thing we do is assign the name of the FASTQ file we're currently working with to a variable called `fq` and
tell the script to `echo` the filename back to us so we can check which file we're on.

~~~
for fq in data/trimmed_fastq/*.fastq
    do
    echo "working with file $fq"
    done
~~~
{: .bash}


> ## Indentation
> 
> All of the statements within your `for` loop (i.e. everything after the `for` line and including the `done` line) 
> need to be indented. This indicates to the shell interpretor that these statements are all part of the `for` loop
> and should be done once per input.
> 
{: .callout}


> ## Exercise
> 
> This is a good time to check that our script is assigning the FASTQ filename variables correctly. Save your script and run
> it. What output do you see?
>
>> ## Solution 
>> 
>> ~~~
>> $ bash run_variant_calling_2.sh
>> ~~~
>> {: .bash}
>> 
>> ~~~
>> [bwa_index] Pack FASTA... 0.04 sec
>> [bwa_index] Construct BWT for the packed sequence...
>> [bwa_index] 1.06 seconds elapse.
>> [bwa_index] Update BWT... 0.03 sec
>> [bwa_index] Pack forward-only FASTA... 0.02 sec
>> [bwa_index] Construct SA from BWT and Occ... 0.57 sec
>> [main] Version: 0.7.5a-r405
>> [main] CMD: bwa index data/ref_genome/ecoli_rel606.fasta
>> [main] Real time: 1.801 sec; CPU: 1.724 sec
>> working with file data/trimmed_fastq/SRR097977.fastq
>> working with file data/trimmed_fastq/SRR098026.fastq
>> working with file data/trimmed_fastq/SRR098027.fastq
>> working with file data/trimmed_fastq/SRR098028.fastq
>> working with file data/trimmed_fastq/SRR098281.fastq
>> working with file data/trimmed_fastq/SRR098283.fastq
>> ~~~
>> {: .output}
>> 
>> You should see "working with file . . . " for each of the six FASTQ files in our `trimmed_fastq/` directory.
>> If you don't see this output, then you'll need to troubleshoot your script. A common problem is that your directory might not
>> be specified correctly. Ask for help if you get stuck here! 
> {: .solution}
{: .challenge}

Now that we've tested the components of our loops so far, we will add our next few steps. Remove the line `done` from the end of
your script and add the next two lines. These lines extract the base name of the file
(excluding the path and `.fastq` extension) and assign it
to a new variable called `base` variable. Add `done` again at the end so we can test our script.

~~~
    base=$(basename $fq .fastq)
    echo "base name is $base"
    done
~~~
{: .output}

Now if you save and run your script, the final lines of your output should look like this: 

~~~
working with file data/trimmed_fastq/SRR097977.fastq
base name is SRR097977
working with file data/trimmed_fastq/SRR098026.fastq
base name is SRR098026
working with file data/trimmed_fastq/SRR098027.fastq
base name is SRR098027
working with file data/trimmed_fastq/SRR098028.fastq
base name is SRR098028
working with file data/trimmed_fastq/SRR098281.fastq
base name is SRR098281
working with file data/trimmed_fastq/SRR098283.fastq
base name is SRR098283
~~~
{: .output}

For each file, you see two statements printed to the terminal window. This is because we have two `echo` statements. The first
tells you which file the loop is currently working with. The second tells you the base name of the file. This base name is going
to be used to create our output files.

Next we will create variables to store the names of our output files. This will make your script easier
to read because you won't need to type out the full name of each of the files. We're using the `base` variable that we just
defined, and adding different file name extensions to represent the files that will come out of each step in our workflow.

~~~
    fq=data/trimmed_fastq/$base\.fastq
    sai=results/sai/$base\_aligned.sai
    sam=results/sam/$base\_aligned.sam
    bam=results/bam/$base\_aligned.bam
    sorted_bam=results/bam/$base\_aligned_sorted.bam
    raw_bcf=results/bcf/$base\_raw.bcf
    variants=results/bcf/$base\_variants.bcf
    final_variants=results/vcf/$base\_final_variants.vcf    
~~~
{: .output}

Now that we've created our variables, we can start doing the steps of our workflow. Remove the `done` line from the end of
your script and add the following lines. 

1) align the reads to the reference genome and output a `.sai` file:

~~~
    bwa aln $genome $fq > $sai
~~~
{: .output}

2) convert the output to SAM format:

~~~
    bwa samse $genome $sai $fq > $sam
~~~
{: .output}

3) convert the SAM file to BAM format:

~~~
    samtools view -S -b $sam > $bam
~~~
{: .output}

4) sort the BAM file:

~~~
    samtools sort -f $bam $sorted_bam
~~~
{: .output}

5) index the BAM file for display purposes:

~~~
    samtools index $sorted_bam
~~~
{: .output}

6) do the first pass on variant calling by counting
read coverage

~~~
    samtools mpileup -g -f $genome $sorted_bam > $raw_bcf
~~~
{: .output}

7) call SNPs with bcftools:

~~~
    bcftools view -bvcg $raw_bcf > $variants
~~~
{: .output}

8) filter the SNPs for the final output:

~~~
    bcftools view $variants | /usr/share/samtools/vcfutils.pl varFilter - > $final_variants
    done
~~~
{: .output}

We added a `done` line after the SNP filtering step because this is the last step in our `for` loop.

Your script should now look like this:

~~~
cd ~/dc_workshop/variant_calling

genome=data/ref_genome/ecoli_rel606.fasta

bwa index $genome

mkdir -p results/sai results/sam results/bam results/bcf results/vcf

for fq in data/trimmed_fastq/*.fastq
    do
    echo "working with file $fq"
    base=$(basename $fq .fastq)
    echo "base name is $base"

    fq=data/trimmed_fastq/$base\.fastq
    sai=results/sai/$base\_aligned.sai
    sam=results/sam/$base\_aligned.sam
    bam=results/bam/$base\_aligned.bam
    sorted_bam=results/bam/$base\_aligned_sorted.bam
    raw_bcf=results/bcf/$base\_raw.bcf
    variants=results/bcf/$base\_variants.bcf
    final_variants=results/vcf/$base\_final_variants.vcf  

    bwa aln $genome $fq > $sai
    bwa samse $genome $sai $fq > $sam
    samtools view -S -b $sam > $bam
    samtools sort -f $bam $sorted_bam
    samtools index $sorted_bam
    samtools mpileup -g -f $genome $sorted_bam > $raw_bcf
    bcftools view -bvcg $raw_bcf > $variants
    bcftools view $variants | /usr/share/samtools/vcfutils.pl varFilter - > $final_variants
    done
~~~
{: .output}

> ## Exercise
> It's a good idea to add comments to your code so that you (or a collaborator) can make sense of what you did later. 
> Look through your existing script. Discuss with a neighbor where you should add comments. Add comments (anything following
> a `#` character will be interpreted as a comment, bash will not try to run these comments as code). 
{: .challenge}

Now we can run our script:

~~~
$ bash run_variant_calling_2.sh
~~~
{: .bash}

> ## Exercise
> 
>
>
{: .challenge}


> ## BWA variations
> BWA is a software package for mapping low-divergent sequences 
> against a large reference genome, such as the human genome, and 
> it's freely available [here](http://bio-bwa.sourceforge.net). It 
> consists of three algorithms: BWA-backtrack, BWA-SW and BWA-MEM, 
> each being invoked with different sub-commands: `aln + samse + sampe` for BWA-backtrack, `bwasw` for BWA-SW and `mem` for the 
> BWA-MEM algorithm. BWA-backtrack is designed for Illumina sequence reads up to 100bp, while the rest two are better fitted for 
> longer sequences ranged from 70bp to 1Mbp. A general rule of thumb is to use `bwa mem` for reads longer than 70 bp, whereas 
> `bwa aln` has a moderately higher mapping rate and a shorter run 
> time for short reads (~36bp). You can find a more indepth discussion in the [bwa doc page](http://bio-bwa.sourceforge.net/bwa.shtml) as well as in this 
> [blog post](http://crazyhottommy.blogspot.ca/2017/06/bwa-aln-or-bwa-mem-for-short-reads-36bp.html).
> In this lesson we have been using the `aln` for performing the 
> alignment, but the same process can be performed with `bwa mem` which doesn't require the creation of the index files. The 
> process is modified starting from `mkdir` step, and omitting all directories relevant to the `.sai` index files, i.e.:
> 
> *Create output paths for various intermediate and result files.*
>
>    $ mkdir -p results/sam
>    $ mkdir -p results/bam
>    $ mkdir -p results/bcf
>    $ mkdir -p results/vcf
>
>
> *Assign file names to variables*
>
> $ fq=data/trimmed_fastq/$base\.fastq
> $ sam=results/sam/$base\_aligned.sam
> $ bam=results/bam/$base\_aligned.bam
> $ sorted_bam=results/bam/$base\_aligned_sorted.bam
> $ raw_bcf=results/bcf/$base\_raw.bcf
> $ variants=results/bcf/$base\_variants.bcf
> $ final_variants=results/vcf/$base\_final_variants.vcf  
>
> *Run the alignment*
> 
> bwa mem -M $genome $fq > $sam
> 
> As an exercise, try and change your existing script file, from using the `aln` method to the `mem` method.
{: .callout}





