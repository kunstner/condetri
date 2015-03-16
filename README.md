# condetri
Automatically exported from code.google.com/p/condetri

##Summary
Trim reads from the 3'-end and extract reads (or read pairs) of good quality. If the reads are paired, the filtering is done pairwise, and if one read in a pair has low quality, the remaining read is saved as single end.


##Usage
perl condetri.pl -fastq1=file1 [-fastq2=file2 -prefix=s -cutfirst=i -hq=i -lq=i -frac=[0,1] -minlen=i -mh=i -ml=i -sc=i -rmN]


##Description
**The trimming is performed in two steps:**

**(1)** Trimming low quality bases from the 3'-end

**(2)** Overall quality check of read/pair

**Details:**

**(1)** Bases are removed from the 3'-end if the quality score is lower than some threshold (*hq*). When a base with higher quality is reached, it is kept temporarily and the preceding bases are considered. After this point, also bases with low quality can be saved, if they are surrounded by high quality bases. Up to *ml* consecutive bases are saved temporarily, but if the following base also has low quality, all temporarily saved bases are removed, and the trimming starts all over again. The trimming stops either when finding *mh* consecutive high quality bases, or when the read is trimmed down to a certain length (*minlen*).

**(2)** After the trimming step, the quality scores of the remaining read are controlled for. A read is approved if a certain fraction (*frac*) of the bases have a quality score higher than *hq*, and there is no base in the read that has a quality score below some lower bound (*lq*). If the input is paired-end reads, both reads in a pair must be approved for keeping the pair. If only one of the reads is approved, it is saved to an additional, unpaired file which can be used as single-end data.


###Comments
The parameters -cutfirst and -rmN was added to ConDeTri? in version 2.0 (for details, see release notes). If -cutfirst is used, -rmN is somewhat needless (it will still scan for non ATGC-characters from the 5' end, but the first i bases has already been removed). Also keep in mind that these choices will affect the minlen-parameter. For example if you have 75bp read, set the -minlen=50 and use the -cutfirst=6, the maximum possible number of bases that can be trimmed from the 3' end is 19 (instead of 25 without -cutfirst). Hence one might want to decrease the -minlen when using -cutfirst.


###Input parameters
(default values in brackets [])
<pre>
 -fastq1=file   Fastq file. If a second file is given, the files are trimmed as
 -fastq2=file    a pair. The reads must have the same order in both files.
 -prefix=string Prefix for the output file(s). The filtered fastq file(s) will be
                 named prefix_trim1.fastq (and prefix_trim2.fastq if present).
                 For pairs, a third file will be given with unpaired reads (reads 
                 from pairs where one low quality read has been removed).
 -cutfirst=i    Removes the first i base from the 5' end before trimming [0].
 -rmN           Removes N from the 5' end before trimming[no].
 -hq=i          Hiqh quality threshold [25].
 -lq=i          Low quality threshold [10].
 -frac=[0,1]    Fraction of read that must exceed hq [0.8].
 -minlen=i      Min allowed read length [50].
 -mh=i          When this no of consecutive hq bases is reached, the trimming stops [5].
 -ml=i          Max no of lq bases allowed after a stretch of hq bases from 3'-end [1].
 -sc=i          Illumina scoring table: Score=ASCII-sc. Illumina GAII (for which ConDeTri
                 was written) used 64, newer Illumina data uses 33, which is Sanger
                 standard. Can be set to any other integer if wanted [64].
 -q             Prints Illumina scoring table.
 -h             Prints a help message.
 </pre>

###Output files
<pre>
 prefix_trim1.fastq             File(s) with the trimmed reads (one file for
 prefix_trim2.fastq              single-end data, three for paired-end data, where
 prefix_trim_unpaired.fastq      the last file includes reads from the two input
                                 files whose read pair had too poor quality.
 prefix.stats                   Includes basic statistics in columns.
 </pre>
**The columns for the .stats file are the following:**<BR>
1) PREFIX <BR>
2) NUMBER OF READS IN ORIGINAL FILE(S) <BR>
3) NUMBER OF BASES IN ORIGINAL FILE(S) <BR>
4) NO OF PAIRED READS AFTER TRIMMING <BR>
5) NO OF BASES IN PAIRS AFTER TRIMMING  <BR>
6) NO OF UNPAIRED READS AFTER TRIMMING <BR>
7) NO OF UNPAIRED BASES AFTER TRIMMING <BR>

The .stats files are suitable for concatenation to make a summary table for several fastq files, for example one file for each lane in a flowcell or a set of transcriptome samples. Just use:

$ cat *.stats
