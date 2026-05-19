Rare Variant Analysis Pipeline
Version 03.23.2023

This repository contains a step-by-step pipeline for rare variant burden analysis using exome sequencing data, including variant calling, annotation, harmonization, quality control, and gene-based burden testing. The workflow was developed for rare disease and consortium-based genetic studies where datasets may originate from different sequencing platforms, cohorts, or analysis pipelines.

The pipeline emphasizes harmonization across heterogeneous datasets by applying standardized QC filters, annotation strategies, and burden testing procedures. It includes scripts for:

* joint variant calling
* VEP annotation
* coverage harmonization
* allele balance and frequency filtering
* synonymous variant calibration
* rare variant burden testing
* QQ plot generation
* gene-level carrier counting

This framework is particularly useful for multi-site consortium projects because each collaborating group can contribute their own analysis scripts, QC workflows, and notes within a centralized and version-controlled environment. GitHub allows all changes and updates to be tracked over time, improving transparency, reproducibility, and replication across groups even when different sequencing platforms or primary pipelines are used.

Each folder/repository can function like a shared project workspace where collaborators document exactly what was done, what filters were applied, and how variants were interpreted, making downstream comparison and harmonization much easier.

The repository is intended for research and collaborative development purposes and can be adapted to different cohorts, sequencing platforms, and rare disease studies.

Pipeline document adapted from rare variant burden testing workflows used in consortium-based exome sequencing studies. 
