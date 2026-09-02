# PS7 Lab Notebook - RBH alignments, human & zebrafish

## Overview

**Concept:** Build the front end of a project to do reciprocal best hit (RBH) hits between human and zebrafish proteins. Download proteomes from Ensemble, filter each down to the longest protein per gene, then blastp each species against the other. Actual RBH calling happens later in Bi623; this is the setup.

**Start date:** 7.8.2026

---

## Pipeline / approach

1. Download human and zebrafish protein FASTA + Biomart gene tables from Ensembl (release 116) onto Talapas.
2. Write `longest_protein.py` to keep only the longest protein record per gene, with clean headers.
3. Build a blast database for each species from its longest-protein file.
4. blastp each species' longest proteins against the other species' database.
5. Summarize hits with Bash (counts, top bitscores, lowest-evalue hits into `HLE.txt` / `ZLE.txt`).
6. TBD in Bi623...ooooo

---

## Environment

- Blast 2.17.0, installed via pixi into the repo on talapas
- Python 3.14 (pixi).
- Full environment details in pixi.toml / pixi.lock. found at
	- https://github.com/2026-BGMP/palimpsestkenlyn-Bi621-PS7/tree/main
	- https://github.com/2026-BGMP/palimpsestkenlyn-Bi621-PS7/pixi.toml - commit 7f762e3b751d9e7245640ab21c429310660af894
	- https://github.com/2026-BGMP/palimpsestkenlyn-Bi621-PS7/pixi.lock - commit 7f762e3b751d9e7245640ab21c429310660af894

---

## Reference data / location map

- Working dir: `/projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/`
	- And possibly then `/projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/original_run/`
	- nevermind, this never happened there is no original run.
- Input proteomes: `Homo_sapiens.GRCh38.pep.all.fa`, `Danio_rerio.GRCz11.pep.all.fa`
	- from https://www.ensembl.org/index.html version 116, soon to be replaced, archieved https://www.ensembl.org/index.html
- Biomart tables: `human_genes.txt`, `zebrafish_genes.txt` also from ensembl.
- Longest-protein outputs: `human_longest.fa` (23879 records), `zebrafish_longest.fa` (30313 records)


---

# Log: chronological

### 07.08.2026 - setup + files onto Talapas

**ran:**
- Cloned my git repo to Talapas.
- Setup and file download as outlined in PS7.md Part 1, steps 1–2.

**notes:**
- Downloaded proteome FASTAs and Biomart tables for both species. Ready to start the Python.

---

### 07.15.2026 - writing longest_protein scripts. Going quick, made one for human one for zebra after testing the first. just hard coded.

**ran:**
- Wrote `longest_proteinH.py` and `longest_proteinZ.py` Approach: build a lookup dict from the Biomart table (protein ID to get gene ID + gene name) to serve as dictionary for lookup since header data was messy and missing fileds, collapse the multi-line FASTA to one sequence line per record since currently many lines, then keep the longest protein per GENE. So see the same gene id lots, but only want it in final file once for the one with the longest protein id associated. 
- /projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/original_run/longest_proteinH.py
- /projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/original_run/longest_proteinZ.py
- Tested the oneline-FASTA step:

```
grep -c "^>" human_oneline.fa; grep -c "^>" Homo_sapiens.GRCh38.pep.all.fa
382428
382428
```

**got:**
- Header counts match between the original and the collapsed file, so no records added or lost during collapsing.

```
wc -l human_oneline.fa
764856 human_oneline.fa
```

- 764856 = 2 × 382428, i.e. exactly two lines per entry (header + sequence). Collapsing worked.

**notes:**
- Final longest-protein counts came out to the expected 23879 (human) and 30313 (zebrafish).

---

### 07.16.2026 - Part 2, build databases

**ran:**
- Built blast databases as a batch job so I'd have a script record.

```
sbatch create_databases.sh
```

- Job ID 45500680.

**on:**
- `human_longest.fa`
- `zebrafish_longest.fa`

**got:**
- `human_db.*` (various index files)
- `zebrafish_db.*` (various index files)

**cost/usage:** from `cat slurm-45500680.out`

Human DB: 23879 sequences added.
- Command: `pixi run makeblastdb -in human_longest.fa -dbtype prot -out human_db`
- Wall clock: 0:04.04
- Max resident set size: 34580 KB
- Exit status: 0

Zebrafish DB: 30313 sequences added.
- Command: `pixi run makeblastdb -in zebrafish_longest.fa -dbtype prot -out zebrafish_db`
- Wall clock: 0:00.76
- Max resident set size: 34840 KB
- Exit status: 0

