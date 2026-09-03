


vdb-dump- filename SRR -info will return info in bits so have to convert

### 8.27.26

found the SRR files assigned to me.
SRR25630303     KenlynHodapp
SRR25630398     KenlynHodapp

ran /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/merun_SRAdownload.sh

to SBATCH and prefetch, fasterq-dump SRR25630303, and fastqc, and then zip my files

Issue bc fastqc command not found. try with pixi run and redo on a srun interactive

pixi run fastqc SRR* -o /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/fastqc_outputs

forgot usr/bin/time but can tell time run from:
11:26 AM $ pixi run fastqc SRR* -o /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/fastqc_outputs

Approx 60% complete for SRR25630398_1.fastq.gz
Approx 65% complete for SRR25630398_1.fastq.gz
Approx 70% complete for SRR25630398_1.fastq.gz
Approx 75% complete for SRR25630398_1.fastq.gz
Approx 80% complete for SRR25630398_1.fastq.gz
Approx 85% complete for SRR25630398_1.fastq.gz
Approx 90% complete for SRR25630398_1.fastq.gz
Approx 95% complete for SRR25630398_1.fastq.gz
Approx 100% complete for SRR25630398_1.fastq.gz
It seems our guess for the total number of records wasn't very good.  Sorry about that.
Still going at 105% complete for SRR25630398_1.fastq.gz
Still going at 110% complete for SRR25630398_1.fastq.gz
Still going at 115% complete for SRR25630398_1.fastq.gz
Still going at 120% complete for SRR25630398_1.fastq.gz
Still going at 125% complete for SRR25630398_1.fastq.gz
Still going at 130% complete for SRR25630398_1.fastq.gz
Still going at 135% complete for SRR25630398_1.fastq.gz
Still going at 140% complete for SRR25630398_1.fastq.gz
Still going at 145% complete for SRR25630398_1.fastq.gz
Still going at 150% complete for SRR25630398_1.fastq.gz
Still going at 155% complete for SRR25630398_1.fastq.gz
Still going at 160% complete for SRR25630398_1.fastq.gz
Still going at 165% complete for SRR25630398_1.fastq.gz


### 8.31.26- 
Part1_part2 (what is up with this numbering)
5- confirm versions install
```
pixi run cutadapt --version
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
5.2

pixi run trimmomatic -version
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
0.41

```
So is it a problem that my trimmomatic is .41 vs the .40? Hope says no, so moving on.
cutadapt version 5.2 (Python 3.12.14)
trimmomatic version 0.41

For adapters:
I looked at fastqc outputs, only one of my files showed a prediction for adapter content.
File SRR25630303 shows almost no prediction or rise towards the 3' end. However the other file SRR25630398 does show adapter prediction for the Illumina Universal Adapter. I have looked up the sequence for that and will look for it in my files:

```
ALAPAS login2 (/gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs) 09:18 PM $ zcat SRR25630303_1.fastq.gz | head -200 | grep -c "AGATCGGAAGAGC"
1
TALAPAS login2 (/gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs) 09:19 PM $ zcat SRR25630303_1.fastq.gz | head -400 | grep -c "AGATCGGAAGAGC"
1
TALAPAS login2 (/gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs) 09:19 PM $ zcat SRR25630303_1.fastq.gz | head -1000 | grep -c "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA"
1
TALAPAS login2 (/gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs) 09:20 PM $ zcat SRR25630398_1.fastq.gz | head -1000 | grep -c "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA"
4
TALAPAS login2 (/gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs) 09:21 PM $ zcat SRR25630398_1.fastq.gz | head -4000 | grep -c "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA"
24
TALAPAS login2 (/gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs) 09:22 PM $ zcat SRR25630398_1.fastq.gz | head -4000 | grep -c "AGATCGGAAGAGC"
66
```

