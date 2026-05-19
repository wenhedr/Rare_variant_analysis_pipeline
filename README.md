Rare Variant Analysis Pipeline:  Step-By-Step
Version 03.23.2023
<img width="468" height="73" alt="image" src="https://github.com/user-attachments/assets/32861959-fa57-4e59-9e51-0ca28f395d98" />


Aim: to identify genes associated with case disorder using exome-wide gene-based burden testing, in which the aggregate frequency of rare protein-altering variants in each gene in the exome is tested against a set of suitable controls. 

Case sample ascertainment and sequencing 
-	Whole-exome sequencing was performed from DNA extracted from peripheral blood or saliva.

Step 1: Pre-processing Steps
1.	Joint Calling 
Aim: identify and call variants using information from multiple samples simultaneously. 
Rationale: This step empowers variant discovery by providing the ability to leverage population-wide information from a cohort of multiple samples, allowing us to detect variants with great sensitivity and genotype samples as accurately as possible. 
a.	Requirements: 
•	GATK version 3.8
b.	Quality criteria: 
•	--minBaseQuality 10 or greater 
•	--minMappingQuality 20 or greater
c.	Input: 
•	BAM files
d.	Output: 
•	Variant call files (VCF) files

2.	VEP Annotation
Aim: provide functional and biological information about variants.
Rationale: This step annotates genomic variants with prediction on molecular consequences of genetic variant. For variant annotation, we have achieved our best results using Ensembl VEP version 100 GRCh 38 (https://www.ensembl.org/info/docs/tools/vep/script/index.html). 

a.	We highly recommend annotating the case and control data in the same way.
b.	Requirements: 
•	GATK version 3.8
•	Ensemble Vep GRCh38 (https://www.ensembl.org/info/docs/tools/vep/script/index.html)
•	BCFTOOLS (https://samtools.github.io/bcftools/)
c.	Input: 
•	Pre-annotation VCF files
d.	Output: 
•	Annotated VCF files
e.	Command line run:
•	Separating multi-allelic coordinates and left-aligning indels: 
“bcftools norm -m -any -f Homo_sapiens_assembly38.fasta in.vcf.gz | bgzip > out.vcf.gz”
•	Run VEP Annotation:
Appendix 1: VEP annotation JavaScript

3.	Exclude Chr X, Y, MT
Aim: Exclude chromosome X, Y, and MT from both case and control. 
Rationale: More efficiency and convenience in data-handling by reducing the data size to only include chromosomes of interest. Separate analyses are needed for the X, Y, and mitochondrial chromosomes because if their unique patterns of inheritance.
a.	Requirements: 
•	Running environment: Python v3.7
•	split_x_chr_multiproc.py (Appendix 2: TRAPD Script)
b.	Input:
•	Case_annotated.vcf
•	Control_annotated.vcf
c.	Output:
•	Vcfs without Chr X, Y, MT
o	Case_annotated_exl_chrxymt.vcf
o	Control_annotated_exl_chrxymt.vcf
•	Chr X, Y, MT vcfs
o	Case_annotated_chr_xymt.vcf
o	Control_annotated_chr_xymt.vcf
d.	Command lines to run:
•	Exclude Chr X, Y, MT in case.
“python C3_split_x_chr_multiproc.py -v Case_annotated.vcf -o Case_annotated_exl_chrxymt.vcf --xchroutfile Case_annotated_chr_xymt.vcf”
•	Exclude Chr X, Y, MT in control:
python C3_split_x_chr_multiproc.py -v Control_annotated.vcf -o Control_annotated_exl_chrxymt.vcf --xchroutfile Control_annotated_chr_xymt.vcf

4.	Assessing Coverage 
Aim: Obtain a quality depth value for each sample at each chromosomal position.
Rationale: This step generates coverage information for quality control, allowing us to assess the coverage of each batch during the quality control step (referred to as "QC1 Coverage" below).
•	If you use individual-level data for cases and controls, you will need to generate the files for each batch for cases and controls. 
•	If you use gnomAD as controls, the depth of coverage for each position for gnomAD samples can be downloaded from the gnomAD v2 website: https://gnomad.broadinstitute.org/downloads#v2-coverage                   Coverage was then lifted over to hg38 using the UCSF LiftOver tool on the web (https://genome.ucsc.edu/cgi-bin/hgLiftOver). The coverage file in hg19 can be uploaded to the website, and after the lift-over process is complete, the lifted over coverage file can be downloaded.
b.	Requirements: 
•	GATK v 3.5:  "Depth of coverage" function (https://gatk.broadinstitute.org/hc/en-us/articles/360041851491-DepthOfCoverage-BETA-)

c.	Criteria for quality depth to be calculated:
•	--minBaseQuality 10 or greater 
•	--minMappingQuality 20 or greater

d.	Input:
•	BAM/CRAM files 
•	BAM/CRAM index files (.bai/.crai)
•	Reference fasta
•	Reference fasta dict file 
•	Reference fasta index file 
•	The interval list (The interval list required should contain positions that were sequenced by the platform used. Our samples were sequenced by the Broad using Illumina whole exome kit. If your samples were sequenced using different kit, please use the interval list for the specific sequencing method. We can also help find the right interval list if you have trouble find one.)
e.	Output: An .csv file with a quality depth value for each sample at a given chromosomal position. (format/example of header)
•	Case_coverage_batch1.csv
•	Case_coverage_batch2.csv
•	…
•	Control_coverage_batch1.csv
•	Control_coverage_batch2.csv
…
f.	Command lines to run for generating coverage file for each batch:
“ java -jar "GenomeAnalysisTK.jar" DepthOfCoverage -I bam1 -I bam2 -I bam3 --reference reference.fasta -L interval_list.txt --output coverage_batch.csv --minBaseQuality 10 --minMappingQuality 20 --omitIntervalStatistics --omitLocusTable --includeRefNSites --countType COUNT_FRAGMENTS” 

Step2:  Calibrate filters using synonymous variants. 
QC1: Coverage
Aim: Retain only positions with a quality read depth of 10 in 90% of samples.
Rationale: Ensure that the data being analyzed meet criteria for high-quality coverage in both cases and controls.
a.	Requirements: 
•	Running environment: Python v3.7 (https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-python.html) (running environment)
•	Bedtools “intersect” and “merge” functions (https://bedtools.readthedocs.io/en/latest/) 
•	QC1_get_quality_positions.py 
•	coverage.csv (one for each batch)
b.	Input:
•	Coverage.csv (one for each batch in cases and controls (Unless using gnomAD))
•	Case_annotated_exl_chrxymt.vcf
•	Control_annotated_exl_chrxymt.vcf
c.	Output:
•	“batch1_batch2….control.quality_postition_interval.bed”
Example of position_interval.bed: Column 1 is the chromosome. Column 2 is the start of the interval. Column 3 is the end of the interval.
chr1	925939	926015
chr1	930244	930330
chr1	931036	931091
chr1	935769	935898
chr1	939057	939104
•	Case_QC1.vcf
•	Control_QC1.vcf

d.	Command lines to run:
•	Get quality position “.bed” file for each batch
“python QC1_get_quality_positions.py -i case_batch1_coverage_file.csv" -o case_batch1.updated.bed 
•	Intersect good positions for across all batches in cases and controls
“bedtools intersect -a “batch1_coverage.bed” -b "batch2_coverage.bed”  > batch1_batch2.intersect.bed”
..
“bedtools intersect -a “batch1_batch2_....coverage.bed” -b "control_coverage.bed”  > batch1_batch2….control.intersect.bed”
•	Sort position in intersected .bed files
“sort-bed “batch1_batch2….control.intersect.bed” > “batch1_batch2….control.quality_postition_interval.bed””
•	Merge into high-quality interval list
“bedtools merge -i batch1_batch2….control.intersect.sorted.bed > batch1_batch2….control.intersect.sorted.merged.bed
•	Retain only high-quality positions in case and control .vcf files
“bedtools intersect -a Case_annotated_exl_chrxymt.vcf -b "batch1_batch2….control.quality_postition_interval.bed" -header > Case_QC1.vcf”
“bedtools intersect -a Control_annotated_exl_chrxymt.vcf -b "batch1_batch2….control.quality_postition_interval.bed" -header > Control_QC1.vcf”

Filter1: Filter by Annotation (synonymous protein-coding variants)
Aim: Retain only variants with a canonical synonymous protein-coding transcript 
Rationale: This step only includes variants with canonical synonymous protein-coding transcripts. We calibrate filters using synonymous variants because they are likely to be benign and present at equal frequencies in cases and controls and thus can be used to test the null distribution of P-values in burden test results.
a.	Requirements: 
•	Running environment: Python v3.7
•	Filter1_synonymous_variant_Create_exclusion_SNPlist_by_annotation_1_29_2023.py (Appendix 2: TRAPD Script)
•	Exclude_snps_and_make_new_vcf_6-22-2022.py (Appendix 2: TRAPD Script)
b.	Input: 
•	Case _QC1.vcf
•	Control_QC1.vcf
c.	Output: 
•	Case_synonymous_Filter1.vcf
•	Control_ synonymous_Filter1.vcf
•	Case_synonymous.Filter1_exclusion_SNPlist.txt
•	Control_synonymous.Filter1_exclusion_SNPlist.txt
d.	Command lines to run:
•	Retain only variants with a canonical synonymous protein-coding transcript.
“python Filter1_synonymous_variant_Create_exclusion_SNPlist_by_annotation_1_29_2023.py -v Case_QC1.vcf --excludeoutfile Case_synonymous.Filter1_exclusion_SNPlist.txt “
“python Filter1_synonymous_variant_Create_exclusion_SNPlist_by_annotation_1_29_2023.py -v Control_QC1.vcf --excludeoutfile Control_ synonymous. Filter1_exclusion_SNPlist.txt”
•	Combine the exclusion list:
“cat Case_synonymous.Filter1_exclusion_SNPlist.txt Control_synonymous.Filter1_exclusion_SNPlist.txt > Filter1_synonymous_exclusion_SNPlist.txt
•	Exclude from both case and control:
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_QC1.vcf -o Case_synonymous_Filter1.vcf -m Case_synonymous_Filter1.excluded_more_info.txt -s QC2_synonymous_exclusion_SNPlist.txt
python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Control_QC1.vcf -o Control_synonymous_Filter1.vcf -m Control_synonymous_Filter1.excluded_more_info.txt -s QC2_synonymous_exclusion_SNPlist.txt”

QC2: “PASS” 
Aim: Retain only variants that “PASS” GATK quality metrics.
Rationale: "PASS" is a GATK filter that is used to retain only the high-quality variants that meet certain quality metrics for analysis.
a.	Requirements and running environment: 
•	QC2_Get_ONLY_PASS_snps_10-13-2022.py (Appendix 2: TRAPD Script)
•	Python v3.7 (running environment)
b.	Input: 
•	Case_synonymous_Filter1.vcf
•	Control_ synonymous_Filter1.vcf
c.	Output: 
•	Case_synonymous_QC2.vcf
•	Control_ synonymous_QC2.vcf
•	QC2.synonymous.exclusion_snplist.txt:
d.	Command lines to run:
•	Keep only variants with “PASS” in filter fiel.
“python QC2_Get_ONLY_PASS_failed_snps_10-13-2022.py -v "Case_synonymous_Filter1.vcf -o Case_synonymous.PASS_excluded_snplist.txt -m Case_synonymous.QC2_more_info.txt”
“python QC3_Get_ONLY_PASS_failed_snps_10-13-2022.py -v Control_synonymous_Filter1.vcf -o Control_synonymous.PASS_excluded_snplist.txt -m Control_synonymous.QC2_more_info.txt
•	Combine the exclusion list.
“cat Case_synonymous.PASS_excluded_snplist.txt Control_synonymous.PASS_excluded_snplist.txt > QC2.synonymous.exclusion_snplist.txt”
•	Exclude from both case and control.
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_synonymous_Filter1.vcf -o Case_synonymous_QC2.vcf -m Case_synonymous.QC2.excluded_more_info.txt -s QC2.synonymous.exclusion_snplist.txt”
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Control_synonymous_Filter1.vcf -o Control_synonymous.QC2.vcf -m Control_synonymous.QC2.excluded_more_info.txt -s QC2.synonymous.exclusion_snplist.txt”
QC3 “AC0” 
Aim: Exclude variants flagged with “AC0”. “AC0” indicates no sample had a high-quality genotype (depth >= 10, genotype quality >= 20 and minor allele balance > 0.2 for heterozygous genotypes). 
Rationale: The "AC0" flag is used by gnomAD to exclude sites with no high-quality genotypes (AC=0), and we have adopted these criteria for quality control of our case data as well.
e.	Requirements and running environment: 
•	QC3_get_case_AC0_6-22-2022.py (Appendix 2: TRAPD Script)
•	Python v3.7 (running environment)
f.	Input: 
•	Case_synonymous_QC2.vcf
•	Control_ synonymous_QC2.vcf
g.	Output: 
•	Case_synonymous_QC3.vcf
•	Control_ synonymous_QC3.vcf
•	Case_synonymous.AC0_excluded_snplist.txt 
h.	Command lines to run:
•	Get AC0 for Case 
“python QC3_get_case_AC0_6-22-2022.py -v "Case_synonymous.QC2.vcf" -o Case_synonymous.AC0_excluded_snplist.txt”
•	Exclude from both case and control.
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_synonymous_QC2.vcf -o Case_synonymous_QC3.vcf -m Case_synonymous.QC3.excluded_more_info.txt -s Case_synonymous.AC0_excluded_snplist.txt”
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Control_synonymous_QC2.vcf -o Control_synonymous_QC3.vcf -m Control_synonymous.QC3.excluded_more_info.txt -s Case_synonymous.AC0_excluded_snplist.txt”
QC4: Filter by quality by Depth 
Aim: Retain variant with Quality by Depth (QD) >2.
Rationale: QD is the ratio of the variant quality score (QUAL) to the total number of reads (depth of coverage) supporting the variant call, and it is used to estimate the confidence of a variant call. By using a hard cut-off of QD > 2, we can filter out variants with low QC scores, which may be due to sequencing or alignment error. This helps to ensure that only high-quality variants are included in downstream analyses.
a.	Requirements: 
•	Python v3.7 (running environment)
•	QC4_get_QD_Exclusion_SNPlist_10-27-2022.py (Appendix 2: TRAPD Script)
b.	Input:
•	Case_synonymous_QC3.vcf
•	Control_ synonymous_QC3.vcf
c.	Output:
•	Case_synonymous_QC4.vcf
•	Control_ synonymous_QC4.vcf

d.	Command lines to run:
•	Generate exclusion list.
“python QC4_Get_QD_Exclusion_SNPlist_10-27-2022.py -v Case_synonymous_QC3.vcf -i Case_synonymous_QC4.snps_to_include.txt -e Case_synonymous_QC4.snps_to_exclude.txt -m Case_synonymous_QC4.excludedmoreinfo.txt --filterval 2”
“python QC4_Get_QD_Exclusion_SNPlist_10-27-2022.py -v Control_synonymous_QC3.vcf -i Control_synonymous_QC4.snps_to_include.txt -e Control_synonymous_QC4.snps_to_exclude.txt -m Control_synonymous_QC4.excludedmoreinfo.txt --filterval 2”
•	Combine the above exclusion list.
“cat Case_synonymous_QC4.snps_to_exclude.txt Control_synonymous_QC4.snps_to_exclude.txt > QC4_synonymous_exlcusion_list.txt”
•	Exclude from both case and control.
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_synonymous_QC3.vcf -o Case_synonymous_QC4.vcf -m Case_synonymous.QC4.excluded_more_info.txt -s QC4_synonymous_exclusion_list.txt”
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Control_synonymous_QC3.vcf -o Control_synonymous_QC4.vcf -m Control_synonymous.QC4.excluded_more_info.txt -s QC4_synonymous_exclusion_list.txt”

QC5: Allele Balance
Aim: This step only retains heterozygous variants for which 98% of samples in the cohort meet the criteria of normal allele balance (0.2<AB<0.8).
Rationale: Allele balance (AB) is the ratio of altered allele counts to the total number of alleles. For heterozygous variants, AB is expected to be 0.5 but can vary between 0.2 to 0.8.
a.	Requirements: 
•	QC5_get_Allele_Balance_Exclusion_List_6-22-2022.py (Appendix 2: TRAPD Script)
•	Python v3.7 (running environment)
b.	Input:
•	Case_synonymous_QC4.vcf
•	Control_ synonymous_QC4.vcf
c.	Output:
•	Case_synonymous_QC5.vcf
•	Control_synonymous_QC5.vcf
d.	Command lines to run:
•	Get the exclusion list for allele balance from case:
“python QC5_get_Allele_Balance_Exclusion_List_6-22-2022.py -v Case_synonymous_QC4.vcf -o Case_synonymous_QC5_allele_balance.exclusion_snplist.txt -m Case_synonymous_QC5_allele_balance.exclusion.more_info.txt”
•	Exclude from both case and control:
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_synonymous_QC4.vcf -o Case_synonymous_QC5.vcf -m Case_synonymous_QC5.excluded_more_info.txt -s Case_synonymous_QC5.allele_balance.exclusion_snplist.txt”
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Control_synonymous_QC4.vcf -o Control_synonymous_QC5.vcf -m Control_synonymous_QC5.excluded_more_info.txt -s Case_synonymous_QC5_allele_balance.exclusion_snplist.txt”

Filter2: Filter by MAF in gnomAD
Aim: Exclude variants with a MAF > % in gnomAD v2 and v3. Alternative threshold of MAF > 0.1% could also be considered.
Rationale: Delayed puberty affects approximately 2% of the general population. For this analysis, we will only retain rare variants with a MAF < 1% (or alternatively, less than 0.1%). To complement the coverage limitations of gnomAD v2 data, we also utilize gnomAD v3 MAF data for this filter. This step applies more to protein-altering variants than synonymous ones, but we need to treat synonymous variants in the same way for harmonization. 
a.	Requirements: 
•	Python v3.7 (running environment)
•	Filter2_get_popmax_by_VEP_excluded.py (Appendix 2: TRAPD Script)
•	Filter2_get_popmax_excluded_snps_6-22-2022.py (Appendix 2: TRAPD Script)
•	Gnomad_v3_MAF01_exclusion_list.txt (Appendix 2: TRAPD Script)
b.	Input:
•	Case_synonymous_QC5.vcf
•	Control_ synonymous_QC5.vcf
c.	Output:
•	Filter2_synonymous_MAF_01_exclusion_list.txt 
d.	Command lines to run:
•	Generate an exclusion list of variants with MAF <1% in VEP annotation for both case and control. 
“python Filter2_get_Popmax_by_VEP_annotation_WH_10_18_2022.py -v Case_synonymous_QC5.vcf -o Case_ synonymous.VEP_MAF01_excluded_snps.txt -m Case_synonymous_MAF01_excluded_snps_moreinfo.txt -f 0.01”
“python Filter2_get_Popmax_by_VEP_annotation_WH_10_18_2022.py -v Control_synonymous_QC5.vcf -o Control_synonymous.VEP_MAF01_excluded_snps.txt -m Control_ synonymous_MAF01_excluded_snps_moreinfo.txt -f 0.01”
•	Generate exclusion list of variants with popmax <1% in gnomAD v2. 
“python Filter2_get_popmax_excluded_snps_6-22-2022.py -v Control_ synonymous_QC5.vcf -o Control_synonymous_POP_MAF01_excluded_snps.txt -m Control_ synonymous_excluded_snps_moreinfo.txt -f 0.01”
•	Generate exclusion list for MAF < 1%
“cat Case_ synonymous.VEP_MAF01_excluded_snps.txt Control_ synonymous.VEP_MAF01_excluded_snps.txt Control_synonymous_POP_MAF01_excluded_snps.txt Gnomad_v3_MAF01_exclusion_list.txt > Filter2_synonymous_MAF_01_exclusion_list.txt”

QC6: Allele Frequency (For Case)
Aim: Exclude variants with allele frequency >5% in our cases.
Rationale: Variants with an allele frequency greater than 5% are more likely to be artifacts rather than true associations. Alternatively, if we target rare variants with a MAF less than 0.1% in the previous MAF filter step, variants with an allele frequency greater than 1.5% are more likely to be artifacts rather than true associations. 
e.	Requirements: 
•	Python v3.7 (running environment)
•	QC6_get_case_af_excluded.py (Appendix 2: TRAPD Script)
•	Filter2_synonymous_MAF_01_exclusion_list.txt 
f.	Input:
•	Case_synonymous_QC5.vcf
•	Control_ synonymous_QC5.vcf
g.	Output:
•	Case_synonymous_QC6.vcf
•	Control_ synonymous_QC6.vcf
•	Cae_synonymous.AF05_excluded_snps.txt
•	QC6_synonymous_exclusion_list.txt

h.	Command lines to run:
•	Generate an exclusion list of variants with AF >5% in case.
“python QC6_get_case_af_excluded.py -v Case_synonymous_QC5.vcf -o Cae_synonymous_AF05_excluded_snps.txt -m case_synonymous_AF05_excluded_snps_moreinfo.txt -f 0.05”
•	Combine the above exclusion list.
“cat Case_synonymous_AF05_excluded_snps.txt Filter2_synonymous_MAF_01_exclusion_list.txt > QC6_synonymous_exclusion_list.txt”
•	Exclude from both case and control.
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_synonymous_QC5.vcf -o Case_synonymous_QC6.vcf -m Case_synonymous.QC6.excluded_more_info.txt -s QC6_synonymous_exclusion_list.txt”
“python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Control_synonymous_QC5.vcf -o Control_synonymous_QC6.vcf -m Control_synonymous.QC6.excluded_more_info.txt -s QC6_synonymous_exclusion_list.txt”

Generate SNP list & Counting carriers in case and control.
Aim: Counting carriers in case and control cohort.  
Rationale: This step estimates cumulative counts of high-quality variant calls for all variants within each gene in the exome based on individual-level data or summary statistic counts. For individual-level data, we only count each individual once per gene.
a.	Requirements: 
•	Python v3.7 (running environment)
•	If you use gnomAD (summary counts) as control:
o	counting_gnomad_w_optional_exclusion_list_3-1-2023.py (Appendix 2: TRAPD Script)
•	If you have individual-level data for case and controls, use this script on both:
o	For step2: 
counting_controls_w_optional_exclusion_list_11-2-2022 (Appendix 2: TRAPD Script)
o	For step3: counting_individual_round0.9_w_optional_exclusion_list_ONLY_ONE_CASE_PER_GENE_2-22-2023.py (Appendix 2: TRAPD Script)
•	make_snpfile_no_ref_alt_len10_HC_HIGH_WH_2-19-2023.py (Appendix 2: TRAPD Script)

b.	Input:
•	Case_synonymous_QC6.vcf
•	Control_ synonymous_QC6.vcf
c.	Output:
•	SNP list files:
o	Case_synonymous_QC6_ snplist.txt
o	Control_synonymous_QC6_snplist.txt

The SNP list file will contain 2 columns:
1.	#GENE: gene name
2.	SNPS: A list of variants counted for the gene (separated by “,”) 
Example:
#GENE		SNPS
SAMD11	1:925973:C:T,1:930248:G:A
NOC2L		1:945122:C:T,1:946207:C:T,1:946222:C:T
KLHL17		1:962424:A:G,1:962442:G:A,1:963994:G:An

•	Gene count files 
o	If you use gnomAD summary counts as control, you will follow the script for getting gnomAD gene counts (specified in “Command lines to run” below). The output file is:
•	control_ synonymous_QC7.gene_count.txt
The gnomAD gene count file will contain Six columns:
1. #GENE: gene name
2. HET_IND_FREQ: Frequency of individuals carrying at least one qualifying variant in the gene
3. HOM_IND_FREQ: Frequency of individuals carrying at least one qualifying variant in the gene.
4. SMALLEST_IND_NUMBER: Because not all qualifying variants in a given gene may be well genotyped in all individuals in the cohort, this parameter determines for each qualifying variant the number of individuals in the cohort who are well genotyped, then takes the smallest of these numbers. This number is then used in the calculations to estimate counts of individuals with qualifying variants. 
5. HET_IND_EST_COUNT: Estimated number of individuals carrying at least one heterozygous qualifying variant in the gene.
6. HOM_IND_EST_COUNT: Estimated number of individuals carrying at least one homozygous qualifying variant in the gene.
Example of gene count file:
#GENE	HET_IND_FREQ	HOM_IND_FREQ	SMALLEST_IND_NUMBER	HET_IND_EST_COUNT	HOM_IND_EST_COUNT
SAMD11	0.0007999491385682687	0.0	19517.0	16.0	0.0
NOC2L	0.0	0.0	21369.0	0.0	0.0


o	If you use individual-level data for case and control, you will run the script on case and control data both. You will get output gene count files for the case and control separately.
•	Individual_level_data_synonymous_QC7.gene_count.txt
The individual-level data gene count file will contain SIX columns:
1. #GENE: gene name
2. HET_IND_FREQ: Frequency of individuals carrying at least one heterozygous qualifying variant in the gene
3. HOM_IND_FREQ: Frequency of individuals carrying at least one homozygous qualifying variant in the gene.
4. SMALLEST_IND_NUMBER: The smallest number of individuals that pass sequencing quality filters at the position of the qualifying variants in the gene.
5. HET_IND_EST_COUNT: Estimate # of individuals carrying at least one heterozygous qualifying variant in the gene 
6. HOM_IND_EST_COUNT: Estimate # of individuals carrying at least one homozygous qualifying variant in the gene.
Example of gene count file:
#GENE	HET_IND_FREQ	HOM_IND_FREQ	SMALLEST_IND_NUMBER	HET_IND_EST_COUNT	HOM_IND_EST_COUNT
CIAO1	0.01351351	0	148	2	0
C1orf159	0.01360607	0	146	1	0

d.	Command lines to run:
•	Make SNP files for both case and control.
“python make_snpfile_no_ref_alt_len10_HC_HIGH_WH_2-19-2023.py -v Case_synonymous_QC6.vcf -o Case_synonymous_QC6_snipfile.txt “
“python make_snpfile_no_ref_alt_len10_HC_HIGH_WH_2-19-2023.py -v Case_synonymous_QC6.vcf -o Case_synonymous_QC6_snipfile.txt”
•	If you use gnomAD as control, please run the following script on gnomAD data: 
“python counting_gnomAD_w_optional_exclusion_list_6-24-2022.py -v Control_ synonymous_QC6.vcf -s Control_ synonymous_QC6_snipfile.txt --pop NFE --snpoutfile Control_ synonymous_QC6.snp_counts.txt -o Control_ synonymous_QC6.gene_counts.txt”

•	If you are using individual-level data for case and control, please run the following script for both case and control: 
“python counting_individual_round0.9_w_optional_exclusion_list_ONLY_ONE_CASE_PER_GENE_2-22-2023.py -v Case_ synonymous_QC6.vcf -s Case_ synonymous_QC6_snipfile.txt --snpoutfile Case_ synonymous_QC6.snp_counts.txt -o Case_ synonymous_QC6.gene_counts.txt”
“python counting_individual_round0.9_w_optional_exclusion_list_ONLY_ONE_CASE_PER_GENE_2-22-2023.py -v Control_ synonymous_QC6.vcf -s Control_ synonymous_QC6_snipfile.txt --snpoutfile Control_ synonymous_QC6.snp_counts.txt -o Control_ synonymous_QC6.gene_counts.txt”

Burden test for synonymous variants 
Aim: perform a burden test is to determine if there is an apparent excess burden of rare synonymous variants in a specific gene or genomic region in cases compared to controls. 
Rationale: The null distribution of  p-values in synonymous variant burden test would indicate well harmonization of case and control data. If not, additional harmonization filters might be necessary.
a.	Requirements: 
•	Python v3.7 (running environment)
•	burden_WH_3_1_2023.R (Appendix 2: TRAPD Script)
•	QQ_new_wh_10_12_2022.R (Appendix 2: TRAPD Script)
b.	Input:
•	Control_ synonymous_QC6.gene_count.txt
•	Case_ synonymous_QC6.gene_count.txt
c.	Output:
•	QQ_plot
•	Burden test.txt
The burden test result file is a tab delimited file with 12 columns:
1.	#GENE: gene name
2.	CASE_HET_IND_FREQ: Frequency of case individuals carrying at least one heterozygous qualifying variant in the gene.
3.	 CASE_HOM_IND_FREQ: Frequency of case individuals carrying at least one heterozygous qualifying variant in the gene.
4.	CASE_SMALLEST_IND_NUMBER: The smallest number of case individuals that pass the quality filter at the position of qualifying variants in the gene.
5.	CASE_HET_IND_EST_COUNT: Estimate number of case individuals carrying at least one heterozygous qualifying variant in the gene.
6.	CASE_HOM_IND_EST_COUNT: Estimate number of case individuals carrying at least one homozygous qualifying variant in the gene.
7.	CONTROL_HET_IND_FREQ: Frequency of control individuals carrying at least one heterozygous qualifying variant in the gene.
8.	CONTROL_HOM_IND_FREQ: Frequency of control individuals carrying at least one homozygous qualifying variant in the gene.
9.	CONTROL_SMALLEST_IND_NUMBER: The smallest number of control individuals that pass sequencing quality filters at the position of the qualifying variants in the gene.
10.	CONTROL_HET_IND_EST_COUNT: Estimate # of individuals carrying at least one heterozygous qualifying variant in the gene.
11.	CONTROL_HOM_IND_EST_COUNT: Estimate # of individuals carrying at least one homozygous qualifying variant in the gene.
12.	P_DOM: p-value under the dominant model.

d.	Command lines to run:
•	Run burden test R script.
“Rscript burden_WH_3_1_2023.R --casefile Case_ synonymous_QC6.gene_counts.txt --controlfile Control_ synonymous_QC6.gene_counts.txt --outfile QC6.synonymous.burden.txt”
•	sort result by P value (small to large)
“sort -g -k12 QC6.synonymous.burden.txt > QC6.synonymous.burden.sorted.txt”
•	Generate Q-Q Plot
“Rscript QQ_new_wh_10_12_2022.R --pvalfile QC6.synonymous.burden.txt --plotfile QC6.synonymous.burden.png”
→ Review point:
	Review QQ PLOT and evaluate evidence of artifacts/poor harmonization.
	Based on the different sets of data, additional filters and parameters may be needed.

Step3:  Burden Test to Identify High and Mod impact variants.
	Perform QC1 through QC6, except Filter1 from STEP2 with the same filters and parameters used for calibrating the synonymous variant Q-Q plots.
Filter1: Filter by Annotation (High impact protein-coding variants)
Aim: Retain only variants with a canonical High impact (by VEP annotation) protein-coding transcript.
Rationale: This step makes sure to include only variants with protein-coding transcripts that have impact of “High” in VEP annotation in the downstream analyses. VEP categorizes high impact variants into the following types: 1) Stop gained, 2) Frameshift, 3) Splice acceptor and donor, 4) Start lost, 5) Stop lost, 6) Transcript ablation, 7) Transcript amplification.
a.	Requirements: 
•	Python v3.7 (running environment)
•	Filter1_impact_Create_exclusion_SNPlist_by_annotation_10_18_2022.py (Appendix 2: TRAPD Script)
b.	Input: 
•	Case_QC6.vcf
•	Control_QC6.vcf
c.	Output: 
•	Case_HIGH_impact.vcf
•	Control_HIGH_impact.vcf
d.	Command lines to run:
•	Retain only variants with a canonical High impact protein-coding transcript.
“Python Filter1_impact_Create_exclusion_SNPlist_by_annotation_10_18_2022.py -v Case_QC6.vcf --excludeoutfile Case_HIGH_exclusion_SNPlist.txt -csq HIGH
python Filter1_impact_Create_exclusion_SNPlist_by_annotation_10_18_2022.py -v Control_QC6.vcf --excludeoutfile Control_HIGH_exclusion_SNPlist.txt -csq HIGH”
•	Combine the exclusion list.
“cat Case_HIGH_exclusion_SNPlist.txt Control_ HIGH_exclusion_SNPlist.txt > Filter1_HIGH_exclusion_SNPlist.txt”
•	Exclude from both case and control.
python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_QC6.vcf -o Case_HIGH_impact.vcf -m Case_HIGH.excluded_more_info.txt -s Filter1_HIGH_exclusion_list.txt
python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_QC6.vcf -o Control_HIGH_impact.vcf -m Control_HIGH.excluded_more_info.txt -s Filter1_HIGH_exclusion_list.txt
Filter1: Filter by Annotation (Moderate impact protein-coding variants)
Aim: Retain only variants with a canonical Moderate impact (by VEP annotation) protein-coding transcript.
Rationale: This step makes sure to include only variants with protein-coding transcripts that have impact of “Moderate” in VEP annotation in the downstream analyses. VEP categorizes moderate impact variants into the following types: 1) Inframe insertion; 2) inframe deletion, 3) Missense variant, and 4) Protein altering variant.
a.	Requirements: 
•	Python v3.7 (running environment)
•	Filter1_impact_Create_exclusion_SNPlist_by_annotation_10_18_2022.py (Appendix 2: TRAPD Script)
b.	Input: 
•	Case_QC6.vcf
•	Control_QC6.vcf
c.	Output: 
•	Case_MODERATE_impact.vcf
•	Control_ MODERATE_impact.vcf
d.	Command lines to run:
•	Retain only variants with a canonical MODERATE impact protein-coding transcript.
“Python Filter1_impact_Create_exclusion_SNPlist_by_annotation_10_18_2022.py -v Case_QC6.vcf --excludeoutfile Case_MODERATE_exclusion_SNPlist.txt -csq MODERATE
python Filter1_impact_Create_exclusion_SNPlist_by_annotation_10_18_2022.py -v Control_QC6.vcf --excludeoutfile Control_MODERATE_exclusion_SNPlist.txt -csq MODERATE”
•	Combine the exclusion list.
“cat Case_MODERATE_exclusion_SNPlist.txt Control_ MODERATE_exclusion_SNPlist.txt > Filter1_MODERATE_exclusion_SNPlist.txt”
•	Exclude from both case and control.
python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_QC6.vcf -o Case_MODERATE_impact.vcf -m Case_MODERATE.excluded_more_info.txt -s Filter1_MODERATE_exclusion_list.txt
python Exclude_snps_and_make_new_vcf_6-22-2022.py -v Case_QC6.vcf -o Control_MODERATE_impact.vcf -m Control_MODERATE.excluded_more_info.txt -s Filter1_MODERATE_exclusion_list.txt
	Perform Generate SNP list & Counting carriers in case and control, and then running burden test and Generate QQ plot.
Appendix in “TRAPD” script folder:
Appendix 1: VEP annotation JavaScript.
Appendix 2: TRAPD Scripts.
