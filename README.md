# DEANA

This pipeline is intended as decona 2.0, where ONT sequencing data is transformed into ASVs, instead of clustered into concensus sequences. The main functions are 1) demultiplexing (nanopore sequence data), 2) trimming (adaptbers/barcodes/primers), 3) merging (ASVs) and 4) identifying (species)

**To Do Version 0.1**
- Update pipeline functions
- Silence unnecessary and redundant lines in decona script
- Create wiki
- Update .README
- Create example .config file

##  From sequencing to species identification for Nanopore amplicon data
DEANA can process multiple samples in one line of code:
- Mixed samples containing multiple species from bulk and eDNA
- Mixed amplicons in one barcode
- Multiplexed barcodes
- Multiple samples in one run
- Outputs ASVs, corresponding read counts and corresponding species identifications

<img src="https://github.com/DvBMolEc/DEANA/blob/main/FlowChart_DEANA_20260423.png" width="600" />

## Installation
DEANA is only supported for use with Linux, the Ubuntu command line app for Windows also works but is recommended only for use with smaller datasets.
DEANA is sensitive to installation version of dependencies. 

## Dependencies

DEANA works with the following sequence processing tools:
| Tool | version |  function |
| ------ | ------ | ------ |
| NanoFilt | X.X | Filter reads based on Q-score |
| dorado | 1.3.0 | Demulitplex ONT sequencing data |
| dorado | 1.3.0 | Trim adapters and barcodes |
| Cutadapt | X.X | Trim primer sequences |
| CD-hit | X.X.X | Merge ASVs within samples |
| Blast | X.X.X | Blast ASVs |


## Usage
**DEANA works on basecalled .bam files in your working directory.** It is a good idea to have an empty directory with just the files you want to run. A Demux folder will be made for demultiplexed data. 
DEANA requires a .config file in your working directory which contains:


| Variable | Description | Allowed value(s) | Optional? |
| ------ | ------ | ------ | ------ |
| DemuxKit | Kit name provided to dorado --kit-name | SQK-LSK114 or Custom names | No |
| CustomBarcodesSequences | In case of custom barcode sequences, .fa file provided to dorado --barcode-sequences | .fa file | Yes |
| CustomBarcodeArrangement | In case of custom barcode sequences, .toml file provided to dorado --barcode-arrangement | .toml file | Yes |
| MinimumLength | Minimum read length after adapter, barcode and primer trimming. | integer | No |
| MaximumLength | Maximum read length after adapter, barcode and primer trimming. | integer | No |
| FWPrimer | Sequence forward primer | ACTGWSMKRYBDHVN | No |
| RVPrimer | Sequence reverse primer REVERSE COMPLIMENT | ACTGWSMKRYBDHVN | No |
| ErrorRate | Error rate used for cutadapt | numerical between 0 and 1 | No |
| OverlapFWPrimer | Minimum nt found by cutadapt in FW primer | numerical | No |
| OverlapRVPrimer | Minimum nt found by cutadapt in RV primer | numerical | No |
| Q | minimum Q-score | integer | No |
| Dorado | /path/dorado/bin/ | Any path | No |
| Cutadapt | /path/cutadapt/bin/ | Any path | No |
| CDHIT | /path/cdhit/bin/ | Any path | No | 
| BLAST_DB | /path/BLAST_database/name | Any path | No |
| BLAST | /path/BLAST/bin/ | Any path | No |
| TargetTaxa | Only BLAST target taxa | integer | Yes, requires BLAST version 2.15+ |
| THREADS | allow multithreading to this number of threads | integer | Yes, default = 4 |


Example
```sh
$ deana -i input_folder/input_file.bam -c /path/configfile.config
```
Where the .config file looks like the example:
DemuxKit: MiFish_9bp_tags
CustomBarcodesSequences: /path/MiFish_9bp_tags.fa
CustomBarcodeArrangement: /path/MiFish_9bp_tags.toml
MinimumLength: 150
MaximumLength: 200
FW: GTYGGTAAAWCTCGTGCCAGC
RV: CAAACTRGGATTAGATACCCCACTAT
ErrorRate: 0.1
OverlapFWPrimer: 17
OverlapRVPrimer: 21
Q: 10
Dorado: /path/dorado-1.3.0/bin/
Cutadapt: /path/cutadapt/bin/
CDHIT: /path/cdhit/bin/
BLAST_DB: /path/BLAST_database/core_nt
BLAST: /path/BLAST_2.17/bin/
TargetTaxa: 33208
Threads: 24

Will: Demultiplex a custom barcode kit using dorado 1.3.0, filter for read length 150-200 and quality score 10, trim primers and BLAST sequences in the core_nt database, only finding metazoans.

| Command | Function |
| ------ | ------ |
| -h   | help |
|  -i   | input folder |
|  -c    | path to the config file |
|  -p    | plot readlength distribution histogram (plots then exits program)|
|  -f    | folder structure: your dat is already  demultiplexed and stored in barcode folders (such as output from Mk1C)|