So somewhat as to be expected, there are more matches when I use a shorter sequence of the adapter. Also from the fastqc reports the 398 file shows more frequent matches than the 303 file when doing just some initital greping. However the presence in both files of the full sequence of the Illumina Universial Adapter even a few times in the start of the file, for me confirms this is the correct adapter. 

#### Cutadapt: 
wrote an sbatch to run the paired end trim/In paired-end mode version of cutadapt on the fastq files from the part1_part1. with default settings so no additional flags except adapter sequences. 
/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project-2-Electric-organ-RNA-seq-analysis/Project2_Part2/me_run_cutadapt.sh

```
#!/bin/bash
#SBATCH --time=01:00:00
#SBATCH --partition=bgmp
#SBATCH --account=bgmp
#SBATCH --job-name=cutadapt_trim
#SBATCH --output=cutadapt_trim_%j.out

cd /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA

# Sample 1: SRR25630303
/usr/bin/time -v pixi run cutadapt \
  -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA \
  -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
  -o /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmed_outputs/SRR25630303_1_trimmed.fastq.gz \
  -p /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmed_outputs/SRR25630303_2_trimmed.fastq.gz \
  /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs/SRR25630303_1.fastq.gz \
  /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs/SRR25630303_2.fastq.gz

# Sample 2: SRR25630398
/usr/bin/time -v pixi run cutadapt \
  -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA \
  -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT \
  -o /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmed_outputs/SRR25630398_1_trimmed.fastq.gz \
  -p /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmed_outputs/SRR25630398_2_trimmed.fastq.gz \
  /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs/SRR25630398_1.fastq.gz \
  /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/SRA_outputs/SRR25630398_2.fastq.gz
```

Output statistics/summary:
**Proportions trimmed:**
SRR25630303:
- Read 1 with adapter: 1,767,108 / 41,934,422 (4.2%)
- Read 2 with adapter: 2,066,337 / 41,934,422 (4.9%)

SRR25630398:
- Read 1 with adapter: 4,844,719 / 34,878,180 (13.9%)
- Read 2 with adapter: 4,978,371 / 34,878,180 (14.3%)

Far far higher adapter contamination/presence in the 398 sample/file compared to the 303 reads. Explains why the grep found so few in 303 compared to 398.

Resource Usage Summary: 
from /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project-2-Electric-organ-RNA-seq-analysis/Project2_Part2/cutadapt_trim_46964814.out
```
SRR25630303:
- Wall time: 6:21.96
- Max memory (RSS): 42,224 KB 
- CPU: 98%
- Exit status: 0

SRR25630398:
- Wall time: 5:50.02
- Max memory (RSS): 38,292 KB 
- CPU: 99%
- Exit status: 0
```

#### Trimmomatic
ran sbatch for both files: /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project-2-Electric-organ-RNA-seq-analysis/Project2_Part2/me_run_trimmomatic.sh 
in paired-end (PE) mode on the Cutadapt-trimmed output files. Applied LEADING:3, TRAILING:3, SLIDINGWINDOW:5:15, MINLEN:35, in that order per assignment. Output produces four files per sample (paired R1, unpaired R1, paired R2, unpaired R2), compressed. Phred encoding left unspecified, Trimmomatic autodetects since v0.32. 