```
Building a new DB, current time: 07/16/2026 10:59:08

New DB name: /gpfs/projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/human_db

New DB title: human_longest.fa

Sequence type: Protein

Keep MBits: T

Maximum file size: 3000000000B

Adding sequences from FASTA; added 23879 sequences in 0.470021 seconds.

Command being timed: "pixi run makeblastdb -in human_longest.fa -dbtype prot -out human_db"

User time (seconds): 0.42

System time (seconds): 0.14

Percent of CPU this job got: 14%

Elapsed (wall clock) time (h:mm:ss or m:ss): 0:04.04

Average shared text size (kbytes): 0

Average unshared data size (kbytes): 0

Average stack size (kbytes): 0

Average total size (kbytes): 0

Maximum resident set size (kbytes): 34580

Average resident set size (kbytes): 0

Major (requiring I/O) page faults: 0

Minor (reclaiming a frame) page faults: 14253

Voluntary context switches: 539

Involuntary context switches: 6

Swaps: 0

File system inputs: 0

File system outputs: 24

Socket messages sent: 0

Socket messages received: 0

Signals delivered: 0

Page size (bytes): 4096

Exit status: 0

WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.

Building a new DB, current time: 07/16/2026 10:59:09

New DB name: /gpfs/projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/zebrafish_db

New DB title: zebrafish_longest.fa

Sequence type: Protein

Keep MBits: T

Maximum file size: 3000000000B

Adding sequences from FASTA; added 30313 sequences in 0.531164 seconds.

Command being timed: "pixi run makeblastdb -in zebrafish_longest.fa -dbtype prot -out zebrafish_db"

User time (seconds): 0.52

System time (seconds): 0.09

Percent of CPU this job got: 80%

Elapsed (wall clock) time (h:mm:ss or m:ss): 0:00.76

Average shared text size (kbytes): 0

Average unshared data size (kbytes): 0

Average stack size (kbytes): 0

Average total size (kbytes): 0

Maximum resident set size (kbytes): 34840

Average resident set size (kbytes): 0

Major (requiring I/O) page faults: 0

Minor (reclaiming a frame) page faults: 14341

Voluntary context switches: 200

Involuntary context switches: 7

Swaps: 0

File system inputs: 0

File system outputs: 24

Socket messages sent: 0

Socket messages received: 0

Signals delivered: 0

Page size (bytes): 4096

Exit status: 0
```

---

### 07.16.2026 - Part 2, run blastp

**ran:**
- Ran `blastp.sh` for both directions (human→zebrafish and zebrafish→human), 8 threads, `-evalue 1e-6`, `-use_sw_tback`, `-outfmt 6`.

```
sbatch blastp.sh
```

- Submitted batch job 45504816.

**on:**
- `human_longest.fa` (query) vs `zebrafish_db`
- `zebrafish_longest.fa` (query) vs `human_db`

**got:**
- `human_vs_zebrafish.tsv`
- `zebrafish_vs_human.tsv`

**cost/usage:** from `cat slurm-45504816.out`

human vs zebrafish:
- Command: `pixi run blastp -query human_longest.fa -db zebrafish_db -out human_vs_zebrafish.tsv -evalue 1e-6 -use_sw_tback -num_threads 8 -outfmt 6`
- User time: 16297.41 s; CPU: 641%; wall clock: 42:26.46
- Max resident set size: 630828 KB
- Exit status: 0

zebrafish vs human:
- Command: `pixi run blastp -query zebrafish_longest.fa -db human_db -out zebrafish_vs_human.tsv -evalue 1e-6 -use_sw_tback -num_threads 8 -outfmt 6`
- User time: 16654.84 s; CPU: 784%; wall clock: 35:27.57
- Max resident set size: 726008 KB
- Exit status: 0
```
WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.

Command being timed: "pixi run blastp -query human_longest.fa -db zebrafish_db -out human_vs_zebrafish.tsv -evalue 1e-6 -use_sw_tback -num_threads 8 -outfmt 6"

User time (seconds): 16297.41

System time (seconds): 31.01

Percent of CPU this job got: 641%

Elapsed (wall clock) time (h:mm:ss or m:ss): 42:26.46

Average shared text size (kbytes): 0

Average unshared data size (kbytes): 0

Average stack size (kbytes): 0

Average total size (kbytes): 0

Maximum resident set size (kbytes): 630828

Average resident set size (kbytes): 0

Major (requiring I/O) page faults: 0

Minor (reclaiming a frame) page faults: 23296178

Voluntary context switches: 27800

Involuntary context switches: 38845

Swaps: 0

File system inputs: 0

File system outputs: 8

Socket messages sent: 0

Socket messages received: 0

Signals delivered: 0

Page size (bytes): 4096

Exit status: 0

WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.

Command being timed: "pixi run blastp -query zebrafish_longest.fa -db human_db -out zebrafish_vs_human.tsv -evalue 1e-6 -use_sw_tback -num_threads 8 -outfmt 6"

User time (seconds): 16654.84

System time (seconds): 43.83

Percent of CPU this job got: 784%

Elapsed (wall clock) time (h:mm:ss or m:ss): 35:27.57

Average shared text size (kbytes): 0

Average unshared data size (kbytes): 0

Average stack size (kbytes): 0

Average total size (kbytes): 0

Maximum resident set size (kbytes): 726008

Average resident set size (kbytes): 0

Major (requiring I/O) page faults: 0

Minor (reclaiming a frame) page faults: 30118326

Voluntary context switches: 32215

Involuntary context switches: 47269

Swaps: 0

File system inputs: 0

File system outputs: 8

Socket messages sent: 0

Socket messages received: 0

Signals delivered: 0

Page size (bytes): 4096

Exit status: 0
```

