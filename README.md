Rare Variant Analysis Pipeline
Version 03.23.2023

This repository contains a step-by-step workflow for rare variant burden analysis using exome sequencing data. The pipeline includes joint variant calling, VEP annotation, quality control, coverage harmonization, variant filtering, carrier counting, and gene-based burden testing.

The workflow is designed to support harmonized analysis across datasets generated from different sequencing platforms and cohorts. Quality control steps include coverage assessment, PASS filtering, allele balance filtering, allele frequency filtering, and calibration using synonymous variants to evaluate potential technical artifacts and case-control harmonization.

The pipeline supports both individual-level control data and gnomAD summary statistics and includes scripts for generating SNP lists, estimating carrier frequencies, performing burden testing, and generating QQ plots for quality assessment.

Methods included:

* Joint variant calling (GATK)
* VEP functional annotation
* Coverage harmonization
* Variant-level QC filtering
* Rare variant filtering by population frequency
* Gene-level carrier counting
* Rare variant burden testing
* QQ plot evaluation for calibration and harmonization

The repository is intended for reproducible and collaborative rare disease and consortium-based genetic analyses.