```
#!/bin/bash
#SBATCH --time=01:00:00
#SBATCH --partition=bgmp
#SBATCH --account=bgmp
#SBATCH --job-name=trimmomatic_trim
#SBATCH --output=trimmomatic_trim_%j.out

cd /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA

# Sample 1: SRR25630303
/usr/bin/time -v pixi run trimmomatic PE \
  cutadapt_outputs/SRR25630303_1_trimmed.fastq.gz \
  cutadapt_outputs/SRR25630303_2_trimmed.fastq.gz \
  trimmomatic_outputs/SRR25630303_1_paired.fastq.gz \
  trimmomatic_outputs/SRR25630303_1_unpaired.fastq.gz \
  trimmomatic_outputs/SRR25630303_2_paired.fastq.gz \
  trimmomatic_outputs/SRR25630303_2_unpaired.fastq.gz \
  LEADING:3 TRAILING:3 SLIDINGWINDOW:5:15 MINLEN:35

# Sample 2: SRR25630398
/usr/bin/time -v pixi run trimmomatic PE \
  cutadapt_outputs/SRR25630398_1_trimmed.fastq.gz \
  cutadapt_outputs/SRR25630398_2_trimmed.fastq.gz \
  trimmomatic_outputs/SRR25630398_1_paired.fastq.gz \
  trimmomatic_outputs/SRR25630398_1_unpaired.fastq.gz \
  trimmomatic_outputs/SRR25630398_2_paired.fastq.gz \
  trimmomatic_outputs/SRR25630398_2_unpaired.fastq.gz \
  LEADING:3 TRAILING:3 SLIDINGWINDOW:5:15 MINLEN:35
```

Ok so after I realized I should have set memory and CPUs for these... woops.

Resource Usage Summary: 

==WHAT ARE THE INTERMEDIATE FILES TO CLEAR OUT???==- the cutadapt files after i am sure that t he trimmomatic are done/fine
8. 
plotting- histogram, yes and doing the full 150 possible lengths.
4 paired files total: 303_R1_paired, 303_R2_paired, 398_R1_paired, 398_R2_paired.

Make 2 plots, one per sample? meaning: YES confirmed

**Plot 1 (sample SRR25630303):** 303_R1_paired length distribution + 303_R2_paired length distribution
**Plot 2 (sample SRR25630398):** 398_R1_paired length distribution + 398_R2_paired length distribution

**X-axis:** read length, in bins, or like for every read length from 0-150?
**Y-axis:** count of reads falling in that length/bin.

get information from trimmed files:
```
zcat /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmomatic_outputs/SRR25630303_1_paired.fastq.gz | sed -n '2~4p' | awk '{print length($0)}' | sort -n | uniq -c > 303R1_Pairedlength_dist.txt
zcat /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmomatic_outputs/SRR25630303_2_paired.fastq.gz | sed -n '2~4p' | awk '{print length($0)}' | sort -n | uniq -c > 303R2_Pairedlength_dist.txt

zcat /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmomatic_outputs/SRR25630398_1_paired.fastq.gz | sed -n '2~4p' | awk '{print length($0)}' | sort -n | uniq -c > 398R1_Pairedlength_dist.txt
zcat /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/trimmomatic_outputs/SRR25630398_2_paired.fastq.gz | sed -n '2~4p' | awk '{print length($0)}' | sort -n | uniq -c > 398R2_Pairedlength_dist.txt
```
Now have txt files with the distributions. plot. have to install plotting on talaps for this environment, so could put them in /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA where we were told to put the pixi, and then move the outputs into the Repo for upload.... orrrr? Yes doing that, will move finals. Cannot do multiple sample bar graphs in matplot lib without like setting the distances? ah. transparent (alpha = ) doesn't work to overlay the two its too messy. Pandas? still looks terrible. huh.


## Notes

FOR kegg or GO, (says hope) current info we have in terms of gene names are not real/known in databases, so we will need to convert. take our genes, find orthologs in the csv file hope gives us to match, then use the names from the known orthologs to get names for our unknown named ones, then can search in KEGG or Go.
orthologs from csv file, fish 