---

### 07.16.2026 - Leslie flagged an issue with longest protein, re-ran original scripts, same output, rewrote a new one? with argparse to see if file tracking flow was issue... = longest_protein.py. 

**note on what happened:**
- Leslie ran her checker on my longest-protein files and reported a problem. I moved everything from this run into `original_run/` as a backup and re-verified from scratch.
- Checked the actual file contents: `human_longest.fa` contains human records (`>ENSP...`) and `zebrafish_longest.fa` contains zebrafish records (`>ENSDARP...`), both with correct counts and clean headers. The data was correct in every way I checked, redownloaded base files from ensemble, rechecked counts, looked at formatting, grep for header lines, etc etc. Cannot figure it out. Long story short, no issue. 
	-  Test swapped the two files (the HUMAN section was reading my zebrafish file). Flagged this back to Leslie to confirm. 
	- Decision, delete all other work (except longest_protein.py its better) and move everything OUT of the original_run/ back up to the base level and just delete original_run directory.
- I rewrote `longest_protein.py` to take `-t`, `-f`, `-o` via argparse so one script handles both species, and re-ran to confirm. All old stuff now in original_run

**ran + got:**

```
python longest_protein.py -t human_genes.txt -f Homo_sapiens.GRCh38.pep.all.fa -o human_longest.fa
23879

python longest_protein.py -t zebrafish_genes.txt -f Danio_rerio.GRCz11.pep.all.fa -o zebrafish_longest.fa
30313
```

- Both counts match the expected values. Sanity-checked headers and counts: ALL LOOKS WELL

```
grep -c "^>" human_longest.fa       -> 23879
grep -c "^>" zebrafish_longest.fa   -> 30313
head human_longest.fa               -> >ENSP... ENSG... GENE_NAME
head zebrafish_longest.fa           -> >ENSDARP... ENSDARG... GENE_NAME
```

- Blast outputs in `original_run/` are still valid, since the input files were correct all along. No need to re-blast. Leslie confirmed both old and new files were fine. yay. 

---

### 07.16.2026 - Part 2, Bash summaries

Worked with `.tsv` results files. Column 11 = evalue, column 12 = bitscore. Since ran with default setting of 6. 

#### Number of hits

```
wc -l human_vs_zebrafish.tsv
1986147 human_vs_zebrafish.tsv

wc -l zebrafish_vs_human.tsv
2556891 zebrafish_vs_human.tsv
```

#### 10 highest bitscores - human vs zebrafish

```
sort -k12 -rn human_vs_zebrafish.tsv | head -10
ENSP00000467141 ENSDARP00000107866  61.421  23829  8544  111  12403  35991  5733   29152  0.0  30359
ENSP00000467141 ENSDARP00000099532  67.302  20671  6694  36   13626  34253  5617   26265  0.0  29867
ENSP00000356224 ENSDARP00000124184  57.149  8812   3659  25   1      8797   1      8710   0.0  9902
ENSP00000364403 ENSDARP00000128666  86.119  5194   667   16   20     5183   3      5172   0.0  8997
ENSP00000364403 ENSDARP00000152557  86.102  5195   667   17   20     5183   3      5173   0.0  8992
ENSP00000508153 ENSDARP00000111741  84.418  4749   658   24   32     4772   31     4705   0.0  8062
ENSP00000261609 ENSDARP00000098322  82.457  4851   813   16   1      4834   1      4830   0.0  8047
ENSP00000271588 ENSDARP00000005919  66.347  5634   1858  14   11     5635   12     5616   0.0  7940
ENSP00000356224 ENSDARP00000118057  48.910  8857   4266  54   14     8797   27     8697   0.0  7929
ENSP00000508153 ENSDARP00000028785  83.038  4746   598   19   32     4772   31     4574   0.0  7890
```

