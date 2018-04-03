Quick start
==============

Overview
--------


Run_sspace-longread workflow consists of three main processing steps:

-  **Prepare**: Merging pacbio raw reads in fasta format
-  **Filter or Split**: Filterring the sequence longer than 100k, which will cause an error (Segmentation Fault), for mecat alignment, or splitting sequences for blasr alignment
-  **Align**: Alignment using blasr, mecat or minimap2
-  **Scaffold**: Linking the contigs using all the pacbio reads having more than 1 alignment
-  **Clean** Deleting big intermediate files.


Run_sspace-longread supports the following formats of sequencing data: ``fasta``, ``fastq``, ``fastq.gz``. 


Basic parameters
----------------

There are many parameters that user can change to adapt Run_sspace-longread for particular needs. While all these parameters are optional there is a set of parameters that are worth considering before running the analysis:


- ``--list`` sets the fasta file list of pacbio reads in absolute path. Required.
- ``--ref`` sets the fasta file of pre-assembled contig or scaffold. Required.
- ``--out`` sets the output file. default current directory (./).
- ``--ali`` sets the way of alignment (mecat|blasr|minimap2, default:mecat):

	- ``mecat`` **(default)** is generally suitable for majority of use cases. Applied on the **alignment** stage. Choice of the value for this parameter depends on the sequencing platform of library, and pacbio data is better than nanopore which generate partial reads longer than 100Kbp causing a running error (Segmentation fault).
	- ``minimap2`` is suitale for reads longer than 100k at comparable speed with mecat, Choice of the value for this parameter will be better for nanopore data.
	- ``blasr`` is slower than the above two methods and consume more computing resources.
	
- ``--step`` set the running steps(01234), default 0123.
	- step00: Merging pacbio raw reads in fasta format. 
	- step01: Filterring the sequence longer than 100k, which will cause an error (Segmentation Fault), for mecat alignment, or splitting sequences for blasr alignment. 
	- step02: Alignment using blasr, mecat or minimap2. 
	- step03: Linking the contigs using all the pacbio reads having more than 1 alignment. step04: Deleting big intermediate files.

The following sections describes common use cases


Default workflow
----------------

.. tip::
  Parameters used in this example are suitable for analysis of all alignment ways, **(mecat, minimap and blasr)**.

Run_sspace-longread can be used with the default parameters in most cases by executing the following sequence of commands:

.. code-block:: console

  > perl Run_sspace-longread.pl \
  	--list lib.list \
  	--ref contig.fasta \
  	--ali mecat \
  	--step 0123
  		
  ... Mecat alignments

  > perl SSPACE-LongRead.review.pl \
  	--list lib.list \
  	--ref contig.fasta \
  	--ali minimap2 \
  	--step 023

  ... minimap2 alignments 

  > perl SSPACE-LongRead.review.pl \
  	--list lib.list \
  	--ref contig.fasta \
  	--ali blasr \
  	--step 0123

  ... blasr alignments