### 9.2.26 Part 3
#### installing packages
```
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA) 03:02 PM $ pixi add Star
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
 WARN cache for PypiMapping at /home/hodapp/.cache/rattler/cache/conda-pypi-mapping is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/conda-pypi-mapping for this run. Set [cache.pypi-mapping] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
✔ Added Star >=2.7.11b,<3
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA) 03:02 PM $ pixi add Samtools
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
 WARN cache for PypiMapping at /home/hodapp/.cache/rattler/cache/conda-pypi-mapping is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/conda-pypi-mapping for this run. Set [cache.pypi-mapping] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
✔ Added Samtools >=1.23.1,<2
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA) 03:03 PM $ pixi add NumPy
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
 WARN cache for PypiMapping at /home/hodapp/.cache/rattler/cache/conda-pypi-mapping is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/conda-pypi-mapping for this run. Set [cache.pypi-mapping] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
✔ Added NumPy >=2.5.2,<3
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA) 03:03 PM $ pixi add Matplotlib
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
 WARN cache for PypiMapping at /home/hodapp/.cache/rattler/cache/conda-pypi-mapping is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/conda-pypi-mapping for this run. Set [cache.pypi-mapping] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
✔ matplotlib is already a dependency
  Run `pixi upgrade matplotlib` to get the newest compatible version
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA) 03:03 PM $ pixi add HTSeq
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
 WARN cache for PypiMapping at /home/hodapp/.cache/rattler/cache/conda-pypi-mapping is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/conda-pypi-mapping for this run. Set [cache.pypi-mapping] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
✔ Added HTSeq >=2.1.2,<3
```

and AGAT
```
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/C_compressirostris) 03:26 PM $ pixi add AGAT
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
 WARN cache for PypiMapping at /home/hodapp/.cache/rattler/cache/conda-pypi-mapping is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/conda-pypi-mapping for this run. Set [cache.pypi-mapping] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
✔ Added AGAT >=1.7.0,<2
```

####  Getting source data and prepping
Downloaded source files for _Campylomormyrus compressirostris_ genome fasta and gff from talapas:
`/projects/bgmp/shared/Bi623/Project2/campylomormyrus.fasta` `/projects/bgmp/shared/Bi623/Project2/campylomormyrus.gff`

run AGAT in an sbatch to convert the gff file to gtf as STAR needs a gff file. Ran SBATCH: /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/C_compressirostris/me_run_agat.sh
```
#!/bin/bash
#SBATCH --account=bgmp
#SBATCH --partition=bgmp
#SBATCH --cpus-per-task=8
#SBATCH --mem=16G***NOOOOOO needs to be more! bumped to 64 for 2nd try
#SBATCH --job-name=gff_to_gtf
#SBATCH --output=gff_to_gtf_%j.out
#SBATCH --time=3:00:00


/usr/bin/time -v pixi run agat_convert_sp_gff2gtf.pl --gff campylomormyrus.gff -o campylomormyrus.gtf
```

Usage summary take 2 (successful):
```
command : /gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/.pixi/envs/default/bin/agat_convert_sp_gff2gtf.pl --gff campylomormyrus.gff -o campylomormyrus.gtf
date : 09/02/2026 at 19h51m12s
Job done! Bye Bye!

	Command being timed: "pixi run agat_convert_sp_gff2gtf.pl --gff campylomormyrus.gff -o campylomormyrus.gtf"
	
	Percent of CPU this job got: 99%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 18:47.19
	Maximum resident set size (kbytes): 27959816
	Exit status: 0
```


**AGAT gff2gtf conversion: OOM failure and retry** 
/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Input_data/gff_to_gtf_47035765_errored.out
Exit status 0 is a lie! First attempt (`--mem=16G`) failed: AGATdid a bunch of stuff parsed 100% but then stopped on mering at 72%. killed by SLURM's OOM killer at 72% through the merge step apparently.... peak RSS ~16.7G  job log showed `Exit status: 0`  which makes no sense, but there was nothing actually in the GTF that could be used in STAR. Resulting GTF was truncated/unusable (0 exon lines when fed to STAR). Deleted the partial GTF and resubmitted with `--mem=64G`.


#### Alignments (STAR)
1. plug the fasta and gtf (converted above by agat) into star to create a database
2. Then align my trimmed paired reads from trimmomatic output TO the database of _Campylomormyrus compressirostris_  using star. 
3. The output will be a SAM file
4. Then count the mapped and unmapped reads (using my script from 621_PS8)

