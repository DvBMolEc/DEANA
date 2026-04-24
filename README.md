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

<img src="https://github.com/DvBMolEc/DEANA/blob/master/FlowChart_DEANA_20260423.png" width="600" />

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
DemuxKit: Kit name provided to dorado --kit-name
CustomBarcodesSequences: In case of custom barcode sequences, .fa file provided to dorado --barcode-sequences [.fa file] [optional]
CustomBarcodeArrangement: In case of custom barcode sequences, .toml file provided to dorado --barcode-arrangement [.toml file] [optional]
MinimumLength: Minimum read length after adapter, barcode and primer trimming. [integer]
MaximumLength: Maximum read length after adapter, barcode and primer trimming. [integer]
FW: Sequence forward primer [ACTGWSMKRYBDHVN]
RV: Sequence reverse primer REVERSE COMPLIMENT [ACTGWSMKRYBDHVN]
ErrorRate: Error rate used for cutadapt [numerical between 0 and 1]
OverlapFWPrimer: Minimum nt found by cutadapt in FW primer [numerical]
OverlapRVPrimer: Minimum nt found by cutadapt in RV primer [numerical]
Q: minimum Q-score [integer]
Dorado: /path/dorado/bin/
Cutadapt: /path/cutadapt/bin/
CDHIT: /path/cdhit/bin/
BLAST_DB: /path/BLAST_database/name
BLAST: /path/BLAST/bin/
TargetTaxa: Flag to only BLAST for target taxa [integer] (requires BLAST version 2.15+)
THREADS: allow multithreading to this number of threads [integer, default = 4]


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
| BLAST | Optional, needs additional install: [NCBI BLAST+](https://www.ncbi.nlm.nih.gov/books/NBK52640/) |
|  -B    | yourblastdatabase.fasta |
|  -b    | /path/to/existing/blast/database/existing-data-base-file.fasta |

## Warning
**Selecting the right stringency for the clustering setting is very important.** If it is set too low, species will be clustered together; if it is set too high, species will be lost as more singletons emerge. R9 and R10 data absolutely require different stringencies. Genetic markers with more or less genetic variation will also need to be clustered with higher or lower stringency. **It is advisable to conduct several small runs to determine the appropriate level of stringency for your amplicon.**

<img src="https://github.com/Saskia-Oosterbroek/decona/blob/master/cluster_settings.jpg" width="400" />
Comparison of DECONA cluster settings for two mitochondrial amplicons. The MiFish amplicon a ~ 170 bp fragment of the 12S rRNA gene and the 2Kb long amplicon partly covering 2Kb of the 12S & 16S rRNA gene.
The figure shows the number of unique species, clusters and the number of reads remaining when setting a certain minimal q‐score (Q 8 - 20) in combination with different minimal % identity thresholds for clusters (80 – 100%).  (A & B) Number of unique species, (C & D) number of unique clusters (E & F) the amount of reads that remain with each combination of minimal q‐score and clustering percentage.

*(This is figure 3 from our paper "The Long and the Short of It: Nanopore-Based eDNA Metabarcoding of Marine Vertebrates Works; Sensitivity and Species-Level Assignment Depend on Amplicon Lengths"
 DOI <a href="http://doi.org/10.1111/1755-0998.14079">10.1111/1755-0998.14079</a> )*


## Running Decona on the example data

To run Decona on the example data:
```sh
decona -f -l 800 -m 2100 -q 10 -c 0.80 -n 25 -M -i ~/computer/work_dir/example_data/
```
from within the directory `example_data/`. It will generate output in the directory `data/`.