#### 10 highest bitscores - zebrafish vs human

```
sort -k12 -rn zebrafish_vs_human.tsv | head -10
ENSDARP00000107866  ENSP00000467141  63.604  22461  7785  50   6997   29152  13616  35991  0.0  30322
ENSDARP00000099532  ENSP00000467141  67.315  20673  6688  37   5617   26265  13626  34253  0.0  29819
ENSDARP00000124184  ENSP00000356224  57.149  8812   3659  25   1      8710   1      8797   0.0  9893
ENSDARP00000128666  ENSP00000364403  85.943  5193   670   17   7      5172   24     5183   0.0  9017
ENSDARP00000152557  ENSP00000364403  85.926  5194   670   18   7      5173   24     5183   0.0  9010
ENSDARP00000098322  ENSP00000261609  82.543  4852   807   17   1      4830   1      4834   0.0  8118
ENSDARP00000111741  ENSP00000508153  84.033  4785   670   26   1      4705   2      4772   0.0  8035
ENSDARP00000118057  ENSP00000356224  48.967  8857   4261  53   27     8697   14     8797   0.0  7990
ENSDARP00000005919  ENSP00000271588  66.347  5634   1858  14   12     5616   11     5635   0.0  7935
ENSDARP00000028785  ENSP00000508153  82.643  4782   611   21   1      4574   2      4772   0.0  7854
```

#### Lowest-evalue hits, sorted by bitscore in HLE.txt / ZLE.txt

Lowest evalue in both files is `0.0`. Kept only `0.0`-evalue rows, sorted by bitscore high to low.

```
awk '$11 == "0.0"' human_vs_zebrafish.tsv | sort -k12 -rn > HLE.txt
awk '$11 == "0.0"' zebrafish_vs_human.tsv | sort -k12 -rn > ZLE.txt

wc -l HLE.txt
28383 HLE.txt

wc -l ZLE.txt
27995 ZLE.txt
```

Both files check out: column 11 all `0.0`, column 12 descending from the top.

**note (best bitscore vs best evalue):** the lowest-evalue hits are all at `0.0` (evalues too small to represent), so evalue alone can't rank the top hits, huge numbers of hits tie at `0.0`. Bitscore still spreads them out some to distinguish (30359 down to the thousands), so bitscore is the finer ranking among the strongest hits here.

---

## Concepts 

**Reciprocal best hit (RBH).** The idea is that if human protein A's best match in zebrafish is protein B, AND zebrafish protein B's best match back in human is protein A, then A and B are probably orthologs (same gene, diverged by speciation). It has to go both directions, that's the "reciprocal" part. This PS only builds the inputs, the actual RBH calling is Bi623.

**Why longest protein per gene.** One gene can produce many protein isoforms, so the raw pep.all file has tons of entries per gene. I am seeing the same gene ID many times, with different protein IDs. If I blasted all of them I'd get a mess of similar or duplicate hits and couldn't cleanly say "gene A's best match is gene B." Collapsing to one representative protein per gene (the longest) makes the comparison faster, easier, and by using the longest gene I have the most information AKA the longest sequence to probe matches. 

**Why build a blast database.** `makeblastdb` turns a FASTA into an indexed structure blast can search fast, instead of scanning the raw file every time. `-dbtype prot` because these are proteins (would be `nucl` for DNA).

**blastp direction matters.** `-query` is what I'm searching WITH, `-db` is what I'm searching AGAINST. Human query vs zebrafish db is a different question than zebrafish query vs human db, which seems counter intuitive but is different. Raw scores and aligments can differen depending on what is the query vs database. This can be seen comparing bit scores between the same matches run in opposite directions (meaning switching what is the query vs database).

**outfmt 6.** flag for blastp means in the output. Column 11 = evalue, column 12 = bitscore. That's why my awk/sort commands use `$11` and `$12`.



---
## Final

Fill out answers.md and upload everything to github. The end for now. 
---


# Part 2: RBH in 623: started 8.27.26

Environment: Python 3.14.6
## Log: chronological