##### Create Database
pixi run STAR --version
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
2.7.11b

ok Created a database directory for STAR database:
```
/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Campylomormyrus_compressirostris.dryad_c59zw3rcj.STAR_2.7.11b
```

and ran sbatch: /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Campylomormyrus_compressirostris.dryad_c59zw3rcj.STAR_2.7.11b/me_run_STAR_Database.sh

FAILED:
```
#!/bin/bash

#SBATCH --account=bgmp
#SBATCH --partition=bgmp
#SBATCH --cpus-per-task=8
#SBATCH --mem=16G
#SBATCH --job-name=star_database
#SBATCH --output=star_database_%j.out
#SBATCH --time=3:00:00

# OUTPUT: the (currently empty) folder STAR will fill with the index
GENOME_DIR=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Campylomormyrus_compressirostris.dryad_c59zw3rcj.STAR_2.7.11b
# INPUT 1: the genome sequence itself
FASTA=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Input_data/campylomormyrus.fasta
# INPUT 2: the annotation (where genes/exons/introns sit on that sequence = map) 
GTF=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Input_data/campylomormyrus.gtf


# THE OPERATION: read the two inputs, build the index/database, write it to the output 
/usr/bin/time -v pixi run STAR \
    --runThreadN 8 \
    --runMode genomeGenerate \
    --genomeDir $GENOME_DIR \
    --genomeFastaFiles $FASTA \
    --sjdbGTFfile $GTF
    # use 8 threads, --runMode genomeGenerate = "build an index", output dir location, input1, input2
```

FAILED: memory needs bumped and also warning flag on `--genomeSAindexNbases` so setting that to be 13 bumping mem to 64 and try to rerun. Second genomeGenerate attempt now have valid GTF, but) (`--mem=16G`) again OOM-killed. Additionally, STAR emitted a warning during this run: `--genomeSAindexNbases 14` (default) is oversized for this genome (862,592,683 bp) and could cause a seg-fault at the mapping step; recommended value is 13. 

```
#!/bin/bash

#SBATCH --account=bgmp
#SBATCH --partition=bgmp
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --job-name=star_database
#SBATCH --output=star_database_%j.out
#SBATCH --time=3:00:00

# OUTPUT: the (currently empty) folder STAR will fill with the index
GENOME_DIR=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Campylomormyrus_compressirostris.dryad_c59zw3rcj.STAR_2.7.11b
# INPUT 1: the genome sequence itself
FASTA=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Input_data/campylomormyrus.fasta
# INPUT 2: the annotation (where genes/exons/introns sit on that sequence = map) 
GTF=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Input_data/campylomormyrus.gtf


# THE OPERATION: read the two inputs, build the index/database, write it to the output 
/usr/bin/time -v pixi run STAR \
    --runThreadN 8 \
    --runMode genomeGenerate \
    --genomeDir $GENOME_DIR \
    --genomeFastaFiles $FASTA \
    --sjdbGTFfile $GTF \
    --genomeSAindexNbases 13 #added this after the first run memory failed, the slurm out contained an error and recomendation
    # use 8 threads, --runMode genomeGenerate = "build an index", output dir location, input1, input2

```

worked!
Usage Summary take 3:
```
Command being timed: "pixi run STAR --runThreadN 8 --runMode genomeGenerate --genomeDir /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Campylomormyrus_compressirostris.dryad_c59zw3rcj.STAR_2.7.11b --genomeFastaFiles /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Input_data/campylomormyrus.fasta --sjdbGTFfile /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Input_data/campylomormyrus.gtf --genomeSAindexNbases 13"
	Percent of CPU this job got: 430%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 4:28.45
	Maximum resident set size (kbytes): 26207536
	Exit status: 0
```

Notes/Questions/Wonderings:
**--sjdbOverhang consideration**

