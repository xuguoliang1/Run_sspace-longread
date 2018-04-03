.. |br| raw:: html

   <br />

.. _ref-example:

Examples
=========

.. _ref-exampleMecat:


Run_sspace-longread using mecat alignment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. **Prepare** SSPACE-Long input file, merging all pacbio raw reads:

  ::

    > cat *.subreads.fasta >pacbio.merge.subreads.bam.fasta


2. **Filter** the sequence longer than 100Kbp:

  .. code-block:: console

    > perl length_filter.pl \
    	--infile *.subreads.fasta \
    	--len 100000 \
    	--out 01.filter_reads

3. **Align** pacbio reads to reference genome parallelly using mecat and change the mecat output to blasr m1 format:

  .. code-block:: console

    > mecat2ref \
    		-d *.subreads.fasta.less100000.fa \
    		-r ref.fa \
    		-w 02.mecat_alignment/wrk_dir.1 \
    		-o 02.mecat_alignment/mecat2ref.1.out \
    		-t 16 -m 1 -x 0
    
     > perl change_alignment_mecat_to_blasr.ID.pl \
     		--query query.fasta \
    		--mecat_out mecat2ref.out \
    		--out macat_to_blasr.out

4. **Scaffold** the contigs using all the pacbio reads having more than 1 alignment:

  .. code-block:: console

    > perl SSPACE-LongRead.pl \
    		-c  ref.fa \
    		-p  pacbio.merge.subreads.bam.fasta \
    		-s  1 \
    		-b PacBio_scaffolder_results

.. _ref-examplemimimap2:


Run_sspace-longread using minimap2 alignment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before the alignment of minimap2, the **filer** step is unnecessary. Run_sspace-longread pipeline using minimap2 alignment has the same way in **Prepare** and **Scaffold** with mecat, so we only display **Align**.


1. **Align** pacbio reads to reference genome parallelly using minimap2 and change the minimap2 output to blasr m1 format:

  .. code-block:: console

    > minimap2 -cx map-pb ref.fa \
    		*.subreads.fasta \
    		-t 8 >*.subreads.fasta.paf \
    	& minimap2 -ax map-pb ref.fa \
    		*.subreads.fasta \
    		-t 8 >*.subreads.fasta.sam
    
     > perl change_alignment_minimap2_to_blasr.ID.pl \
     		--sam *.subreads.fasta.sam \
     		--paf *.subreads.fasta.paf \
     		--out *.subreads.fasta.alignment.out



.. _ref-exampleblasr:


Run_sspace-longread using blasr alignment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. **Prepare** SSPACE-Long input file, merging all pacbio raw reads:

  ::

    > cat *.subreads.fasta >pacbio.merge.subreads.bam.fasta


2. **Split** the pacbio reads per 1G and construct a precomputed suffix array for faster startup:

  .. code-block:: console
  
    > sawriter ref.sa ref.fa 

    > faSplit -verbose=2 about \
    		pacbio.merge.subreads.bam.fasta 1000000000 \
    		01.read_split/ 2>&1 | \
     		awk '{print $2}' > readsCut.list

3. **Align** pacbio reads to reference genome parallelly using blasr:

  .. code-block:: console

    > blasr *.fa ref.fa \
    		-sa ref.sa \
    		-nproc 8 \
    		-minMatch 5 \
    		-bestn 10 \
    		-noSplitSubreads \
    		-advanceExactMatches 1 \
    		-nCandidates 1 \
    		-maxAnchorsPerPosition 1 \
    		-sdpTupleSize 7 \
    		-out *.fa.m4 

4. **Scaffold** the contigs using all the pacbio reads having more than 1 alignment:

  .. code-block:: console

    > perl SSPACE-LongRead.pl \
    		-c  ref.fa \
    		-p  pacbio.merge.subreads.bam.fasta \
    		-s  1 \
    		-b PacBio_scaffolder_results
    		

