README for RSEM
===============

[Bo Li](http://bli25ucb.github.io/) \(bli at cs dot wisc dot edu\)

* * *

Table of Contents
-----------------

* [Introduction](#introduction)
* [Compilation & Installation](#compilation)
* [Usage](#usage)
  * [Build RSEM references using RefSeq, Ensembl, or GENCODE annotations](#built)
* [Example](#example)
* [Simulation](#simulation)
* [Generate Transcript-to-Gene-Map from Trinity Output](#gen_trinity)
* [Differential Expression Analysis](#de)
* [Authors](#authors)
* [Acknowledgements](#acknowledgements)
* [License](#license)

* * *

## <a name="introduction"></a> Introduction

RSEM is a software package for estimating gene and isoform expression
levels from RNA-Seq data. The RSEM package provides an user-friendly
interface, supports threads for parallel computation of the EM
algorithm, single-end and paired-end read data, quality scores,
variable-length reads and RSPD estimation. In addition, it provides
posterior mean and 95% credibility interval estimates for expression
levels. For visualization, It can generate BAM and Wiggle files in
both transcript-coordinate and genomic-coordinate. Genomic-coordinate
files can be visualized by both UCSC Genome browser and Broad
Institute's Integrative Genomics Viewer (IGV). Transcript-coordinate
files can be visualized by IGV. RSEM also has its own scripts to
generate transcript read depth plots in pdf format. The unique feature
of RSEM is, the read depth plots can be stacked, with read depth
contributed to unique reads shown in black and contributed to
multi-reads shown in red. In addition, models learned from data can
also be visualized. Last but not least, RSEM contains a simulator.

## <a name="compilation"></a> Compilation & Installation

To compile RSEM, simply run
   
    make

For cygwin users, please uncomment the 3rd and 7th line in
'sam/Makefile' before you run 'make'.

To compile EBSeq, which is included in the RSEM package, run

    make ebseq

To install, simply put the rsem directory in your environment's PATH
variable.

If you prefer to put all RSEM executables to a bin directory, please
also remember to put 'rsem_perl_utils.pm' and 'WHAT_IS_NEW' to the
same bin directory. 'rsem_perl_utils.pm' is required for most RSEM's
perl scripts and 'WHAT_IS_NEW' contains the RSEM version information.

### Prerequisites

C++, Perl and R are required to be installed. 

To use the '--gff3' option of 'rsem-prepare-reference', Python is also
required to be installed.

To take advantage of RSEM's built-in support for the Bowtie/Bowtie
2/STAR alignment program, you must have
[Bowtie](http://bowtie-bio.sourceforge.net)/[Bowtie
2](http://bowtie-bio.sourceforge.net/bowtie2)/[STAR](https://github.com/alexdobin/STAR)
installed.

## <a name="usage"></a> Usage

### I. Preparing Reference Sequences

RSEM can extract reference transcripts from a genome if you provide it
with gene annotations in a GTF file.  Alternatively, you can provide
RSEM with transcript sequences directly.

Please note that GTF files generated from the UCSC Table Browser do not
contain isoform-gene relationship information.  However, if you use the
UCSC Genes annotation track, this information can be recovered by
downloading the knownIsoforms.txt file for the appropriate genome.
 
To prepare the reference sequences, you should run the
'rsem-prepare-reference' program.  Run 

    rsem-prepare-reference --help

to get usage information or visit the [rsem-prepare-reference
documentation page](rsem-prepare-reference.html).

#### <a name="built"></a> Built

### II. Calculating Expression Values

To calculate expression values, you should run the
'rsem-calculate-expression' program.  Run 

    rsem-calculate-expression --help

to get usage information or visit the [rsem-calculate-expression
documentation page](rsem-calculate-expression.html).

#### Calculating expression values from single-end data

For single-end models, users have the option of providing a fragment
length distribution via the '--fragment-length-mean' and
'--fragment-length-sd' options.  The specification of an accurate fragment
length distribution is important for the accuracy of expression level
estimates from single-end data.  If the fragment length mean and sd are
not provided, RSEM will not take a fragment length distribution into
consideration.

#### Using an alternative aligner

By default, RSEM automates the alignment of reads to reference
transcripts using the Bowtie aligner. Turn on '--bowtie2' for
'rsem-prepare-reference' and 'rsem-calculate-expression' will allow
RSEM to use the Bowtie 2 alignment program instead. Please note that
indel alignments, local alignments and discordant alignments are
disallowed when RSEM uses Bowtie 2 since RSEM currently cannot handle
them. See the description of '--bowtie2' option in
'rsem-calculate-expression' for more details. Similarly, turn on
'--star' will allow RSEM to use the STAR aligner. To use an
alternative alignment program, align the input reads against the file
'reference_name.idx.fa' generated by 'rsem-prepare-reference', and
format the alignment output in SAM or BAM format.  Then, instead of
providing reads to 'rsem-calculate-expression', specify the '--sam' or
'--bam' option and provide the SAM or BAM file as an argument.

RSEM requires the alignments of a read to be adjacent. For
paired-end reads, RSEM also requires the two mates of any alignment be
adjacent. To check if your SAM/BAM file satisfy the requirements,
please run

    rsem-sam-validator <input.sam/input.bam>

If your file does not satisfy the requirements, you can use
'convert-sam-for-rsem' to convert it into a BAM file which RSEM can
process. Please run
 
    convert-sam-for-rsem --help

to get usage information or visit the [convert-sam-for-rsem
documentation
page](convert-sam-for-rsem.html).

However, please note that RSEM does ** not ** support gapped
alignments. So make sure that your aligner does not produce alignments
with intersions/deletions. Also, please make sure that you use
'reference_name.idx.fa' , which is generated by RSEM, to build your
aligner's indices.

### III. Visualization

RSEM contains a version of samtools in the 'sam' subdirectory. RSEM
will always produce three files:'sample_name.transcript.bam', the
unsorted BAM file, 'sample_name.transcript.sorted.bam' and
'sample_name.transcript.sorted.bam.bai' the sorted BAM file and
indices generated by the samtools included. All three files are in
transcript coordinates. When users specify the --output-genome-bam
option RSEM will produce three files: 'sample_name.genome.bam', the
unsorted BAM file, 'sample_name.genome.sorted.bam' and
'sample_name.genome.sorted.bam.bai' the sorted BAM file and indices
generated by the samtools included. All these files are in genomic
coordinates.

#### a) Converting transcript BAM file into genome BAM file

Normally, RSEM will do this for you via '--output-genome-bam' option
of 'rsem-calculate-expression'. However, if you have run
'rsem-prepare-reference' and use 'reference_name.idx.fa' to build
indices for your aligner, you can use 'rsem-tbam2gbam' to convert your
transcript coordinate BAM alignments file into a genomic coordinate
BAM alignments file without the need to run the whole RSEM
pipeline.

Usage:

    rsem-tbam2gbam reference_name unsorted_transcript_bam_input genome_bam_output

reference_name	   		  : The name of reference built by 'rsem-prepare-reference'				
unsorted_transcript_bam_input	  : This file should satisfy: 1) the alignments of a same read are grouped together, 2) for any paired-end alignment, the two mates should be adjacent to each other, 3) this file should not be sorted by samtools 
genome_bam_output		  : The output genomic coordinate BAM file's name

#### b) Generating a Wiggle file

A wiggle plot representing the expected number of reads overlapping
each position in the genome/transcript set can be generated from the
sorted genome/transcript BAM file output.  To generate the wiggle
plot, run the 'rsem-bam2wig' program on the
'sample_name.genome.sorted.bam'/'sample_name.transcript.sorted.bam' file.

Usage:    

    rsem-bam2wig sorted_bam_input wig_output wiggle_name [--no-fractional-weight]

sorted_bam_input        : Input BAM format file, must be sorted  
wig_output              : Output wiggle file's name, e.g. output.wig  
wiggle_name             : The name of this wiggle plot  
--no-fractional-weight  : If this is set, RSEM will not look for "ZW" tag and each alignment appeared in the BAM file has weight 1. Set this if your BAM file is not generated by RSEM. Please note that this option must be at the end of the command line

#### c) Loading a BAM and/or Wiggle file into the UCSC Genome Browser or Integrative Genomics Viewer(IGV)

For UCSC genome browser, please refer to the [UCSC custom track help page](http://genome.ucsc.edu/goldenPath/help/customTrack.html).

For integrative genomics viewer, please refer to the [IGV home page](http://www.broadinstitute.org/software/igv/home). Note: Although IGV can generate read depth plot from the BAM file given, it cannot recognize "ZW" tag RSEM puts. Therefore IGV counts each alignment as weight 1 instead of the expected weight for the plot it generates. So we recommend to use the wiggle file generated by RSEM for read depth visualization.

Here are some guidance for visualizing transcript coordinate files using IGV:

1) Import the transcript sequences as a genome 

Select File -> Import Genome, then fill in ID, Name and Fasta file. Fasta file should be 'reference_name.idx.fa'. After that, click Save button. Suppose ID is filled as 'reference_name', a file called 'reference_name.genome' will be generated. Next time, we can use: File -> Load Genome, then select 'reference_name.genome'.

2) Load visualization files

Select File -> Load from File, then choose one transcript coordinate visualization file generated by RSEM. IGV might require you to convert wiggle file to tdf file. You should use igvtools to perform this task. One way to perform the conversion is to use the following command:

    igvtools tile reference_name.transcript.wig reference_name.transcript.tdf reference_name.genome   
 
#### d) Generating Transcript Wiggle Plots

To generate transcript wiggle plots, you should run the
'rsem-plot-transcript-wiggles' program.  Run 

    rsem-plot-transcript-wiggles --help

to get usage information or visit the [rsem-plot-transcript-wiggles
documentation page](rsem-plot-transcript-wiggles.html).

#### e) Visualize the model learned by RSEM

RSEM provides an R script, 'rsem-plot-model', for visulazing the model learned.

Usage:
    
    rsem-plot-model sample_name output_plot_file

sample_name: the name of the sample analyzed    
output_plot_file: the file name for plots generated from the model. It is a pdf file    

The plots generated depends on read type and user configuration. It
may include fragment length distribution, mate length distribution,
read start position distribution (RSPD), quality score vs observed
quality given a reference base, position vs percentage of sequencing
error given a reference base and histogram of reads with different
number of alignments.

fragment length distribution and mate length distribution: x-axis is fragment/mate length, y axis is the probability of generating a fragment/mate with the associated length

RSPD: Read Start Position Distribution. x-axis is bin number, y-axis is the probability of each bin. RSPD can be used as an indicator of 3' bias

Quality score vs. observed quality given a reference base: x-axis is Phred quality scores associated with data, y-axis is the "observed quality", Phred quality scores learned by RSEM from the data. Q = -10log_10(P), where Q is Phred quality score and P is the probability of sequencing error for a particular base

Position vs. percentage sequencing error given a reference base: x-axis is position and y-axis is percentage sequencing error

Histogram of reads with different number of alignments: x-axis is the number of alignments a read has and y-axis is the number of such reads. The inf in x-axis means number of reads filtered due to too many alignments
 
## <a name="example"></a> Example

Suppose we download the mouse genome from UCSC Genome Browser.  We do
not add poly(A) tails and use '/ref/mouse_0' as the reference name.
We have a FASTQ-formatted file, 'mmliver.fq', containing single-end
reads from one sample, which we call 'mmliver_single_quals'.  We want
to estimate expression values by using the single-end model with a
fragment length distribution. We know that the fragment length
distribution is approximated by a normal distribution with a mean of
150 and a standard deviation of 35. We wish to generate 95%
credibility intervals in addition to maximum likelihood estimates.
RSEM will be allowed 1G of memory for the credibility interval
calculation.  We will visualize the probabilistic read mappings
generated by RSEM on UCSC genome browser. We will generate a list of
genes' transcript wiggle plots in 'output.pdf'. The list is
'gene_ids.txt'. We will visualize the models learned in
'mmliver_single_quals.models.pdf'

The commands for this scenario are as follows:

    rsem-prepare-reference --gtf mm9.gtf --mapping knownIsoforms.txt --bowtie --bowtie-path /sw/bowtie /data/mm9 /ref/mouse_0
    rsem-calculate-expression --bowtie-path /sw/bowtie --phred64-quals --fragment-length-mean 150.0 --fragment-length-sd 35.0 -p 8 --output-genome-bam --calc-ci --memory-allocate 1024 /data/mmliver.fq /ref/mouse_0 mmliver_single_quals
    rsem-bam2wig mmliver_single_quals.sorted.bam mmliver_single_quals.sorted.wig mmliver_single_quals
    rsem-plot-transcript-wiggles --gene-list --show-unique mmliver_single_quals gene_ids.txt output.pdf 
    rsem-plot-model mmliver_single_quals mmliver_single_quals.models.pdf

## <a name="simulation"></a> Simulation

RSEM provides users the 'rsem-simulate-reads' program to simulate RNA-Seq data based on parameters learned from real data sets. Run

    rsem-simulate-reads

to get usage information or read the following subsections.
 
### Usage: 

    rsem-simulate-reads reference_name estimated_model_file estimated_isoform_results theta0 N output_name [-q]

__reference_name:__ The name of RSEM references, which should be already generated by 'rsem-prepare-reference'   	     

__estimated_model_file:__ This file describes how the RNA-Seq reads will be sequenced given the expression levels. It determines what kind of reads will be simulated (single-end/paired-end, w/o quality score) and includes parameters for fragment length distribution, read start position distribution, sequencing error models, etc. Normally, this file should be learned from real data using 'rsem-calculate-expression'. The file can be found under the 'sample_name.stat' folder with the name of 'sample_name.model'. 'model_file_description.txt' provides the format and meanings of this file.    

__estimated_isoform_results:__ This file contains expression levels for all isoforms recorded in the reference. It can be learned using 'rsem-calculate-expression' from real data. The corresponding file users want to use is 'sample_name.isoforms.results'. If simulating from user-designed expression profile is desired, start from a learned 'sample_name.isoforms.results' file and only modify the 'TPM' column. The simulator only reads the TPM column. But keeping the file format the same is required. If the RSEM references built are aware of allele-specific transcripts, 'sample_name.alleles.results' should be used instead.   

__theta0:__ This parameter determines the fraction of reads that are coming from background "noise" (instead of from a transcript). It can also be estimated using 'rsem-calculate-expression' from real data. Users can find it as the first value of the third line of the file 'sample_name.stat/sample_name.theta'.   

__N:__ The total number of reads to be simulated. If 'rsem-calculate-expression' is executed on a real data set, the total number of reads can be found as the 4th number of the first line of the file 'sample_name.stat/sample_name.cnt'.   

__output_name:__ Prefix for all output files.   

__--seed seed:__ Set seed for the random number generator used in simulation. The seed should be a 32-bit unsigned integer.

__-q:__ Set it will stop outputting intermediate information.   

### Outputs:

output_name.sim.isoforms.results, output_name.sim.genes.results: Expression levels estimated by counting where each simulated read comes from.
output_name.sim.alleles.results: Allele-specific expression levels estimated by counting where each simulated read comes from.

output_name.fa if single-end without quality score;   
output_name.fq if single-end with quality score;   
output_name_1.fa & output_name_2.fa if paired-end without quality
score;   
output_name_1.fq & output_name_2.fq if paired-end with quality score.   

**Format of the header line**: Each simulated read's header line encodes where it comes from. The header line has the format:

    {>/@}_rid_dir_sid_pos[_insertL]

__{>/@}:__ Either '>' or '@' must appear. '>' appears if FASTA files are generated and '@' appears if FASTQ files are generated

__rid:__ Simulated read's index, numbered from 0   

__dir:__ The direction of the simulated read. 0 refers to forward strand ('+') and 1 refers to reverse strand ('-')   

__sid:__ Represent which transcript this read is simulated from. It ranges between 0 and M, where M is the total number of transcripts. If sid=0, the read is simulated from the background noise. Otherwise, the read is simulated from a transcript with index sid. Transcript sid's transcript name can be found in the 'transcript_id' column of the 'sample_name.isoforms.results' file (at line sid + 1, line 1 is for column names)   

__pos:__ The start position of the simulated read in strand dir of transcript sid. It is numbered from 0   

__insertL:__ Only appear for paired-end reads. It gives the insert length of the simulated read.   

### Example:

Suppose we want to simulate 50 millon single-end reads with quality scores and use the parameters learned from [Example](#example). In addition, we set theta0 as 0.2 and output_name as 'simulated_reads'. The command is:

    rsem-simulate-reads /ref/mouse_0 mmliver_single_quals.stat/mmliver_single_quals.model mmliver_single_quals.isoforms.results 0.2 50000000 simulated_reads

## <a name="gen_trinity"></a> Generate Transcript-to-Gene-Map from Trinity Output

For Trinity users, RSEM provides a perl script to generate transcript-to-gene-map file from the fasta file produced by Trinity.

### Usage:

    extract-transcript-to-gene-map-from-trinity trinity_fasta_file map_file

trinity_fasta_file: the fasta file produced by trinity, which contains all transcripts assembled.    
map_file: transcript-to-gene-map file's name.    

## <a name="de"></a> Differential Expression Analysis

Popular differential expression (DE) analysis tools such as edgeR and
DESeq do not take variance due to read mapping uncertainty into
consideration. Because read mapping ambiguity is prevalent among
isoforms and de novo assembled transcripts, these tools are not ideal
for DE detection in such conditions.

EBSeq, an empirical Bayesian DE analysis tool developed in UW-Madison,
can take variance due to read mapping ambiguity into consideration by
grouping isoforms with parent gene's number of isoforms. In addition,
it is more robust to outliers. For more information about EBSeq
(including the paper describing their method), please visit [EBSeq's
website](http://www.biostat.wisc.edu/~ningleng/EBSeq_Package).


RSEM includes EBSeq in its folder named 'EBSeq'. To use it, first type

    make ebseq

to compile the EBSeq related codes. 

EBSeq requires gene-isoform relationship for its isoform DE
detection. However, for de novo assembled transcriptome, it is hard to
obtain an accurate gene-isoform relationship. Instead, RSEM provides a
script 'rsem-generate-ngvector', which clusters transcripts based on
measures directly relating to read mappaing ambiguity. First, it
calcualtes the 'unmappability' of each transcript. The 'unmappability'
of a transcript is the ratio between the number of k mers with at
least one perfect match to other transcripts and the total number of k
mers of this transcript, where k is a parameter. Then, Ng vector is
generated by applying Kmeans algorithm to the 'unmappability' values
with number of clusters set as 3. This program will make sure the mean
'unmappability' scores for clusters are in ascending order. All
transcripts whose lengths are less than k are assigned to cluster
3. Run

    rsem-generate-ngvector --help

to get usage information or visit the [rsem-generate-ngvector
documentation
page](rsem-generate-ngvector.html).

If your reference is a de novo assembled transcript set, you should
run 'rsem-generate-ngvector' first. Then load the resulting
'output_name.ngvec' into R. For example, you can use 

    NgVec <- scan(file="output_name.ngvec", what=0, sep="\n")

. After that, set "NgVector = NgVec" for your differential expression
test (either 'EBTest' or 'EBMultiTest').


For users' convenience, RSEM also provides a script
'rsem-generate-data-matrix' to extract input matrix from expression
results:

    rsem-generate-data-matrix sampleA.[genes/isoforms].results sampleB.[genes/isoforms].results ... > output_name.counts.matrix

The results files are required to be either all gene level results or
all isoform level results. You can load the matrix into R by

    IsoMat <- data.matrix(read.table(file="output_name.counts.matrix"))

before running either 'EBTest' or 'EBMultiTest'.

Lastly, RSEM provides two scripts, 'rsem-run-ebseq' and
'rsem-control-fdr', to help users find differential expressed
genes/transcripts. First, 'rsem-run-ebseq' calls EBSeq to calculate related statistics
for all genes/transcripts. Run 

    rsem-run-ebseq --help

to get usage information or visit the [rsem-run-ebseq documentation
page](rsem-run-ebseq.html). Second,
'rsem-control-fdr' takes 'rsem-run-ebseq' 's result and reports called
differentially expressed genes/transcripts by controlling the false
discovery rate. Run

    rsem-control-fdr --help

to get usage information or visit the [rsem-control-fdr documentation
page](rsem-control-fdr.html). These
two scripts can perform DE analysis on either 2 conditions or multiple
conditions.

Please note that 'rsem-run-ebseq' and 'rsem-control-fdr' use EBSeq's
default parameters. For advanced use of EBSeq or information about how
EBSeq works, please refer to [EBSeq's
manual](http://www.bioconductor.org/packages/devel/bioc/vignettes/EBSeq/inst/doc/EBSeq_Vignette.pdf).

Questions related to EBSeq should
be sent to <a href="mailto:nleng@wisc.edu">Ning Leng</a>.

## <a name="authors"></a> Authors

[Bo Li](http://bli25ucb.github.io/) and [Colin Dewey](https://www.biostat.wisc.edu/~cdewey/) designed the RSEM algorithm. [Bo Li](http://bli25ucb.github.io/) implemented the RSEM software. [Peng Liu](https://www.biostat.wisc.edu/~cdewey/group.html) contributed the STAR aligner options.

## <a name="acknowledgements"></a> Acknowledgements

RSEM uses the [Boost C++](http://www.boost.org) and
[samtools](http://samtools.sourceforge.net) libraries. RSEM includes
[EBSeq](http://www.biostat.wisc.edu/~ningleng/EBSeq_Package/) for
differential expression analysis.

We thank earonesty and Dr. Samuel Arvidsson for contributing patches.

We thank Han Lin, j.miller, Jo&euml;l Fillon, Dr. Samuel G. Younkin and Malcolm Cook for suggesting possible fixes. 

## <a name="license"></a> License

RSEM is licensed under the [GNU General Public License
v3](http://www.gnu.org/licenses/gpl-3.0.html).