STAR's `--sjdbOverhang` parameter sets the length of genomic sequence flanking each annotated splice junction, used to build the splice junction database from the GTF. Formula: max(read length) - 1. Default value if unspecified: 100.

Trimmed read lengths for this dataset are variable as MINLENGTH floor of 35 (from Trimmomatic settings), majority of reads around 150bp. Textbook value based on max read length would be 149. Assignment instructions (mirrored from PS8/Bi621) specify five required parameters for genomeGenerate (`--runThreadN`, `--runMode`, `--genomeDir`, `--genomeFastaFiles`, `--sjdbGTFfile`); `--sjdbOverhang` is absent from that list. ==How much of a difference would this make, and should i adjust? ==NO its fine

**STAR genomeGenerate- `--genomeSAindexNbases`**

Genome: campylomormyrus.fasta, 823M, 1497 contigs(?) (checked via `ls -lh` and `grep -c ">"`).

Note on `--genomeSAindexNbases`: STAR default is 14, tuned for large genomes. For genomes under ~1–2 Gb, especially with reduced contiguity (1497 contigs is obviously not chromosome-level), the STAR manual recommends scaling this down using min(14, log2(GenomeLength)/2 − 1). For this genome size, the calculated value lands close to 14, so a warning in the log is not expected but should be checked for on job completion, and if thrown, would need to be adjusted down. ADJUSTED DOWN TO 13

##### Alignments (trimmed paired reads against database created above)

Created sub folders for each file in /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Star_alignments

so 
Star_alignments/SRR25630303
Star_alignments/SRR25630398

Ran Sbatch for each file/sample:  
- /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Star_alignments/SRR303_align.sh
- /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Star_alignments/SRR398_align.sh

```
#!/bin/bash

#SBATCH --account=bgmp
#SBATCH --partition=bgmp
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --job-name=star_align_SRR25630398
#SBATCH --output=star_align_SRR25630398_%j.out
#SBATCH --time=6:00:00

# INPUT 1: the reads RNA (shared class data, referenced not copied)
READ1=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/4_trimmomatic_outputs/SRR25630398_1_paired.fastq.gz
READ2=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/4_trimmomatic_outputs/SRR25630398_2_paired.fastq.gz
# INPUT 2: the genome index built yesterday (output then, input now)
GENOME_DIR=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Campylomormyrus_compressirostris.dryad_c59zw3rcj.STAR_2.7.11b
# OUTPUT: prefix STAR sticks on the front of every file it writes. 
OUT_PREFIX=SRR25630398_

cd /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Star_alignments/SRR25630398

# THE OPERATION: align the reads to the index, write SAM file w/ perameters set to match 621_PS8 
/usr/bin/time -v pixi run STAR \
    --runThreadN 8 \
    --runMode alignReads \
    --outFilterMultimapNmax 3 \
    --outSAMunmapped Within KeepPairs \
    --alignIntronMax 1000000 --alignMatesGapMax 1000000 \
    --readFilesCommand zcat \
    --readFilesIn $READ1 $READ2 \
    --genomeDir $GENOME_DIR \
    --outFileNamePrefix $OUT_PREFIX

```

```
#!/bin/bash

#SBATCH --account=bgmp
#SBATCH --partition=bgmp
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --job-name=star_align_SRR25630303
#SBATCH --output=star_align_SRR25630303_%j.out
#SBATCH --time=6:00:00

# INPUT 1: the reads RNA (shared class data, referenced not copied)
READ1=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/4_trimmomatic_outputs/SRR25630303_1_paired.fastq.gz
READ2=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/4_trimmomatic_outputs/SRR25630303_2_paired.fastq.gz
# INPUT 2: the genome index built yesterday (output then, input now)
GENOME_DIR=/projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Campylomormyrus_compressirostris.dryad_c59zw3rcj.STAR_2.7.11b
# OUTPUT: prefix STAR sticks on the front of every file it writes. 
OUT_PREFIX=SRR25630303_

cd /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris/Star_alignments/SRR25630303

# THE OPERATION: align the reads to the index, write SAM file w/ perameters set to match 621_PS8 
/usr/bin/time -v pixi run STAR \
    --runThreadN 8 \
    --runMode alignReads \
    --outFilterMultimapNmax 3 \
    --outSAMunmapped Within KeepPairs \
    --alignIntronMax 1000000 --alignMatesGapMax 1000000 \
    --readFilesCommand zcat \
    --readFilesIn $READ1 $READ2 \
    --genomeDir $GENOME_DIR \
    --outFileNamePrefix $OUT_PREFIX

```

