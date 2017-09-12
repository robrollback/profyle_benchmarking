# PROFYLE - Pan-Canadian Cancer Benchmarking
Benchmarking cancer tools for pipeline harmonization

This repo will contain all the relevant information to perform cross center benchmarking for particular aspects of the cancer analysis

PHASE 1: SNVs and Indels
========================

##NA12878/NA24385 tumor-like mixture

This is a sequenced mixture dataset of two [Genome in a Bottle] (http://jimb.stanford.edu/giab/) (GIAB) samples:

- [NA12878] (https://catalog.coriell.org/0/Sections/Search/Sample_Detail.aspx?Ref=GM12878&product=CC)
- [NA24385] (https://catalog.coriell.org/0/Sections/Search/Sample_Detail.aspx?Ref=GM24385&product=CC)

The mixture simulates a tumor and normal cancer dataset for validation of low
frequency calling by somatic variant callers. NA12878 variants are the "somatic"
variations, and shared NA12878/NA24385 variants are the "germline" background.
The directory contains:

- 90x "tumor" whole genome fastqs (paired, 150bp) generated by mixing 30%
  NA12878 with 70% NA24385. The samples are physically mixed at these ratios
  prior to sequencing. We expect somatic variants at 15% (unique heterozygotes
  in NA12878) and 30% (unique homozygotes in NA12878).
  
- A 30x "normal" whole genome fastq (paired, 150bp) of NA24385

- A truth set generated from Genome in a Bottle 3.2.2 calls for NA12878/NA24385

The experimental design, sample preparation and sequencing of this set was done
by Edwin Cuppen (Hartwig Medical Foundation, Amsterdam, the Netherlands). Isaac
J Nijman (Utrecht Medical Center, Utrecht, the Netherlands and Center for
Personalized Cancer Treatment) developed the HMF pipeline and collaborated with
Brad in bioinformatics analyses. Brad Chapman at Harvard Chan School provided
the truth set.

Data will be provided to each bioinformatican involve via email

This work is licensed under a [Creative Commons Attribution-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-sa/3.0/deed.en_US). This means that you are able to copy, share and modify the work, as long as the result is distributed under the same license.

## Setup

To expedite comparison, we will provide the full processed bam file generated by McGill's GATK best practices pipeline.

![Data processing diagram](img/dnaseq_pipeline.jpg)

As depicted in the diagram above the ceph mixture was processed as follows:
1. Adapter and quality filtering of raw reads with skewer 0.2.2
2. Alignment of trimmed reads to GRCh37.p13 with bwa+mem 0.7.12
3. GATK 3.6 Indel realignment using both samples to refine indel regions
4. Mark duplicates with sambamba 0.6.5
5.  Base recalibration of each samples with GATK 3.6

The initial structure of your folders should look like this:
```
<ROOT>
|--snv_indel
    `---final_bam
        `---ceph_mixture_normal
        `---ceph_mixture_tumor
    
```

### Goal

Each center will upload the ceph mixture locally and run their variant caller of interest on the provide final bam

Each center will then provide their raw vcf file for benchmark assessment following well established benchmarking paradigm i.e. similar to the CGEN benchmarking

Real time genomics (RTG) vcfeval will be run locally by McGill using same arguments and truth set provided by the Brad Chapman from Havard University

The following data will be generated and the table below will be filled out and publically available: 

| True-pos | False-pos | False-neg | Precision | Sensitivity | F-measure |
|:-------- |:--------- |:--------- |:--------- |:----------- |:--------- |


### Required files for assessment
1. Final raw vcf.

### Contact info
For any questions or concerns regarding information contained within please contact: 

Robert Eveleigh: robert.eveleigh@mcgill.ca

Mathieu Bourgey: mathieu.bourgey@mcgill.ca

Guillaume Bourque: guil.bourque@mcgill.ca

