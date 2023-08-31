# Kuura - An automated workflow for analyzing WES and WGS data ##

Kuura is an open-source application for automated sequencing data analysis. The pipeline is written in nextflow, and the running environment is provided in a docker container to make them scalable and reproducible. This README provides information on how to setup and run Kuura pipeline for DNA-seq data analysis.

**ABSTRACT:**

The advent of high-throughput sequencing technologies has revolutionized the field of genomic sciences by cutting down the cost and time associated with standard sequencing methods. This advancement has not only provided the research community with an abundance of data but has also presented the challenge of analyzing it. The paramount challenge in analyzing the copious amount of data is in using the optimal resources in terms of available tools. To address this research gap, we propose “Kuura - An automated workflow for analyzing WES and WGS data”, which is optimized for both whole exome and whole genome sequencing data.

This workflow is based on the nextflow pipeline scripting language and uses docker to manage and deploy the workflow. The workflow consists of four analysis stages - quality control, mapping to reference genome & quality score recalibration, variant calling & variant recalibration and variant consensus & annotation. An important feature of the DNA-seq workflow is that it uses the combination of multiple variant callers (GATK Haplotypecaller, DeepVariant, VarScan2, Freebayes and Strelka2), generating a list of high-confidence variants in a consensus call file.

The workflow is flexible as it integrates the fragmented tools and can be easily extended by adding or updating tools or amending the parameters list. The use of a single parameters file enhances reproducibility of the results. The ease of deployment and usage of the workflow further increases computational reproducibility providing researchers with a standardized tool for the variant calling step in different projects. The source code, instructions for installation and use of the tool are publicly available at our github repository https://github.com/dhanaprakashj/kuura_pipeline.


## Setting up and running the kuura pipeline


**Step 1:** Install and configure *nextflow* & *docker*.

**Step 2:** Clone the source codes from the github repository.

**Step 3:** Download sample datasets. The publicly available datasets used in the evaluation were downloaded from https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/: 

*HG001_R1 -* https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/NA12878/Garvan_NA12878_HG001_HiSeq_Exome/NIST7035_TAAGGCGA_L001_R1_001.fastq.gz ; https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/NA12878/Garvan_NA12878_HG001_HiSeq_Exome/NIST7035_TAAGGCGA_L002_R1_001.fastq.gz

*HG001_R2 -* https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/NA12878/Garvan_NA12878_HG001_HiSeq_Exome/NIST7035_TAAGGCGA_L001_R2_001.fastq.gz ; https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/NA12878/Garvan_NA12878_HG001_HiSeq_Exome/NIST7035_TAAGGCGA_L002_R2_001.fastq.gz

*HG002_R1 -* https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/AshkenazimTrio/HG002_NA24385_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG002-23100077/MPHG002_S1_L001_R1_001.fastq.gz

*HG002_R2 -* https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/AshkenazimTrio/HG002_NA24385_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG002-23100077/MPHG002_S1_L001_R2_001.fastq.gz

*HG005_R1 -* https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/ChineseTrio/HG005_NA24631_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG005-23100080/MPHG005_S4_L004_R1_001.fastq.gz

*HG005_R2 -* https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/data/ChineseTrio/HG005_NA24631_son/NIST_Stanford_Illumina_6kb_matepair/fastqs/MPHG005-23100080/MPHG005_S4_L004_R2_001.fastq.gz

**Step 4:** Setting up required *docker* images (need root permission):

The pipeline uses official docker images for tools which are made available by the developers (*GATK*, *mosdepth*, *deepvariant* & *ensembl-vep*) and for the remaining tools a custom *docker* image has been developed. 

- Pull images from DockerHub

```bash
docker pull broadinstitute/gatk
docker pull quay.io/biocontainers/mosdepth:0.2.4--he527e40_0
docker pull google/deepvariant:1.4.0
docker pull ensemblorg/ensembl-vep
```
- Build custom *docker* image

After cloning the github repository, the *docker* image required for running the pipeline needs to be built. 

```bash
cd kuura_pipeline/docker/
docker build -t dna-seq-pipeline:0.1 .
```
- Check the images and tags to verify if all the required images are available.

```bash
docker image ls
```
**Step 5:** 

Configure the required variables for running the pipeline. In the configuration folder there is a file called `default.config` where variables such as input data directory, output directory, source data for various processes, data descriptors, etc., need to be completed. More information on editing the `default.config` file is provided in pipeline structure section.

```bash
cd ..
cd kuura_pipeline/configuration/
<text editor of choice> default.config
```

**Step 5:** Running the pipeline:

After defining all the parameters, run the pipeline by executing;

```bash
cd .. 
nextflow run bin/main.nf -work-dir <workspace directory> -profile default
```
Upon executing the above command, nextflow will run the pipeline in a docker container using the docker images predefined in the `execution-parameters.config` file and store the intermediate result files in the directory specified with the `-work-dir` option and their symbolic links in the respective output folder defined in `default.config` file.
