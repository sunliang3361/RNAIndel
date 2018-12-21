# RNAIndel
RNAIndel calls coding indels and classifies them into 
somatic, germline, and artifact from a tumor RNA-Seq data.
Users can also classify indels called by their own callers by
supplying a VCF file.

## Table of Contents
**[Citations](#citations)**<br>
**[Prerequisites](#prerequisites)**<br>
**[Download](#download)**<br>
**[Installation](#installation)**<br>
**[Input BAM file](#input-bam-file)**<br>
**[Usage](#usage)**<br>
**[Preparation of non-somatic indel panel](#preparation-of-non-somatic-indel-panel)**<br>

## Citations
1. RNAIndel (in preparation).

2. Edmonson, M.N., Zhang, J., Yan, C., Finney, R.P., Meerzaman, D.M., and Buetow, K.H. Bambino: A Variant Detector 
and Alignment Viewer for next-Generation Sequencing Data in 
the SAM/BAM Format. Bioinformatics 27.6 (2011): 865–866. 
DOI: [10.1093/bioinformatics/btr032](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3051333/)

## Prerequisites
* [python>=3.5.2](https://www.python.org/downloads/)
    * [pandas>=0.23.0](https://pandas.pydata.org/)
    * [numpy>=1.12.0](https://www.scipy.org/scipylib/download.html)
    * [scikit-learn=0.18.1](http://scikit-learn.org/stable/install.html#)
    * [pysam=0.15.1](https://pysam.readthedocs.io/en/latest/index.html)
    * [pyvcf=0.6.8](https://pyvcf.readthedocs.io/en/latest/index.html)
* [java=1.8.0_66](https://www.java.com/en/download/) (required for Bambino only)


## Download
```
git clone https://github.com/adamdingliang/RNAIndel/tree/master  # Clone the repo
```

## Installation
It is highly recommended to setup a virtual python environment using [conda](https://conda.io/docs/) and install 
the python dependencies in the virtual environment:
```
conda create -n py36 python=3.6 anaconda    # Create a python3.6 virtual environment
source activate py36                        # Activate the virtual environment
pip install -r requirements.txt             # Install python dependencies
```

You can install RNAIndel from source directly:
```
cd RNAIndel                 # Switch to source directory
python setup.py install     # Install bambino and rna_indel from source
bambino -h                  # Check if bambino works correctly
rna_indel -h                # Check if rna_indel works correctly
```

## Data directory set up
Download [data_dir.tar.gz](http://ftp.stjude.org/pub/software/RNAIndel/data_dir.tar.gz).<br>
Place the gziped file under your working directory and unpack it.<br>
```
tar xzvf data_dir.tar.gz
```

## Input BAM file
Please prepare your BAM file as follows:<br>

Step 1. Map your reads with the STAR 2-pass mode to GRCh38.<br>
Step 2. Add read groups, sort, mark duplicates, and index the BAM file with Picard.<br>

Please input the BAM file from Step 2 without caller-specific preprocessing such as indel realignment.<br>
Additional processing steps may prevent desired behavior.

## Usage
The RNAIndel pipeline consists of indel calling and classification components, which are performed by two separate excutables.<br>
The pipeline is provided in [BASH](#use-bash-wrapper) or [CWL](#use-cwl-wrapper).<br>
Run-examples using sample data is [here](./sample_data).<br> 

### Use BASH wrapper
```
rna_indel_pipeline.sh -b input.bam \<br>
                      -o output.vcf \<br>
                      -f reference.fa \<br>
                      -d path/to/data_dir\<br>
                      [other options]
```
When a VCF file is supplied by -c, indel entries in the VCF file are used for classification (indel calling will not be performed).
```
rna_indel_piepline.sh -b input.bam -c input.vcf -o output.vcf -f reference.fa -d path/to/data_dir [other options]
```
#### Options
* ```-b``` input BAM file (required)
* ```-c``` VCF file from other caller (required for using other callers, e.g., [GATK](https://software.broadinstitute.org/gatk/))
* ```-o``` output VCF file (required)
* ```-f``` reference genome (GRCh38) FASTA file (required)
* ```-d``` data directory contains refgene, dbsnp and clinvar databases (required) [Data directory set up](#data-direcotry-set-up) 
* ```-q``` STAR mapping quality MAPQ for unique mappers (default=255)
* ```-p``` number of cores (default=1)
* ```-m``` maximum heap space (default 6000m)
* ```-n``` user-defined panel of non-somatic indels in VCF format
* ```-l``` direcotry to store log files 
* ```-h``` show usage message

### Use CWL wrapper
```
To do
```

Users can run the executables separately.<br>

### Indel calling
```
bambino -i input.bam -f reference.fa -o bambino_call.txt [optional argument (-m)]
```
#### Bambino options
* ```-m``` maximum heap space (default 6000m)
* ```-b``` input BAM file (required)
* ```-f``` reference genome FASTA file (required)
* ```-o``` Bambino output file. Tab-delimited flat file (required)

### Indel classification
#### Classification of Bambino calls
```
rna_indel -b input.bam -i bambino_call.txt -o output.vcf -f reference.fa -d path/to/data_dir [optional arguments]
```
#### Classification of calls from other callers
```
rna_indel -b input.bam -c input.vcf -o output.vcf -f reference.fa -d path/to/data_dir [optional arguments]
```

#### RNAIndel options
* ```-b``` input BAM file (required)
* ```-i``` Bambino output file (required for using Bambino as the indel caller)
* ```-c``` VCF file from other caller (required for using other callers, e.g., [GATK](https://software.broadinstitute.org/gatk/))
* ```-o``` output VCF file (required)
* ```-f``` reference genome (GRCh38) FASTA file (required)
* ```-d``` path to data directory contains refgene, dbsnp and clinvar databases [Data directory set up](#data-direcotry-set-up) 
* ```-q``` STAR mapping quality MAPQ for unique mappers (default=255)
* ```-p``` number of cores (default=1)
* ```-n``` user-defined panel of non-somatic indels in VCF format

## Preparation of non-somatic indel panel
Users may want to prevent a certain set of indels from being classified as somatic. Users can prepare such a user-define exlusion panel to improve 
somatic prediction. RNAIndel reclassifies indels predicted as somatic to either germline or artifact class, whichever has the higher probability, if 
they are found in the panel. As an example, a panel of recurrent non-somatic indels compiled from 
a cohort of 330 samples with RNA-Seq and tumor/normal-paired WGS&WES is provided in [data_dir.tar.gz](http://ftp.stjude.org/pub/software/RNAIndel/data_dir.tar.gz).<br> 
<br>
This panel can be applied by appending the following [option](#rnaindel-options) to the RNAIndel command: <br>
```
-n path/to/data_dir/non_somatic/non_somatic.vcf.gz
```

Alternatively, you can complie your own panel as follows:<br>
### from normal RNA-Seq data
1. Prepare a matched normal RNA-Seq dataset.<br>
2. Perform variant calling and generate a VCF file.<br>
3. Index the VCF file with Tabix.<br>
### from tumor RNA-Seq and paired DNA-Seq data
1. Prepare a dataset with RNA-Seq and DNA-Seq performed. <br>
2. Apply RNAIndel and collect indels predicted somatic. <br>
3. Validate the indels with the DNA-Seq data. <br>
4. Collect indels which are validated as germline or artifact in N samples or more (recurrent non-somatic indels). <br>   
5. Format the recurrent non-somatic indels in VCF format.<br>
6. Index the VCF file with Tabix.<br>     
<br>