### 8.27.26
Through discussion with others, apparently the cases that the script needs to handle from the data:
- Clean 1:1 reciprocal pair, normal 
- Tie at the top evalue  for the hit, (query excluded or dropped as there is a tie no clear winner)
- Same subject repeated at same evalue, not a real tie so should not be dropped
- Query with no match in the reverse file, drop (confirm the reciprocal check doesn't crash in code)
- Query where forward best hit exists but isn't reciprocated (one-directional only, correctly dropped) so basically not a 1:1 hit
- Missing gene ID in biomart (blank field handled) for both files
- Missing gene name in biomart (blank field handled) for both files
- Evalue not at the extreme (e.g. `0.0`) a normal non-zero best hit, and handles numbers as scientific notation correctly.

### 8.30.26
Initial exploration and setup. Decided to use files from projects rather than my outputs from PS7 since will need new files for new species anyways. Cut down on issues. 
INPUTS:
```
Human-Zebrafish:
/projects/bgmp/shared/Bi623/PS1/blasthits/Dre_query_Hsa_db.txt
/projects/bgmp/shared/Bi623/PS1/blasthits/Hsa_query_Dre_db.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Dre_biomart_v116.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Hsa_biomart_v116.txt

Human-ElectricEel:
/projects/bgmp/shared/Bi623/PS1/blasthits/Hsa_query_Eel_db.txt
/projects/bgmp/shared/Bi623/PS1/blasthits/Eel_query_Hsa_db.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Hsa_biomart_v116.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Eel_biomart_v116.txt

Human-ElectricBabyWhale:
/projects/bgmp/shared/Bi623/PS1/blasthits/Hsa_query_Pka_db.txt
/projects/bgmp/shared/Bi623/PS1/blasthits/Pka_query_Hsa_db.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Hsa_biomart_v116.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Pka_biomart_v116.txt

Zebrafish-ElectricEel:
/projects/bgmp/shared/Bi623/PS1/blasthits/Dre_query_Eel_db.txt
/projects/bgmp/shared/Bi623/PS1/blasthits/Eel_query_Dre_db.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Dre_biomart_v116.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Eel_biomart_v116.txt

Zebrafish-ElectricBabyWhale:
/projects/bgmp/shared/Bi623/PS1/blasthits/Dre_query_Pka_db.txt
/projects/bgmp/shared/Bi623/PS1/blasthits/Pka_query_Dre_db.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Dre_biomart_v116.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Pka_biomart_v116.txt

ElectricEel-ElectricBabyWhale:
/projects/bgmp/shared/Bi623/PS1/blasthits/Eel_query_Pka_db.txt
/projects/bgmp/shared/Bi623/PS1/blasthits/Pka_query_Eel_db.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Eel_biomart_v116.txt
/projects/bgmp/shared/Bi623/PS1/biomart/Pka_biomart_v116.txt
```

copy the 12 blastp .txt files into my own directory. Decided to just stay in the 621-PS7 repo and folder on talapas. made a new directory called 623_RBH to work in
sorted the files and created new output sorted files for my own sanity. baby steps:

```
sort -k1,1 -k11,11g Dre_query_Hsa_db.txt > Dre_query_Hsa_sorted.txt
```

- `-k1,1` sorts by column 1 (qseqid) alphabetically, so all IDs like `ENSDARP00000128236` lines land together, so can see each query ID grouped. THEN...
- `-k11,11g` then sorts within each query group by column 11 (evalue), using `-g` (general numeric) instead of plain numeric, because e-values are in scientific notation like `1.00e-103` and plain sort would read that wrong. normal numbers would be too easy.
- Result: for each query, line 1 of its group is the best hit as in highest e-value not yet handling ties and such.

So now need to write a script to read through and keep track of the lowest hit so first evalue since sorted, store in dict for fast lookup, then handle is ties. Use argparse so I can use this on all the files. Going to set it as pairs only, so species 1 vs 2 and run it for each combo rather than mix together, too confusing for me. 

Pseudocode:
```
RECIPROCAL BEST HITS (RBH): PSEUDOCODE

INPUTS:

forward direction: blastp output file, species1 as query vs species2 as database
reverse direction: blastp output file, species2 as query vs species1 as database
biomart file for species1 (protein ID to gene ID, gene name)
biomart file for species2 (protein ID to gene ID, gene name)


STEP 0: SORT (bash, before Python)

For each blastp file:

sort by column 1 (query ID), then by column 11 (evalue, ascending)
- this groups every query's hits together, best evalue on top within each group (group of IDs which is what will be evaluated for ties, to find best/lowest e-value)

  

STEP 1: FIND BEST HIT PER QUERY (run once per direction, forward and reverse)

open sorted file
group consecutive lines by query ID 

for each query's group of lines:
	top evalue = evalue of first line in the group
	collect every DISTINCT subject ID that shares that top evalue
	if more than one distinct subject shares the top evalue:
		this is a real tie = discard this query, no best hit recorded
	else:
		record query = subject as this query's best hit
	return dictionary of query = best hit subject

  

STEP 2: FIND RECIPROCAL PAIRS

for each query in species1's best-hit dictionary:
	subject = species1's best hit for this query
		if subject exists as a key in species2's best-hit dictionary and
		if species2's best hit for that subject equals the original query:
			= reciprocal, keep the pair
		else:
			skip, (subject was never a query in the reverse file, or was excluded there due to its own tie)

  

STEP 3: BUILD GENE LOOKUP TABLES

for each biomart file:
	skip header line
	for each remaining line:
	protein ID, gene ID, gene name = columns
		if protein ID is blank, skip (no usable key)
		gene name may be blank = store empty string, not an error, to fill in spot in output
	store protein ID = (gene ID, gene name)

  

STEP 4: WRITE OUTPUT

write header row

for each reciprocal pair (protein1, protein2):
	look up protein1 in species1's gene lookup = gene ID, gene name
	look up protein2 in species2's gene lookup = gene ID, gene name
	write one row:
		species1 gene ID, protein1, species1 gene name, species2 gene ID, protein2, species2 gene name

```

Writing this as a loop is long and weird and hard to follow. Looking at `itertools.groupby(fh, key=...)` This  returns a sequence of pairs in a tuple. For each group it finds, it hands back the two things bundled together: (the shared value that group has in common, all the lines belonging to that group). so handy. so if this works the way I want it to it would give:
```
group 1's pair: ("ENSDARP00000000004", <the 4 lines starting with that ID>)
group 2's pair: ("ENSDARP00000000005", <the lines starting with that ID>)
```
Then I could evaluate each like group/ID as a unit and figure out ties.

So a few steps of this to think through. Likely write out code, then maybe do functions since will need each one at least twice (for each file in the species pair).

def get_best_hits(filename: str) -> dict[str, str]:
	evaluate and find the best evalue for each queryID looking at ties and storing the pairs for best hits from that file. will be used next in RBH comparison. 

def get_biomart_lookup(filename: str) -> dict[str, tuple[str, str]]:
	Read a biomart file and return protein_id: (gene_id, gene_name)

def find_reciprocal_hits(hits1: dict[str, str], hits2: dict[str, str]) -> list[tuple[str, str]]:
	Compare best-hit dicts from both directions and return pairs where the relationship is reciprocal

```python
def get_best_hits(filename: str) -> dict[str, str]:
    best_hits = {}
    with open(filename, "r") as fh:
        for query, lines in itertools.groupby(fh, key=get_query_id):
            lines = list(lines)
            columns_per_line = []
            for line in lines:
                columns_per_line.append(line.strip("\n").split("\t"))

            best_evalue = columns_per_line[0][10]
            best_subject = columns_per_line[0][1]

            if len(columns_per_line) > 1:
                second_evalue = columns_per_line[1][10]
                is_tied = (second_evalue == best_evalue)
            else:
                is_tied = False

            if not is_tied:
                best_hits[query] = best_subject
    return best_hits
```

1. `best_hits = {}`  start with an empty dictionary, this is what the function will hand back at the end, gets filled throughout.
2. `with open(filename, "r") as fh:`  open the file to read through it.
3. `for query, lines in itertools.groupby(fh, key=get_query_id):` go through the file grouped by query ID. Each time through this loop, `query` = one query ID (e.g. `"ENSDARP00000000004"`), and `lines` = all the raw lines belonging to it, bundled together, still unsplit.
4. `lines = list(lines)`: convert that one-time-use bundle into a real list, so it can be read from more than once.
5. `columns_per_line = []` then the small `for line in lines:` loop for each raw line in this query's group, strip the newline, split it by tab into its 12 columns, and add that split-up line to `columns_per_line`. After this loop, `columns_per_line` looks like seperate "columns" of values. A list of lists, each inner list is one line already broken into its columns.
6. `best_evalue = columns_per_line[0][10]`  grab the evalue from the first (best-ranked) line: `"1.67e-164"`. default as the best hit/lowest evalue until examine ties
7. `best_subject = columns_per_line[0][1]` grab the subject protein ID from that same first line: `"ENSP00000417654"`.
	1. Need both the sub protein ID and the 
8. `if len(columns_per_line) > 1:`  is there more than one line for this query? If yes proceed to check for a tie. If there'd only been one line total, there's no second line to compare against, so `is_tied` gets set straight to `False`.
9. `second_evalue = columns_per_line[1][10]`  grab the evalue from the second-ranked line: `"1.01e-20"`.
10. `is_tied = (second_evalue == best_evalue)` is `"1.01e-20"` equal to `"1.67e-164"`? No. So `is_tied = False`.
11. `if not is_tied:`  since it's not tied, proceed. `best_hits[query] = best_subject`  store the pair: `best_hits["ENSDARP00000000004"] = "ENSP00000417654"`.
12. Loop moves to the next query (`ENSDARP00000000005`), repeats the same 11 steps for it.
13. Once every query in the file's been processed, `return best_hits` hands back the completed dictionary.
***THIS does not work, only looking at top 2 values for tie, not catching other case where there is the same queryID and sequenceID but different evalues, not real tie shouldn't be thrown out. need to adapt

#### 8.31.26

Fixed the tie logic from yesterday. Was only checking the top 2 lines which missed cases where a 3rd+ line had the same evalue as the top one, or where the same subject showed up more than once at the same evalue (not a real tie, just multiple alignment chunks for the same match). Now walking through all lines in a query's group, collecting every DISTINCT subject that shares the top evalue into a set, only calling it a tie if that set has more than 1 subject in it. Fixes both problems at once.

Added argparse so can run this on all 6 species combos without editing the script each time. Takes `-f1`/`-f2` for the two sorted blastp files (forward and reverse direction), `-b1`/`-b2` for the two biomart files, `-o` for output path.

Also wrote `get_biomart_lookup()` to build protein_id = (gene_id, gene_name) dicts from the biomart files. Skips header line, skips any row with a blank protein ID since that's the lookup key, stores empty string for missing gene name per the assignment's formatting note.

`find_reciprocal_hits()` compares the two best-hit dicts, keeps a pair only if species1's best hit is some species2 protein AND that species2 protein's best hit points back to the original species1 query.

Final output step writes the header row + one line per RBH pair, pulling gene ID/name from the biomart lookups for both proteins, falling back to empty strings if a protein isn't found in biomart at all.

Ran the script on all 6 species combinations (sorted files as input for both directions per pair) as an sbatch /projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/623_RBH/RBHsbatch.sh):