Usage Summary:

#### Counting - mapped and unmapped reads in SAM (from alignments above)
Use 621_PS8 script for this 621_count_mappedreads.py copied into the Bi623/PR2/Project2_QAA for use...


changed my PS8 script to take argparse to accept a file rather than hardcoded. ==I just had it print is that sufficient???? ==
```
python 621_count_mappedreads.py -f /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris_sample.alignments/Star_alignments/SRR25630303/SRR25630303_Aligned.out.sam
File: /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris_sample.alignments/Star_alignments/SRR25630303/SRR25630303_Aligned.out.sam
Mapped reads: 79621037
Unmapped reads: 3651623
```

```
python 621_count_mappedreads.py -f /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris_sample.alignments/Star_alignments/SRR25630398/SRR25630398_Aligned.out.sam
File: /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris_sample.alignments/Star_alignments/SRR25630398/SRR25630398_Aligned.out.sam
Mapped reads: 64341367
Unmapped reads: 4777469
```


Count reads that map to features ==what does this mean???==
htseq-count


RUN the sbatch for the 4 scripts in /projects/bgmp/hodapp/bioinfo/Bi623/PR2/Project2_QAA/6_C_compressirostris_sample.alignments/4_htseq_counts

ICA4...also where the %

for question 15-
SRR25630303_htseqcounts_forwardstranded
total: 41636330
```
awk '{sum+=$2} END {print sum}' SRR25630303_htseqcounts_forwardstranded.txt 
41636330
```

```
grep -v '^__' SRR25630303_htseqcounts_forwardstranded.txt | awk '{sum+=$2} END {print sum}'
1368032
```
mapped total: 1368032

SRR25630303_htseqcounts_reversestranded: 
total: 41636330
```
awk '{sum+=$2} END {print sum}' SRR25630303_htseqcounts_reversestranded.txt 
41636330
```

```
PM $ grep -v '^__' SRR25630303_htseqcounts_reversestranded.txt | awk '{sum+=$2} END {print sum}'
24841228
```
mapped total: 24841228
Crh_rhy50_EO_6cm_1_htseqcounts_[forORrev]stranded.txt FOR SRR25630303


SRR25630398_htseqcounts_forwardstranded
```
awk '{sum+=$2} END {print sum}' SRR25630398_htseqcounts_forwardstranded.txt
34559418
```
total: 34559418

CcoxCts_rhy107_EO_adult_2_htseqcounts_[forORrev]stranded.txt FOR SRR25630398

```
grep -v '^__' SRR25630398_htseqcounts_forwardstranded.txt | awk '{sum+=$2} END {print sum}'
1001555
```
mapped: 1001555
% mapped: (1001555/34559418)=

SRR25630398_htseqcounts_reversestranded
```
awk '{sum+=$2} END {print sum}' SRR25630398_htseqcounts_reversestranded.txt 
34559418
```
total: 34559418

```
grep -v '^__' SRR25630398_htseqcounts_reversestranded.txt | awk '{sum+=$2} END {print sum}'
18653979
```
mapped: 18653979



Factor, the totals of mapped reads. 
answer 15 in answers md. 
move files,
rename FINAL HTCseq files 
make sure to put the name of my repo for my lab notebook SOMEPLACE in a file before uploading to git...
upload to git
update lab notebook
