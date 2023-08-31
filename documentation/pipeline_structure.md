## Get to know the pipeline structure:

The folders in the Kuura pipeline github repository are listed below:

>- dnaseq
>- docker
>- documentation
>- LICENSE
>- README.md

### The files in the dnaseq folder are listed below:
>- dnaseq/
>   - bin/
>        - main.nf
>   - nextflow.config
>   - configuration/
>        - execution-parameters.config
>       - default.config
>       - testing.config


**1.1: bin/main.nf:**

This is the main script file describing the sequence of the analysis to be done and defines the parameters for the run. Any addition/removal of analysis steps are done in this file. An example process is given below,

```bash
process QualityAssessment {

        label "utuprcagenetics"

        maxForks params.lightResourceForkThreshold

        tag "FASTQC on $fastq"

        publishDir "$params.outputDir/FASTQC"

        input:
        file(fastq) from allFastq_ch

        output:
        file("${outputZIP}")
        file("${outputHTML}")
	val("DoneInitialFastQC") into doneFastQC        

        script:
        outputZIP = "${fastq}".replaceFirst(/.fastq.gz$/, "_fastqc.zip")
        outputHTML = "${fastq}".replaceFirst(/.fastq.gz$/, "_fastqc.html")

        """
        fastqc -q -o . ${fastq}
        """
}
```

In the above example, the process is named *QualityAssessment*. The parameters defined inside the process are as follows;

1. label - The label directive allows the user to annotate a process with a specific keyword. In this pipeline we make use of the label directive with ‘withLabel’ process selector (step 2.3) to define docker images to be used for the specific process.

2. maxForks - Maximum number of process instances that can be executed in parallel. The parameters `params.highResourceForkThreshold` and `params.lightResourceForkThreshold` can be used to adjust the number of forks for a process depending on the computational resources available. There are other directives such as maxErrors, maxRetries and memory that can be used to modify the resources made available for running the pipeline.

3. tag - tag directive enables association of each process execution with custom label. This label makes it easier to identify processes in log file and execution summaries.

4. publishDir - This directive allows defining the output folder for that specific process.

5. input - Input block defines from which channels the process expects to receive data. There can be more than one input for a process but not more than one input block. The basic structure of the input block is,

```bash
<input qualifier> <input name> [from <source channel>] [attributes]
```
- input qualifier - refers to the types of input that is expected.

- input name - refers to the name of the expected input.

- from - keyword to specify source of input.

- source channel - channel from which input is expected.

- attributes - optional attributes to select input can be specified using this directive.

More information on the input block can be found from - https://www.nextflow.io/docs/latest/process.html#inputs

6. output - Output block defines the channels that carry data from the process once it is complete. There can be more than one output for a process and they can be defined together in one output block. The basic structure of the output block is as follows;

```bash
<output qualifier> <output name> [into <target channel>[,channel,..]] [attribute [,..]] 
```
- output qualifier - refers to the types of output that is expected. 
- output name - refers to the name of the expected output. 
- into - keyword to specify output to channel. 
- target channel - channel over which output is sent to next process. 
- attributes - optional conditional attributes for specifying outputs to channels.

More information on the output block can be found from - https://www.nextflow.io/docs/latest/process.html#outputs

7. script - Script block defines the command that should be executed by the process to carry out its task. More information on scripts can be found from - https://www.nextflow.io/docs/latest/process.html#script

8. Shell (execution) block - The shell block is denoted by triple quotes and the shell commands to be executed by the process are defined here.

In-depth information on defining a process can be obtained from - https://www.nextflow.io/docs/latest/process.html#

**1.2 nextlfow.config:**

The `nextflow.config` file contains scripts to include custom config profiles for running the analysis. A custom config file can be defined as follows,

```bash
Profiles {
	custom {
		includeConfig 'configuration/custom.config'
		includeConfig 'configuration/execution-parameters.config'
	}
}
```

Profiles can also be defined to include information such as the process executor, memory and so on. More information on creating custom profiles and their scopes is available from *nextflow* documentation (https://www.nextflow.io/docs/latest/config.html?highlight=params#).

**1.3 configuration/execution-parameters.config:**

The execution-parameters.config file contains information on the containers and the containerization technology (*docker*) that is used for running the pipeline. The file is divided into two blocks - *docker* configurations and *docker* profiles for specific processes.

The parameters for the *docker* container can be defined in the first block, more information on *docker* scope can be found from https://www.nextflow.io/docs/latest/config.html#config-docker.

```bash
docker {
        enabled = true
        temp = 'auto'
        runOptions = "--volume /mount/:/mount/ -u \$(id -u):\$(id -g)"		
}
```

The second block contains process selectors for defining *docker* containers to be used with processes tagged with specific labels.

```bash
process {
    withLabel: 'utuprcagenetics' {
        container = 'dna-seq-pipeline:0.1'
	}
}
```

**1.4 configuration/default.config & configuration/testing.config:**

This is the configuration profile where the parameters for running the pipeline are defined. The parameters are listed in section **Configure the *nextflow* parameters**.

### The files in the *docker* folder are listed below:

>- docker/
>    - dockerfile

The *docker* folder contains the dockerfile that is used to build a custom *docker* image containing tools used in the pipeline that do not have official *docker* image from developers.

### The files in the documentation folder are listed below:

>- documentation/
>   - pipeline_structure.md
>   - figures/
>       - Fig1.png

This folder contains documentation and figures explaining the kuura pipeline.

### Configuring the *nextflow* parameters

| Parameters                   | Description                                                                                                              |
|:------------------------------:|:--------------------------------------------------------------------------------------------------------------------------:|
| params.outputDir             | Full path to the directory where the soft links of the files are stored.                                                 |
| params.groupIdentifier       | Unique identifiers for your sample.                                                                                      |
| params.libraryType           | Specify whether the sample is WES/WGS.                                                                                   |
| params.seqPlatform           | Specify sequencing platform.                                                                                             |
| params.noAdaptor             | If set to TRUE, uses *fastp* for adapter trimming, if set to FALSE uses *cutadapt* and adapters mentioned in params.adaptors |
| params.exomeRegionsBed       | Exome targeted regions in the form of a bed file.                                                                        |
| params.dataDir               | Full path to the directory where the input files are stored.                                                             |
| params.allFastq              | Full path to the fastq files identifying them with a glob pattern.                                                       |
| params.reads                 | Full path to the fastq files with glob pattern to identify matching reads.                                               |
| params.adaptors              | Adapter information present in the sample.                                                                               |
| params.alignmentReferenceDir | Full path to the directory where the FASTA reference file is stored.                                                     |
| params.alignmentReference    | Full path to the FASTA reference file.                                                                                   |
| params.dbSNPcommon           | dbSNP filter of common variants.                                                                                         |
| params.millsReference        | Databases of known polymorphic sites used to exclude regions around known polymorphisms from analysis                    |
| params.dbSNPReference             | dbSNP database (Human variations without clinical assertions).        |
| params.hapmapReference            | HapMap genotypes and sites.                                           |
| params.omniReference              | omni_2.5 genotypes for 1000 genomes samples and sites.                |
| params.reference1000G             | 1000 genomes project high confidence snps.                            |
| params.knownIndels                | Broad Institute list of know indels vcf.                              |
| params.heavyResourceForkThreshold | Maximum number of process instances that can be executed in parallel. |
| params.lightResourceForkThreshold | Maximum number of process instances that can be executed in parallel. |