Final script, output files, pseudocode, and answers all live in `/projects/bgmp/hodapp/bioinfo/Bi621/PS/palimpsestkenlyn-Bi621-PS7/623_RBH`
test files and blast hits are in sub folders: `/623_RBH/blasthits` and `/623_RBH/test_files`

Output statistics/summary: 

```
Human-Zebrafish:
Percent of CPU this job got: 99%
Elapsed (wall clock) time (h:mm:ss or m:ss): 0:03.75
Maximum resident set size (kbytes): 148056
Exit status: 0

Human-ElectricEel:
Percent of CPU this job got: 99%
Elapsed (wall clock) time (h:mm:ss or m:ss): 0:02.28
Maximum resident set size (kbytes): 145392
Exit status: 0

Human-ElectricBabyWhale:
Percent of CPU this job got: 99%
Elapsed (wall clock) time (h:mm:ss or m:ss): 0:02.48
Maximum resident set size (kbytes): 142044
Exit status: 0

Zebrafish-ElectricEel:
Percent of CPU this job got: 99%
Elapsed (wall clock) time (h:mm:ss or m:ss): 0:02.81
Maximum resident set size (kbytes): 50512
Exit status: 0

Zebrafish-ElectricBabyWhale:
Percent of CPU this job got: 99%
Elapsed (wall clock) time (h:mm:ss or m:ss): 0:03.08
Maximum resident set size (kbytes): 45848
Exit status: 0

ElectricEel-ElectricBabyWhale:
Percent of CPU this job got: 99%
Elapsed (wall clock) time (h:mm:ss or m:ss): 0:01.86
Maximum resident set size (kbytes): 45980
Exit status: 0
```

