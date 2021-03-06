Whole-genome sequencing and targeted amplicon capture
=====================================================

CNVkit is primarily designed for use on **hybrid capture** sequencing data,
where off-target reads are present and can be used improve copy number
estimates. However, CNVkit can also be used on **whole-genome sequencing** (WGS)
and **targeted amplicon sequencing** (TAS) datasets by using alternative
command-line options.

The :ref:`batch` command supports these workflows through the
``-m``/``--method`` option.


.. _wgs:

Whole-Genome Sequencing (WGS)
-----------------------------

CNVkit treats WGS data as a capture of all of the genome's sequencing-accessible
regions, with no off-target regions.

The ``batch --method wgs`` option uses the given reference genome's
sequencing-accessible regions ("access" BED) as the "targets" -- these will be
calculated on the fly if not provided. No "antitarget" regions are used.
Since the input does not contain useful per-target gene labels, a  gene
annotation database is required and used to label genes in the outputs::

    cnvkit.py batch -m wgs -g data/access-5kb-mappable.hg19.bed --annotate refFlat.txt *.bam

Equivalently::

    cnvkit.py target data/access-5kb-mappable.hg19.bed --split --short-names --annotate refFlat.txt -o targets.bed
    # For each sample
    cnvkit.py coverage Sample.bam targets.bed -p 0 -o Sample.targetcoverage.cnn
    # Create an empty antitarget coverage file, header only
    head -n1 Sample.targetcoverage.cnn -o Sample.antitargetcoverage.cnn
    cnvkit.py reference *.targetcoverage.cnn *.antitargetcoverage.cnn -o ref-wgs.cnn
    cnvkit.py fix Sample.targetcoverage.cnn Sample.antitargetcoverage.cnn ref-wgs.cnn --no-edge

To speed up WGS analyses, try any or all of the following:

- Instead of analyzing the whole genome, use the "access" or "target" BED file
  to limit the analysis to just the genic regions. You can get such a BED file
  from the [UCSC Genome Browser](https://genome.ucsc.edu/cgi-bin/hgTables), for
  example.
- Increase the "target" average bin size to 500 or 1000 bases.
- Specify a smaller p-value threshold (``segment -t``). For the CBS method,
  ``1e-6`` may work well.
- Use the ``-p/--processes`` option in the :ref:`batch`, :ref:`coverage` and
  :ref:`segment` commands to ensure all available CPUs are used.
- Ensure you are using the most recent version of CNVkit. Each release includes
  some performance improvements.

The ``batch -m wgs`` option does all of these except the first automatically.


.. _tas:

Targeted Amplicon Sequencing (TAS)
----------------------------------

When amplicon sequencing is used as a targeted capture method, no off-target
reads are sequenced. While this limits the copy number information available in
the sequencing data versus hybrid capture, CNVkit can analyze TAS data using
only on-target coverages and excluding all off-target regions from the analysis.

The ``batch -m amplicon`` option uses the given targets to infer coverage, and
leaves the antitarget coverage file empty::

    cnvkit.py batch -m amplicon -t targets.bed *.bam

Equivalently::

    cnvkit.py target targets.bed --split -o targets.split.bed
    # For each sample
    cnvkit.py coverage Sample.bam targets.split.bed -p 0 -o Sample.targetcoverage.cnn
    # Create an empty antitarget coverage file, header only
    head -n1 Sample.targetcoverage.cnn -o Sample.antitargetcoverage.cnn
    cnvkit.py reference *.targetcoverage.cnn *.antitargetcoverage.cnn -o ref-tas.cnn
    cnvkit.py fix Sample.targetcoverage.cnn Sample.antitargetcoverage.cnn ref-tas.cnn --no-edge

This approach does not collect any copy number information between targeted
regions, so it should only be used if you have in fact prepared your samples
with a targeted amplicon sequencing protocol. It also does not attempt to
normalize each amplicon at the gene level, though this may be addressed in a
future version of CNVkit.
