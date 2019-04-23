# PROFYLE - Pan-Canadian Cancer Benchmarking
Benchmarking cancer tools for pipeline harmonization

This repo will contain all the relevant information to perform cross center benchmarking for particular aspects of cancer analysis

PHASE 1: SNVs and Indels
------------------------

**NA12878/NA24385 tumor-like mixture**

This is a sequenced mixture dataset of two [Genome in a Bottle](http://jimb.stanford.edu/giab/) (GIAB) samples:

- [NA12878](https://catalog.coriell.org/0/Sections/Search/Sample_Detail.aspx?Ref=GM12878&product=CC)
- [NA24385](https://catalog.coriell.org/0/Sections/Search/Sample_Detail.aspx?Ref=GM24385&product=CC)

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

~~To expedite comparison, we will provide the full processed bam file generated by McGill's GATK best practices pipeline.~~

**Revision:** As requested in last weeks call the fastqs will be provided.

![Data processing diagram](img/dnaseq_pipeline.jpg)

As depicted in the diagram above the ceph mixture was processed as follows:
1. Adapter and quality filtering of raw reads with skewer 0.2.2
2. Alignment of trimmed reads to GRCh37.p13 with bwa+mem 0.7.12
3. GATK 3.6 Indel realignment using both samples to refine indel regions
4. Mark duplicates with sambamba 0.6.5
5. Base recalibration of each samples with GATK 3.6

The initial structure of your folders should look like this:
```
<ROOT>
|--profyle_benchmark
    `---snv_indel
        `---raw_data
            `---ceph_mixture_normal
            `---ceph_mixture_tumor
        `---final_bam
            `---ceph_mixture_normal
            `---ceph_mixture_tumor
    
```

The raw data (fastq/bam) and final bam has been uploaded to the [GenAP datahub](https://datahub-l3w01g9s.udes.genap.ca/profyle_benchmark/)

### Goal

Each center will upload the ceph mixture locally and run their pipeline/variant callers of interest on the provided fastqs/final bam

Each center will then provide their raw vcf file for benchmark assessment following well established benchmarking paradigm i.e. similar to the CGEN benchmarking

Real time genomics (RTG) vcfeval will be run locally by McGill using same arguments and truth set provided by the Brad Chapman from Havard University

The following data will be generated and the table below will be filled out and made available to the group: 

|  Center  | True-pos | False-pos | False-neg | Precision | Sensitivity | F1-measure |
|:-------- |:-------- |:--------- |:--------- |:--------- |:----------- |:---------- |
| BCGSC    | 964369   | 2995      | 140417    | 0.9969    | 0.8729      | 0.9308     |
| McGill   | 1076763  | 12621     | 28023     | 0.9884    | 0.9746      | 0.9815     |
| SickKids | 1077765  | 56143     | 27021     | 0.9595    | 0.9755      | 0.9629     |

Further information regarding processing see [google doc](https://docs.google.com/spreadsheets/d/1Q_YhYl_vkF8nNmNsZplAWuKsmKJK8iojaNNqQD4Q78M/edit?usp=sharing)

PHASE 2: Common Practice Benchmarking
--------------------------------------

### August 16, 2018

In this section the group identified areas in both bam processing and post annotations which could harmonized to ensure more comparable end point results.

**BAM processing refinements**

Using McGill's inhouse PCRFree data of NA12878 used in CGEN comparisons previously. Results were compared against the high quality Genome in a bottle (GIAB) truth set. Specific steps were reassessed to determine their downstream impact on benchmarking metrics (F1-score)

### Setup

As depicted in the diagram above the 45x NA127878 PCRFree dataset was processed as follows:
1. Adapter and quality filtering of raw reads with skewer 0.2.2
2. Alignment of trimmed reads to GRCh37.p13 with bwa+mem 0.7.15
3. GATK 3.8 Indel realignment to refine indel regions
4. Mark duplicates with sambamba 0.6.6
5. Base recalibration with GATK 3.8
6. SNVs and Indels identified by GATK 3.8 Haplotype caller   

Three variations of the above steps were run:  
 - The baseline run which contained all six steps (NA12878_baseline).   

 - Run without base recalibration (BQSR) which contains steps: 1-4 and 6 (NA12878_noBQSR)
 
 - Run without indel realignment (IR) and base recalibration (BQSR) which contains steps: 1-2, 4 and 6 (NA12878_noIR_noBQSR)

### Results

Generated using Realtime Genomics (RTG) vcfeval 3.6.2 and GIAB 3.3.2 truth set

|       Dataset       | True-pos | False-pos | False-neg | Precision | Sensitivity | F1-measure |
|:------------------- |:-------- |:--------- |:--------- |:--------- |:----------- |:---------- |
| NA12878_baseline    | 3684547  | 36515     | 6609      | 0.99097   | 0.9972      | 0.99408    |
| NA12878_noBQSR      | 3685160  | 42854     | 5996      | 0.9885    | 0.99838     | 0.99342    |
| NA12878_noIR_noBQSR | 3685139  | 42777     | 6017      | 0.98853   | 0.99837     | 0.99342    |


### Conclusion

Based on the comparison of the baseline versus no BQSR it would appear that BQSR improves precision at the cost of sensitivity. 

The inclusion of indel realignment and absences of BQSR improves sensitivity at the cost of precision.

Based on these outcomes and a preference for sensitivity, the second varitions; the inclusion of IR would be preferred.  

However, a few caveats:
 - Indel realignment is deprecated in GATK4
 - Indel realignement changes the mapping qualities of the original aligned file

### Sept 13, 2018

Based on the results from August 16, 2018, running our in-house NA12878 PCRFree library through the full steps took ~72 hours, without BSQR it would take roughly 48hr.

However, for resource and variant caller testing it would be benefical to select or subset datasets for faster assessment times to allow for replicate testing, while still maintaining enough snp and indel diversity to 
achieve meaningful benchmarking metrics.

Two datasets come to mind.  DREAM 3 WES dataset and subsetting the ceph mixture

|       Dataset         | Total variants |Num SNPs   | Num indels |  Size indels | 
|:--------------------: |:-------------- |:--------- |:---------- |:------------ |
|      DREAM3_WES       | 760            | 361       | 399        | -53 to 70bp  |
|     ceph_mixture      | 1104786        | 10007805  | 96981      | -29 to 66bp  |             


## DREAM3 WES setup

DREAM synthetic 3 dataset with 100% cellularity and 3 subclones (50%, 33% and 20%) 
Normal 45x, Tumor 45x

1. Adapter and quality filtering of raw reads with skewer 0.2.2
2. Alignment of trimmed reads to GRCh37.p13 with bwa+mem 0.7.15
3. GATK 3.8 Indel realignment to refine indel regions
4. Mark duplicates with sambamba 0.6.6
5. Base recalibration with GATK 3.8
6. SNVs and indels where identified using numerous somatic variant callers

Generated using Realtime Genomics (RTG) vcfeval 3.6.2 and DREAM3 truth set generated by BAMsurgeon

|   Somatic caller    | Version     | Precision   | Sensitivity | F1-measure |  TP indel size | cpu time (chunks)    |
|:------------------- |:----------- |:----------- |:----------- |:---------- |:-------------- |:-------------------- |
|     **varscan**     |  2.4.3      | 0.1114      | 0.8934      | 0.1981     |  -32 to 25     |    15 mins (23)      |
|     **vardict**     |  1.4.8      | 0.3355      | 0.9395      | 0.4945     |  -53 to 39     |    14 mins (23)      |
|     **samtools**    |  1.3        | 0.1831      | 0.8632      | 0.3022     |  -32 to 25     |    10 mins (23)      |
|     **mutect2**     |  3.8        | 0.9087      | 0.9303      | 0.9194     |  -53 to 70     | 4.5 hrs (1 - 1 cpu)  |
|       mutect2       |  4.0.8.1    | 0.8958      | 0.9276      | 0.9114     |  -53 to 39     | 105 mins (1 - 8 cpu) |
|       strelka2      |  2.9.6      | 0.9265      | 0.8461      | 0.8845     |  -32 to 25     | 16 mins (1-12cpu)    |
|    strelka2+manta   | 2.9.6+1.3.2 | 0.9279      | 0.8803      | 0.9034     |  -40 to 39     |  14+22mins (1-12,6)  |  
|        lancet       |  1.0.7      | 0.8603      | 0.8671      | 0.8637     |  -53 to 39     |    13.5h (12 cpu)    |
|       ensemble      | 1+ callers  | 0.0565      | 0.9658      | 0.1058     |  -53 to 70     |                      |
|       ensemble      | 2+ callers  | 0.9954      | 0.8632      | 0.8637     |  -40 to 39     |                      |

**Oct 25, 2018** - Generated violin plots to visualize the 3 subclones at 50%, 33% and 20% VAFs

![d3wes_subclone_vaf](img/vaf_violin.jpeg)

## Observations

1. Most sensitive : Vardict, Mutect2_3.8, Mutect2_4.0.8.1, strelka2+manta, strelka2...
2. Most precise   : Strelka2+manta, strelka2, Mutect2_3.8, Mutect2_4.0.8.1 ...
3. CPU time: Mutect2_4.0.8.1 is faster than GATK3, strelka2+manta faster than GATK4, lancet is extremely slow.
4. TP indel size: Varscan2, Samtool and strelka2 exactly the same, Mutect2 3.8 captures the widest range of indel sizes, surprisely Strelka2+manta and lancet does not.
5. Ensemble approach: calls from all 4 callers is more sensitive than just one caller.  Filtering by 2 or more callers greatly reduces FPs but at the expense of TPs
6. **Oct 25, 2018** Subclone VAFs: Most callers except lancet identify subclones at 50%, 33% and 20%.  

## Conclusions

1. Likely replacing samtools with strelka2+manta but to be confirmed with ceph mixture analysis.
2. Continue to investigate and improve GATK4 mutect2 calls 

### Oct 25, 2018

## Ceph mixture

Selection of ceph mixture chromosome for fast benchmarking framework (GRCh37.p13)

| Chromosome | GC content | Total TP | TP snps | TP indels |  TP indel dist | Precision | Sensitivity| F1-score |
|:---------- |:---------- |:---------|:--------|:--------- |:-------------- |:--------- |:---------- |:-------- |
| All        | 0.4174     | 1104786  | 1007805 | 96981     | -29 to 66      | 0.9401    | 0.9758     | 0.9576   |
| 1	         | 0.4174	  | 88571	 | 80750   | 7821      | -45 to 50      | 0.9395	| 0.9748	 | 0.9568   |
| 2	         | 0.4024	  | 95853	 | 87322   | 8531	   | -48 to 47      | 0.9442	| 0.9752	 | 0.9594   |
| 3	         | 0.3969     | 81770	 | 74633   | 7137	   | -42 to 42      | 0.9406	| 0.9776	 | 0.9587   |
| 4	         | 0.3825	  | 80166	 | 72806   | 7360	   | -40 to 56      | 0.9370	| 0.9768     | 0.9565   |
| 5	         | 0.3952	  | 75522	 | 68888   | 6634      | -37 to 66      | 0.9416	| 0.9776	 | 0.9592   |
| 6	         | 0.3961	  | 71683	 | 65091   | 6592	   | -41 to 58      | 0.9379	| 0.9754     | 0.9563   |
| 7	         | 0.4075	  | 62689	 | 57177   | 5512      | -37 to 48      | 0.9388	| 0.9769	 | 0.9574   |
| 8	         | 0.4018	  | 61280	 | 56249   | 5031	   | -30 to 44      | 0.9385	| 0.9789	 | 0.9583   |
| **9**      | 0.4132	  | 48552	 | 44433   | 4119	   | -40 to 62      | 0.9432	| 0.9757	 | 0.9592   |
| 10	     | 0.4158	  | 53528	 | 48864   | 4664	   | -45 to 52      | 0.9436	| 0.9762	 | 0.9596   |
| 11	     | 0.4157	  | 51846	 | 47354   | 4492      | -49 to 46      | 0.9341	| 0.9768	 | 0.9550   |
| 12         | 0.4081     | 53161	 | 48374   | 4787	   | -35 to 41      | 0.9337	| 0.9747	 | 0.9538   |
| 13	     | 0.3853	  | 41425	 | 37756   | 3669	   | -29 to 54      | 0.9453	| 0.9775	 | 0.9611   |
| 14	     | 0.4089	  | 36149	 | 32931   | 3218	   | -41 to 48      | 0.9374	| 0.9760	 | 0.9563   |
| 15	     | 0.4220	  | 30377	 | 27780   | 2597	   | -50 to 54      | 0.9393	| 0.9765     | 0.9576   |
| 16         | 0.4479	  | 34462	 | 31971   | 2491	   | -40 to 44      | 0.9500	| 0.9754	 | 0.9625   |
| 17	     | 0.4554	  | 27688	 | 25168   | 2520	   | -29 to 42      | 0.9411	| 0.9709	 | 0.9558   |
| 18	     | 0.3979	  | 33736	 | 30649   | 3087	   | -41 to 33      | 0.9447	| 0.9746	 | 0.9594   |
| 19	     | 0.4836	  | 20347	 | 18389   | 1958	   | -28 to 44      | 0.9334	| 0.9580	 | 0.9455   |
| 20	     | 0.4413     | 25771	 | 23584   | 2187	   | -32 to 52      | 0.9484	| 0.9751	 | 0.9616   |
| 21         | 0.4083	  | 15550	 | 14196   | 1354	   | -27 to 36      | 0.9533	| 0.9737	 | 0.9634   |
| 22         | 0.4799	  | 14660	 | 13440   | 1220      | -29 to 39      | 0.9575	| 0.9689	 | 0.9631   |

**Selection criteria**
1. Reasonable number of true positive SNPs and Indels
2. Sizable indel size distribution
3. GC content close to global average
4. Mutect2 3.8 benchmarking metrics (Added Oct)
5. Any others? e.g. # genes, # pseudogenes

Choices either chr 5 or chr9.  Favoring chr 9 due to GC content.

## Next steps

1. Extract chromosome Z from fasta and bam (both reads aligned to chrZ and unmapped reads)
2. Test a select number of callers on chrZ and confirm benchmarking metrics similar to those ran on the full genome
3. If metrics consistent download the two ceph samples used to generate the mixture and create synthetic purities from 10% to 100% at 10% intervals
4. Rerun all callers on all 10 synthetic datasets to help select callers to best capture the full spectrum of purities 

### Nov 22, 2018

## Chromosome 9 of Ceph mixutre

|   Somatic caller    | Version     | Precision   | Sensitivity | F1-measure |  TP indel size | cpu time (chunks)    |
|:------------------- |:----------- |:----------- |:----------- |:---------- |:-------------- |:-------------------- |
|     **varscan**     |  2.4.3      | 0.9661      | 0.9743      | 0.9702     |  -25 to 36     |                      |
|     **vardict**     |  1.4.8      | 0.9074      | 0.9016      | 0.9045     |  -46 to 36     |                      |
|     **samtools**    |  1.3        | 0.9421      | 0.6591      | 0.7756     |  -30 to 36     |                      |
|   **mutect2_all**   |  3.8        | 0.9432      | 0.9757      | 0.9592     |  -46 to 40     |                      |
|       mutect2       |  3.8        | 0.9958      | 0.6269      | 0.7694     |  -46 to 40     |                      |
|     mutect2_all     |  4.0.8.1    | 0.9478      | 0.9782      | 0.9627     |  -46 to 40     |                      |
|       mutect2       |  4.0.8.1    | 0.9948      | 0.8554      | 0.9198     |  -46 to 40     |                      |
|      strelka2       |  2.9.6      | 0.9981      | 0.9219      | 0.9585     |  -34 to 36     |                      |
| **strelka2+manta**  | 2.9.6+1.3.2 | 0.9981      | 0.9221      | 0.9586     |  -46 to 40     |                      |  
|       ensemble      | 1+ callers  | 0.8811      | 0.9909      | 0.9328     |  -46 to 40     |                      |
|       ensemble      | 2+ callers  | 0.9649      | 0.9831      | 0.9739     |  -46 to 40     |                      |

## Insilico purity sample SOP

1. To produce insilico samples for purities ranging from 10% to 90%, reads were extracted from GIAB samples: Normal: 60x HG002 and Tumor: 300x HG001
2. Extract chromosome 9 and subset number of reads based on purity using sambamba with a specific random seed for reproducibility 
3. Sort and merge HG002 and HG001
4. Trim, align, indel realignment, mark duplicates and qc metrcis on each 9 pairs
5. Run somatic caller on final mark dup bam
6. Use ceph mixture truth set subset for chr 9 and run vcfeval 3.6.2

|    Sample name      | mean cov.   | % over 10x  | % over 25x  | % over 50x | est. purity |
|:------------------- |:----------- |:----------- |:----------- |:---------- |:------------|
| HG002_30x           | 29.67	    | 93.95	      | 76.27	    | 1.5	     | NA          |
| HG001_HG002_p10     | 58.83	    | 95.18	      | 93.52	    | 78.54      | 8.88%       |
| HG001_HG002_p20	  | 58.56	    | 95.32	      | 93.59	    | 78.10	     | 20.03%      |
| HG001_HG002_p30	  | 58.28	    | 95.45	      | 93.65     	| 77.60	     | 30.17%      |
| HG001_HG002_p40	  | 58.00     	| 95.54	      | 93.72	    | 77.11	     | 39.25%      |
| HG001_HG002_p50	  | 57.69	    | 95.61	      | 93.79    	| 76.51	     | 49.84%      |
| HG001_HG002_p60	  | 57.37	    | 95.66	      | 93.85	    | 75.87	     | 58.29%      |
| HG001_HG002_p70	  | 57.05	    | 95.69	      | 93.90	    | 75.20	     | 68.79%      |
| HG001_HG002_p80	  | 56.69	    | 95.70	      | 93.94	    | 74.41	     | 79.25%      |
| HG001_HG002_p90	  | 56.35	    | 95.71	      | 93.97	    | 73.59	     | 89.32%      |


## Results

**Precision**

![HG001_HG002.precision](img/HG001_HG002.precision.jpeg)

**Sensitivity**

![HG001_HG002.sensitivity](img/HG001_HG002_sens.jpeg)

**F1-Score**

![HG001_HG002.f1score](img/HG001_HG002_F1score.jpeg)

## Observations

1. Precision >=95%: Ensemble_2+, Strelka2 and mutect2 stable across all purity ranges.  Vardict is good at low purity 10-80%.  Samtools and Varscan2 good between 20% to 60%.  Ensemble_1+ below 95%

2. Sensitivity >=95%: >=20% Ensemble_1+, >=30% purity Ensemble +2+, Mutect2, and Vardict, >=40% purity strelka2,  40-70% varscan2,  50-80% samtools

3. F1-score >=95%: >=20% purity mutect2, Ensemble_2+, >=30% purity strelka2, 30-60% Ensemble_1+, varscan2, 40-80% samtools

Any new caller to test? Or move onto RNA and structural variants.

### February 28, 2018

## Part 1: Assessment of combination of callers of chromosome 9 of Ceph mixture

In this section we investigate the behaviour of combining callers and majority-rule filtering at various purity levels of insilico ceph mixture.  Based on the assessement of individual callers from both the synthetic and insilico mixture: mutect2_3.8, strelka_2.9.6, vardict_1.4.8 and varscan2_2.4.3 were chosen. The expectations is that the addition of more callers should add value in terms of benchmarking metrics.

## Processing SOP

1. VCFs from previous experiment (Nov 22, 2018) for four callers:  mutect2, strelka, vardict and varscan were selected for combinatorial analysis
2. VCFs were combined using a bcftools merge wrapper
3. Ensemble vcf was assessed against the ceph_mixture chr 9 truth set using vcfeval 3.6.2

## Results

**F1-score single callers from Nov 22, 2018**

![HG001_HG002.f1score](img/HG001_HG002_F1score.jpeg)

**F1-score union combinations of callers**

![f1_ensemble_caller1](img/f1_score_ens_comb_caller1.jpeg)

## Observation : Union of calls 

1. Looking at the union of various combination of callers (*_1+) the addition of callers reduces F1 score.  Looking at the specificity and sensitivity more closely, the addition of more callers does increase sensitivity the rate of increase in sensitivity is smaller than the decrease in specificity.  Likely due to caller specific FP singletons.

**F1-score combinations of callers filtering for varinats found in >=2 callers**

![f1_ensemble_caller2](img/f1_score_ens_comb_caller2.jpeg)

## Observation : 2 or more callers

1. Looking at calls identified in 2 or more callers, the addition of callers has minimal impact when purity is above 60%. In fact 3 callers has slightly better F1 scores >=50% purity. However, at lower purity, especially between 10-49%, the addition of callers seems to improve F1 score based on the dramatic jump between 2 and 3 callers, and a minor increase between 3 and 4 callers. Looking at the specificiry and sensivity, the decrease precision due to filtering out TP singletons decreases faster than the increase in sensitivity as callers are added.

## Recommendations

1. The use of ensemble approach and the number of callers involved in the analysis is dependent on the estimated purity.
2. Purity above 50%, I would recommend using at least 3 callers and filter >=2 callers.
3. Purity below 50%, I would recommend using at least 4 callers and filter >=2 callers.

## What's next

1. Missed testing 3 callers implementation using mutect2, strelka2 and varscan2
2. Trying the machine learning approach: somaticSeq 
3. Any other suggestions?

### April 25, 2018

## Part 2a: Assessment of combination of callers of chromosome 9 of Ceph mixture

Again in this section we investigate the behaviour of combining callers and majority-rule filtering at various purity levels of insilico ceph mixture.  In part 1 we looked at the combinations of two, three and four callers where one combination was missed - 3 callers: mutect2, strelka2 and varscan.  Also we wanted to compare these combination of callers to somaticSeq - the machine learning approach which won the DREAM challenge.

## Processing SOP

1. VCFs from previous experiment for four callers:  mutect2, strelka, vardict and varscan were selected for combinatorial analysis
2. VCFs were combined using a bcftools merge wrapper
3. Ensemble vcf was assessed against the ceph_mixture chr 9 truth set using vcfeval 3.6.2

**F1-score for three caller ensemble approaches**

![F1_score_ensemble_3caller](img/F1_score_ensemble_3caller.jpeg)

## Observations : 3 callers

1. The addition of vardict or varscan2 to mutect2 and strelka2 calls has opposing behaviours when looking at different purities.
2. Vardict improves calls at the higher purity range and drops below the 95% mark at 20% purity
3. Varscan2 improves calls at the lower purity range and drops below the 95% mark at 80% purity

## Part 2b: Assessment of SomaticSeq against ensemble approach for chromosome 9 of Ceph mixture

## Processing SOP

1. VCFs from previous experiment for four callers:  mutect2, strelka, vardict and varscan were selected for combinatorial analysis and SomaticSeq
2. Ensemble and SomaticSeq consensus/boosts vcfs were assessed against the ceph_mixture chr 9 truth set using vcfeval 3.6.2

**F1-score for SomaticSeq comparison**

![F1_score_somaticseq](img/F1_score_somaticseq.jpeg)

## Observations : SomaticSeq

1. SomaticSeq F1 scores are slightly better than the ensemble approach at purities below 30%.
2. Conversely the ensemble approach has more consistent F1 scores at the higher purity levels (>30% purity the curve is relative flat)
3. Within somaticSeq, the boost implementation and the consensus 2+ callers have the highest F1 scores at lower putiry levels, however the machine learning approach quickly decreases at purity above 60%.  

# Part 3: Assessment of DRAGENS somatic option for all chromosomes of the ceph_mixture

|   Somatic caller    | Version          | Precision   | Sensitivity | F1-measure | 
|:------------------- |:-----------------|:----------- |:----------- |:---------- |
|       varscan       |  2.4.3           | 0.9296      | 0.9604      | 0.9448     |
|       vardict       |  1.4.8           | 0.9019      | 0.9036      | 0.9027     |
|       mutect2       |  3.8             | 0.9401      | 0.9758      | 0.9576     |
|       strelka2      |  2.9.6           | 0.9976      | 0.9232      | 0.9589     |
|       ensemble      | 1+ callers       | 0.8387      | 0.9937      | 0.9096     |
|       ensemble      | 2+ callers       | 0.9842      | 0.9811      | 0.9826     |
|       dragen        | 07.011.295.3.2.8 | 0.6389      | 0.9816      | 0.7740     |
|     dragen_pass     | 07.011.295.3.2.8 | 0.9524      | 0.6682      | 0.7854     |

## Observations : DRAGEN

1. DRAGEN all variants is more sensitive than all single callers, however, the call set contains more false positives than the other callers.
2. Including only variants that pass DRAGEN's filtering criteria we have removes numerous FPs but at the detriment to TPs.

## Recommendations

1. The previous recommandation still stand i.e. above 50% putiry 3 callers is sufficient to break ties and the use of vardict would be preferred. At lower purity, below 50%, the addition of varscan2 has indeed shown to improve the low purity calls.
2. The use of SomaticSeq and machine leaning is not a necessary step.
3. DRAGEN somatic is not to be included yet but the generation of the bam could be.

## What's next

1. Test DRAGEN generated BAMs with the ensemble approach to see if we can benefit from the processing sped up without lose of accuracy.

### Contact info
For any questions or concerns regarding information contained within please contact: 

Robert Eveleigh: robert.eveleigh@mcgill.ca

Mathieu Bourgey: mathieu.bourgey@mcgill.ca

Guillaume Bourque: guil.bourque@mcgill.ca