### 9.1.26
Summary output of files to count # of RBH: 
```
wc -l *_RBH.tsv
  10663 ElectricEel_ElectricBabyWhale_RBH.tsv
   9036 Human_ElectricBabyWhale_RBH.tsv
   9023 Human_ElectricEel_RBH.tsv
   7962 Human_Zebrafish_RBH.tsv
   9342 Zebrafish_ElectricBabyWhale_RBH.tsv
  10186 Zebrafish_ElectricEel_RBH.tsv
  56212 total
```
need to subtract 1 for the header line. finals:
  
- ElectricEel_ElectricBabyWhale: 10662
- Human_ElectricBabyWhale_RBH: 9035
- Human_ElectricEel_RBH: 9022
- Human_Zebrafish_RBH: 7961 
- Zebrafish_ElectricBabyWhale_RBH: 9341
- Zebrafish_ElectricEel_RBH: 10185 

Counts and full answers to Part 5 questions written up in PS1_answers.txt.

## Part 4

Determine the number of reciprocal best hits per species combination in the python script or a bash script.

```

wc -l *_RBH.tsv

10663 ElectricEel_ElectricBabyWhale_RBH.tsv

9036 Human_ElectricBabyWhale_RBH.tsv

9023 Human_ElectricEel_RBH.tsv

7962 Human_Zebrafish_RBH.tsv

9342 Zebrafish_ElectricBabyWhale_RBH.tsv

10186 Zebrafish_ElectricEel_RBH.tsv

56212 total

```

