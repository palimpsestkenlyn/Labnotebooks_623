
## Environment

```
SRA Toolkit 3.4.1 (`/projects/bgmp/hodapp/sra_toolkit/sratoolkit.3.4.1-alma_linux64/bin`)
```

## Log
Install or download of the toolkit?

So even though it looks like SRA is already on talapas, I think the idea is for us to do the install to practice so I am doing it anyways.
```
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo) 09:45 PM $ module avail sra

---------------------------------------------------------- /packages/modulefiles/t2/modulefiles -----------------------------------------------------------
   sratoolkit/3.4.1

If the avail list is too long consider trying:

"module --default avail" or "ml -d av" to just list the default modules.
"module overview" or "ml ov" to display the number of modules for each name.

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".


TALAPAS login2 (/projects/bgmp/hodapp/bioinfo) 09:50 PM $  module load sratoolkit/3.4.1
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo) 09:52 PM $ which prefetch
/packages/sratoolkit/3.4.1/bin/prefetch
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo) 09:52 PM $ 
```

Making a directory in my bgmp folder: /projects/bgmp/hodapp/sra_toolkit

```
wget --output-document sratoolkit.tar.gz https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-alma_linux64.tar.gz
```
Talapas uses alma_linux...

```
tar -vxzf sratoolkit.tar.gz 
```
creates a long list of tools now in the directory. Successfully installed

However talapas won't know what these commands are, so have to tell it where to look:

```
export PATH=/projects/bgmp/hodapp/sra_toolkit/sratoolkit.3.4.1-alma_linux64/bin:$PATH
```
This has to be redone each time to tell talapas when I say fetch, go here to look for what that command is....? Seems so. 

test to make sure it is working as outlined on github:

```
which fastq-dump
/projects/bgmp/hodapp/sra_toolkit/sratoolkit.3.4.1-alma_linux64/bin/fastq-dump

fastq-dump --stdout -X 2 SRR390728
Read 2 spots for SRR390728
Written 2 spots for SRR390728
@SRR390728.1 1 length=72
CATTCTTCACGTAGTTCTCGAGCCTTGGTTTTCAGCGATGGAGAATGACTTTGACAAGCTGAGAGAAGNTNC
+SRR390728.1 1 length=72
;;;;;;;;;;;;;;;;;;;;;;;;;;;9;;665142;;;;;;;;;;;;;;;;;;;;;;;;;;;;;96&&&&(
@SRR390728.2 2 length=72
AAGTAGGTCTCGTCTGTGTTTTCTACGAGCTTGTGTTCCAGCTGACCCACTCCCTGGGTGGGGGGACTGGGT
+SRR390728.2 2 length=72
;;;;;;;;;;;;;;;;;4;;;;3;393.1+4&&5&&;;;;;;;;;;;;;;;;;;;;;<9;<;;;;;464262
```

yes seems to be working. 

Set up for download. Have decided to do outputs in the current working directory so
```
vdb-config --prefetch-to-cwd
Prefetch will download to Current Directory when Public User Repository is set
```
Then will add the path in the SLURM script.

check files sizes to see that they do not exceed the 20GB limit that needs special handling. Welp. SRR1302052 is over the 20GB size limit and will need the maximum adusted via the flag. 

```
SRR11722023:
|Run|# of Spots|# of Bases|Size|Published|
|---|--:|--:|--:|--:|
|[SRR11722023](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11722023)|52,844,920|15.9G|5.4Gb|2021-06-30|


SRR1302052:
|Run|# of Spots|# of Bases|Size|Published|
|---|--:|--:|--:|--:|
|[SRR1302052](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR1302052)|300,773,959|59.6G|34.9Gb|2015-07-22|
```
SLURM script: merun_SRA.sh
```
#!/bin/bash

#SBATCH --account=bgmp

#SBATCH --partition=bgmp

#SBATCH --cpus-per-task=8

#SBATCH --mem=16G

#SBATCH --job-name=sra_download

#SBATCH --output=sra_download_%j.out
 

#point to where the SRA tools are installed...
export PATH=/projects/bgmp/hodapp/sra_toolkit/sratoolkit.3.4.1-alma_linux64/bin:$PATH

# Move into output directory since configured with $vdb-config --prefetch-to-cwd

cd /projects/bgmp/hodapp/bioinfo/Bi623/ICA/palimpsestkenlyn-Bi623-ICA1/SRA_outputs
  

# Download SRA files (prefetch creates a directory per accession) Must increase the max for second SRR as it is above 20GB limit

/usr/bin/time -v prefetch SRR11722023
/usr/bin/time -v prefetch SRR1302052 -X 40G SRR1302052

  

# Convert to FASTQ (fasterq-dump at the prefetched directories)
/usr/bin/time -v fasterq-dump SRR11722023 --threads 8
/usr/bin/time -v fasterq-dump SRR1302052 --threads 8

  

# Compress FASTQ files
/usr/bin/time -v gzip *.fastq
```

Submitted batch job 46644316. cancelled file was too big. Second attempt:
Submitted batch job 46644493

## Usage 
Usage and outputs summarized from `sra_download_466444493.out `(`/projects/bgmp/hodapp/bioinfo/Bi623/ICA/palimpsestkenlyn-Bi623-ICA1/sra_download_46644493.out`)

```
Command being timed: "prefetch SRR11722023"
Percent of CPU this job got: 4%
Elapsed (wall clock) time (h:mm:ss or m:ss): 0:02.39
Maximum resident set size (kbytes): 98952
Exit status: 0

Command being timed: "prefetch SRR1302052 -X 40G SRR1302052"
Percent of CPU this job got: 27%
Elapsed (wall clock) time (h:mm:ss or m:ss): 19:48.47
Maximum resident set size (kbytes): 141752
Exit status: 0

Command being timed: "fasterq-dump SRR11722023 --threads 8"
Percent of CPU this job got: 135%
Elapsed (wall clock) time (h:mm:ss or m:ss): 2:57.12
Maximum resident set size (kbytes): 1065512
Exit status: 0

Command being timed: "fasterq-dump SRR1302052 --threads 8"
Percent of CPU this job got: 131%
Elapsed (wall clock) time (h:mm:ss or m:ss): 14:16.51
Maximum resident set size (kbytes): 1099596
Exit status: 0

Command being timed: "gzip SRR11722023_1.fastq SRR11722023_2.fastq SRR1302052_1.fastq SRR1302052_2.fastq"
Percent of CPU this job got: 94%
Elapsed (wall clock) time (h:mm:ss or m:ss): 5:11:42
Maximum resident set size (kbytes): 1972
Exit status: 0
```

## Outputs
Output summary:
```
**fasterq-dump outputs:** paired end reads (2 fastq filer per)
- SRR11722023: 52,844,920 spots / 105,689,840 reads
- SRR1302052: 300,773,959 spots / 601,547,918 reads
```