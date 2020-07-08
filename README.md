#sdm - simple demultiplexer (and more)
sdm is a bioinformatics tool to manipulate sequencing output (from 454 or Illumina machines), demultiplex it using a mapping file, perform manipulation steps focused for amplicon sequencing (e.g. dereplication)

Overview
sdm requires a set of parameters depending on the task

Input files (FASTQ files, FNA + QUAL datasets; paired/single end ...)
Output files or destination directory
Mapping file (required for demultiplexing only)
Manipulation parameters (optional, to specify quality filtering and dereplication steps)
Option file (optional: it's a text file with a list of parameters, useful to re-apply a complex set of parameters to multiple dataset)
Input files
-i_path (path): specify the input directory containing a set of FASTQ/FASTA files
-i (file): path to the input file (to specify paired ends, separate their filenames with a comma like file_1.fq,file_2.fq). FASTA/FASTQ format will be autodetected.
-i_fastq (file): path to the input FASTQ file
-i_fna (file): path to the input file in FASTA format
-i_qual (file): path to the corresponding qual file (paired with -i_fna.
-i_MID_fastq (file): FASTQ file with the barcode sequences; if paired reads are supplied with -i_fna/-i_fastq and the MID identifier via -i_MID_fastq, paired has to be set to 2. If e.g. merged reads are supplied + mids, -paired has to be set to 1.
Output files
-o_fna (file): output FASTA file
-o_qual (file): output quality file (for the corresponding -o_fna)
-o_fastq (file): output FASTQ file (will override the previous two)
-options (optional)
-log (optional). Set to "nolog" to deactivate alltogether.
-o_demultiplex (file): write a single demultiplexed file
Mapping file
-map (file): path to the mapping file in Qiime classic format (optional)
The mapping file is a TSV file containing these columns (in the first line, that should start with "#"):

SampleID: sample identifier
BarcodeSequence
LinkerPrimerSequence (can be empty as sdm will not use it)
ReversePrimer (can be empty)
fastqFile (path to the input file or files for paired reads, in this case separated with a comma)
MIDfqFile (path to the optional barcode file)
TechnicalReplicates (integer used to pair technical replicates)
Manipulation parameters (actions)
-sample_sep (string): separator to separate sample and ID (optional, default:"__")

-paired (1/2/3):

1: not paired (default)
2: input is paired (specify two input files separated by a comma)
3: input is paired and the barcodes is in a separate FASTQ file (specify two input files separated by a comma, plus -i_MID_fastq)
-onlyPair (file): [1/2] consider only read pair 1 or 2. Useful when streamlining inputs (LotuS) or considering double barcoding.

-oneLineFastaFormat (0/1): write FASTQ (and quality) file sequence string in one line, opposed to default 80 characters per line.

-o_dereplicate (file): output file with dereplicated sequences (will write the size, number of starting sequences, in the sequence header)

-dere_size_fmt (0/1): (0) usearch format "size=X;" or (1) "_X"

-min_derep_copies (int): only print seq if at least copies present. Can be complex terms like "10:1,3:3", meaning at least 10x in 1 sample or 3x in 3 different samples. See also this document for more information.

-SyncReadPairs (T/F): sdm can check, if read pairs occur in the same (correct) order in the input files, and correct this in case not (T).

-maxReadsPerOutput (file): number of filtered reads in output files. If more reads, a new file is created. Only works with -o_fna

-mergedPairs (1/0): 1: paired sequences were merged externally (e.g. with FLASH), important for assumption that read quality is deteoriating.

-OTU_fallback (file): Fallback FASTA sequences for OTU's, only used in SEED extension mode

-i_qual_offset (0-64): FASTQ offset for quality values. Set this to '0' or 'auto' if you are unsure which fastq version is being used (default: read from sdm option file)

-o_qual_offset (0-64): set quality offset for fastq outfile. Default: 33

-ignore_IO_errors (0/1): 1 = Errors in fastq reads are ignored, with sdm trying to sync reads pairs after corrupted single reads (default: 0)

Option files
The command sdm -help_options will print a demo option file. The option files can contain comments (lines beginning with "#") and parameters. A small example is:

#Parameters starting with "*" refer to the barcode sequences
minSeqLength    170
maxSeqLength    1000
minAvgQuality   27
*minSeqLength   170
*minAvgQuality  20
 
#Ambiguous bases in Sequence
maxAmbiguousNT  0
*maxAmbiguousNT 1

#Filter whole sequence if one window of quality scores is below average
QualWindowWidth 50
QualWindowThreshhold    25

#Trim the end of a sequence if a window falls below quality threshhold. 
#Useful for removing low qulaity trailing ends of sequence
TrimWindowWidth 20
TrimWindowThreshhold    25

#Max Barcode Errors
maxBarcodeErrs  0
maxPrimerErrs   0

Examples
Demultiplex a "454" run using a mapping file (assumes that the .qual file is called test.fna):

sdm -i test.fna -map mapping.txt 
Demultiplex an Illumina run using a mapping file and a barcode sequence file:

sdm -i forward.fq,reverse.fq -i_MID_fastq barcodes.fq -paired 2 \
  -o_demultiplex reads/ -map mapping.txt \
  -log demultiplex -options miseq.txt
Getting sdm
sdm is bundled with Lotus, but you can now easily get it with Conda:

conda install -c bioconda sdm