As there is a header line added to each file, -1 should be subtracted from each total (which I did in my head rather than algorithmically bc I am very advanced) so the final counts of RBH per species combination are as follows:

ElectricEel_ElectricBabyWhale: 10662

Human_ElectricBabyWhale_RBH: 9035

Human_ElectricEel_RBH: 9022

Human_Zebrafish_RBH: 7961

Zebrafish_ElectricBabyWhale_RBH: 9341

Zebrafish_ElectricEel_RBH: 10185

  

## Part 5

1. How does the number of RBH’s vary across combinations? Any ideas why there is variance (biological or technical)?

RBH counts were highest between the two electric fish species (Eel-Pka, 10,662) and lowest for any pairing involving human (Human-Zebrafish, 7,961, the lowest overall). This tracks with evolutionary distance: human diverged from the fish lineages much earlier than the fish diverged from each other, giving more time for gene duplication, loss, and divergence to create a lot of differences and break 1:1 orthology. Also not surprising that two fish (who would be more closely related to eachother than say a human) would have a lot of orthologs as they would have a lot of shared ancestery, meaning lots of genes that haven't diverged or turned into paralogs or been lost in speciation/lineage splits. Annotation quality could also contribute, but doesn't fully explain the pattern, since zebrafish has an extremely well studied and one assumes pretty completely curated genome and still produced the lowest RBH count when paired with human, suggesting evolutionary distance is the stronger driver here. But in some cases data could also be a technical aspect (low data quality in the genome of a comparison species) that would explain variance.

  

2. Describe 1 situation where that you could use 1 or more of your reciprocal best hits file(s) either in an analysis or a workflow.

This could be used to look at phylogeny or evolutionary relationships between species. Having just read about PSMC, it seems like you could apply a similar theoretical framework here. Using coalescent theory and mutation-rate-calibrated time estimation, you could look at when species diverged based on when orthologous genes had their most recent common ancestor (coalescence time), then aggregate or bin that data across all RBH pairs to estimate a divergence rate and work backwards to a split time. I am sure there are fancier tools now (like genespace or orthofinder) that do this better, but it is an option. You could also use this to identify possible orthologs in not well annotated species to identify similar genes to measure in a differential expression experiment.

  

3. What are some limits to the RBH approach?

It does not take synteny into account. It is only using sequence similarity as a metric for ortholog identification, and this has gaps as without synteny (location/features surrounding the sequence) there can be multiple hits that are not the best or not true orthologs. So this can be an initial fairly easy way to probe at the evolutionary relationship between two species (genes) by doing this comparison directly. It is less demanding in terms of compute so it can scale and be done without as many resources. But it does not tell the whole picture, where something more complete like genespace or even orthofinder take more variables and possibly sequences into account to build a more complete picture. It also only catches strict 1:1 relationships by design, so a gene that duplicated in one lineage gets excluded entirely, even if one of the resulting copies/paralogs is a real ortholog that retained the original ancestral function and would be worthy of comparison. That means RBH misses a category of possible orthologs, so it is narrower due to filtering down to only the best. It is also only a species to species comaparison so again it can lack the broader comparison.

  

4. Why did we use protein sequences instead of gene sequences in this analysis?

Using protein sequences is a way to cut down on a lot of messy noise that can complicate the picture. With codon degeneracy there can be changes in DNA/RNA sequences that do not result in protein changes so do not always face the same selection pressure. That means there are more dissimilarities that must be worked through looking for matches which is computationally intensive. Using just the protein sequence hones in on the "functional" aspect and makes comparisons more directed. It also cuts out the noise related to introns, by removing those, the sequence comparison is only between the coding regions as they are represneted in the final protein sequence. This cuts down again on just amount of data to process, but biologically is also important as introns may be under less selective pressure so they may have more mutations or even vary in lengths significantly between species. This again makes a sequence alignment difficult so removing introns simplifies things. Basically the protein sequence is the more conserved aspect and also the least computationally intensive so it makes a better and easier comparison.

