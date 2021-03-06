# nCoV_Finder

## Introduction
nCoV Finder is a pipeline for HCoV-19 genome analyzing. The pipeline could  efficiently classify CoV-like reads from Massively Parallel Sequencing (MPS) data with Kraken, and get the virus genome with SPAdes and Pilon.

![Image](https://github.com/BGI-IORI/nCoV/blob/master/Image.png)

## Requirements:
perl: v5.22.0  
python: v2.7.16   
java: v1.8.0  

Software for CoV-like reads classification:  
* Kraken v1.1 (https://github.com/DerrickWood/kraken)  

Softwares for data quality control:  
* Fastp v0.19.5 (https://github.com/OpenGene/fastp)
* SOAPnuke v1.5.6 (https://github.com/BGI-flexlab/SOAPnuke)  

Software for low complexity reads removing:
* PRINSEQ v0.20.4 (http://prinseq.sourceforge.net/)  

Software for virus genome *De novo* assembly:  
* SPAdes v3.14.0 (http://cab.spbu.ru/software/spades/)  

Software for reference based consensus construction
* Pilon v1.23 (https://github.com/broadinstitute/pilon)  

Other required softwares:  
* BWA v0.7.16 (https://github.com/lh3/bwa)
* Picard v2.10.10 (https://broadinstitute.github.io/picard/)
* Samtools v1.9 (http://samtools.sourceforge.net/)
* bedtools v2.23.0 (https://bedtools.readthedocs.io/en/latest/)

## Installation
```
git clone https://github.com/BGI-IORI/nCoV.git
```
Notes: The above dependent software needs to be installed separately according to their instructions. After installing, the users should edit the input.config file, and change the software path to your own path.


## Usage
### 1.Build the index for database
Notes: Two sequence databases are included in the pipeline, and the users can customize their own databases. 
* CoV.fa, include coronaviridae virus sequences for Kraken classification. 
* HCoV-19.fa, include the reference genome of HCoV-19.  

1)Build Kraken database index:
```
kraken-build --build --threads 8 --db ./YourDBpath/ 
#Notes: Detailed description about Kraken index can be found in the 
#website http://ccb.jhu.edu/software/kraken/MANUAL.html#custom-databases.
#Brefily, the user should put CoV.fa file in the fold named "library" in "./YourDBpath/",  
#Download taxonomy file (ftp://ftp.ncbi.nih.gov/pub/taxonomy) from NCBI and put it in "./YourDBpath/".
#And then run the command to build the kraken index.
```
2)Build BWA index:
```
bwa index HCoV-19.fa
```
3)Build samtools index:
```
samtools faidx HCoV-19.fa
```
4)Edit the input.config file, and change the database path to your own path.

### 2.Run the pipeline.
```
perl nCoV_Finder.pl -i data.txt -c input.config -o ./outpath/
cd ./outpath/shellall/
sh allDependent.sh
#Notes: data.txt includes three columns:  sample_name seq.1.fq.gz seq2.fq.gz
```
## Output
1.*De novo* assembly from SPAdes
```
#original fasta file from SPAdes
./outpath/05.ASS/sample/scaffolds.fasta   
#longest contig
./outpath/05.ASS/sample/scaffolds_longest.fasta 
```
2.Consensus sequence from Pilon
```
#original fasta file from pilon
./outpath/06.CNS/sample/sample.pilon.fasta 
#consensus after masked position with depth lower 10X
./outpath/06.CNS/sample/sample.masked.fasta
```
## Additional Information
For *De novo* assembly, if too much data was left after “Remove low complexity reads”, to reduce the burden of computing, the data can be downsized to a certain amount (Such as data amount equivalent to about 100X of HCoV-19 genome).

For consensus from pilon, the default depth cutofff was set to 10X, and the position with depth lower than 10X would be masked to N.
