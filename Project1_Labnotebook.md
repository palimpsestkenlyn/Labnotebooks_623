Environment:
```
RStudio Version 2026.08.0+187 (2026.08.0+187)
Talaps: pixi toml file 
```
## Part 1 RoCC script: 9/6/26- Talapas

###  Data exploration: Started by looking at the data (partial output shown for ref of format):

```
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo) 10:24 PM $ zcat /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/241-mammalian-2020v2.bigWigToBedGraph.gz | head -50
chr1    10074   10075   0.053
chr1    10075   10078   0.064
chr1    10078   10079   -2.109
chr1    10079   10081   0.053
chr1    10081   10084   0.064
chr1    10084   10085   -2.109
chr1    10085   10087   0.053
chr1    10087   10090   0.064
chr1    10090   10093   0.053
chr1    10093   10096   0.064
chr1    10096   10099   0.053
chr1    10099   10102   0.064
chr1    10102   10103   0.053
chr1    10103   10104   -2.603
chr1    10104   10105   -2.413
chr1    10105   10106   0.064
chr1    10106   10107   -2.112
```

This was not what I expected, I thought it would be by bp position, but it has stretches... what is this? Ah the coordinates only change when the score does, so there are already some stretches. 

This makes assessing continuous more tricky, not line to line if there are gaps... can there be gaps?

```
zcat /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/241-mammalian-2020v2.bigWigToBedGraph.gz | head -5 | awk '{print $3 - $2}'
1
3
1
2
3
```

Important considerations:
- Four columns: chromosome, start, end, phyloP value/score
- BedGraph format- zero based, half-open... so => start:inclusive end:exclusive describes position of bases again starting zero based. 
	- `chr1 0 1` would be the VERY first base of chr1 listed as the 0 based start position (included) and the 1 position end (not included), yes
	- Essentially though for math `end - start` = length of segment
	- chr1 10075 10078 0.064 `end - start` = 3bp
	- numbers start over 

Thinking through code needs, biology, and starting on pseudocode:

==need to add more to this, and also the slurm out stuff.....

## Part 2 Clinical variant processing: R 9/8/26- 
Talapas with pixi to run R, or testfiles and write R script on computer then run on Talapas, OR download data if its resonable and just do on R then upload output back up to Talapas

### Data exploration 
Downloaded zip file onto local computer to do parsing of data for comparison. Library found 
/projects/bgmp/shared/Bi623/ZoonomiaWorkshop/variant_summary.txt.gz

working in Rstudio on my local computer. 

#### initial exploration indicates lots of complexity:
```
glimpse(variants)
Rows: 8,918,806
Columns: 43
$ `#AlleleID`                                <dbl> 15041, 15041, 15042, 15042, 15043, 15043, 15044, 15044, 15045,…
$ Type                                       <chr> "Indel", "Indel", "Deletion", "Deletion", "single nucleotide v…
$ Name                                       <chr> "NM_014855.3(AP5Z1):c.80_83delinsTGCTGTAAACTGTAACTGTAAA (p.Arg…
$ GeneID                                     <dbl> 9907, 9907, 9907, 9907, 9640, 9640, 55572, 55572, 55572, 55572…
$ GeneSymbol                                 <chr> "AP5Z1", "AP5Z1", "AP5Z1", "AP5Z1", "ZNF592", "ZNF592", "FOXRE…
$ HGNC_ID                                    <chr> "HGNC:22197", "HGNC:22197", "HGNC:22197", "HGNC:22197", "HGNC:…
$ ClinicalSignificance                       <chr> "Pathogenic/Likely pathogenic", "Pathogenic/Likely pathogenic"…
$ ClinSigSimple                              <dbl> 1, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
$ LastEvaluated                              <chr> "Dec 17, 2024", "Dec 17, 2024", "Jun 29, 2010", "Jun 29, 2010"…
$ `RS# (dbSNP)`                              <dbl> 397704705, 397704705, 397704709, 397704709, 150829393, 1508293…
$ `nsv/esv (dbVar)`                          <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ RCVaccession                               <chr> "RCV000000012|RCV005255549|RCV004998069", "RCV000000012|RCV005…
$ PhenotypeIDS                               <chr> "MONDO:MONDO:0013342,MedGen:C3150901,OMIM:613647,Orphanet:3065…
$ PhenotypeList                              <chr> "Hereditary spastic paraplegia 48|Macular dystrophy with or wi…
$ Origin                                     <chr> "germline;unknown", "germline;unknown", "germline", "germline"…
$ OriginSimple                               <chr> "germline", "germline", "germline", "germline", "germline", "g…
$ Assembly                                   <chr> "GRCh37", "GRCh38", "GRCh37", "GRCh38", "GRCh37", "GRCh38", "G…
$ ChromosomeAccession                        <chr> "NC_000007.13", "NC_000007.14", "NC_000007.13", "NC_000007.14"…
$ Chromosome                                 <chr> "7", "7", "7", "7", "15", "15", "11", "11", "11", "11", "14", …
$ Start                                      <dbl> 4820844, 4781213, 4827361, 4787730, 85342440, 84799209, 126145…
$ Stop                                       <dbl> 4820847, 4781216, 4827374, 4787743, 85342440, 84799209, 126145…
$ ReferenceAllele                            <chr> "na", "na", "na", "na", "na", "na", "na", "na", "na", "na", "n…
$ AlternateAllele                            <chr> "na", "na", "na", "na", "na", "na", "na", "na", "na", "na", "n…
$ Cytogenetic                                <chr> "7p22.1", "7p22.1", "7p22.1", "7p22.1", "15q25.3", "15q25.3", …
$ ReviewStatus                               <chr> "criteria provided, multiple submitters, no conflicts", "crite…
$ NumberSubmitters                           <dbl> 4, 4, 1, 1, 1, 1, 6, 6, 2, 2, 6, 6, 57, 57, 57, 57, 16, 16, 2,…
$ Guidelines                                 <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "A…
$ TestedInGTR                                <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "Y…
$ OtherIDs                                   <chr> "ClinGen:CA215070,OMIM:613653.0001", "ClinGen:CA215070,OMIM:61…
$ SubmitterCategories                        <dbl> 3, 3, 1, 1, 1, 1, 3, 3, 3, 3, 2, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3,…
$ VariationID                                <dbl> 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 214885, 214885, 9, 9, 10, 10, 11…
$ PositionVCF                                <dbl> 4820844, 4781213, 4827360, 4787729, 85342440, 84799209, 126145…
$ ReferenceAlleleVCF                         <chr> "GGAT", "GGAT", "GCTGCTGGACCTGCC", "GCTGCTGGACCTGCC", "G", "G"…
$ AlternateAlleleVCF                         <chr> "TGCTGTAAACTGTAACTGTAAA", "TGCTGTAAACTGTAACTGTAAA", "G", "G", …
$ SomaticClinicalImpact                      <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ SomaticClinicalImpactLastEvaluated         <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ ReviewStatusClinicalImpact                 <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ Oncogenicity                               <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ OncogenicityLastEvaluated                  <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ ReviewStatusOncogenicity                   <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ SCVsForAggregateGermlineClassification     <chr> "SCV001451119|SCV005622007|SCV005909190", "SCV001451119|SCV005…
$ SCVsForAggregateSomaticClinicalImpact      <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
$ SCVsForAggregateOncogenicityClassification <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…

```

Going to need columns for the variables to filter by. data in many categories is inconsistent, different case types Cranio vs cranio. also the designation for pathogenic in ClinicalSignifigance is very tricky as there are 99 different combos and many contain the words described.... what counts?
#### counts detail on Clinical Significance 
```
pathogenictypes = variants %>%
  count(ClinicalSignificance, sort = TRUE)

Pathogenic	402554
Conflicting classifications of pathogenicity	323607
Likely pathogenic	236466
Pathogenic/Likely pathogenic	78509
Pathogenic, low penetrance	193
Likely pathogenic, low penetrance	175
Pathogenic; other	150
Pathogenic; drug response	83
Pathogenic; risk factor	45
Conflicting classifications of pathogenicity; other	42
Conflicting classifications of pathogenicity; risk factor	34
Likely pathogenic; drug response	34
Likely pathogenic/Likely risk allele	26
Pathogenic/Likely risk allele	20
Pathogenic; Affects	16
Conflicting classifications of pathogenicity; association	14
Pathogenic/Likely pathogenic/Pathogenic, low penetrance	14
Likely pathogenic; association	10
Likely pathogenic; risk factor	10
Pathogenic/Likely pathogenic; other	10
Pathogenic/Pathogenic, low penetrance	10
Likely pathogenic/Likely pathogenic, low penetrance	8
Likely pathogenic/Pathogenic, low penetrance	8
Pathogenic/Likely pathogenic, low penetrance	8
Pathogenic/Likely pathogenic/Likely risk allele	8
Pathogenic; association	8
Conflicting classifications of pathogenicity; drug response	6
Conflicting classifications of pathogenicity; other; risk factor	6
Pathogenic/Likely pathogenic; risk factor	6
Conflicting classifications of pathogenicity; association; risk factor	4
Conflicting classifications of pathogenicity; protective	4
Pathogenic; protective	4
Conflicting classifications of pathogenicity; Affects	2
Conflicting classifications of pathogenicity; drug response; other	2
Likely pathogenic; Affects	2
Likely pathogenic; protective	2
Pathogenic/Likely pathogenic/Established risk allele	2
Pathogenic/Likely pathogenic/Pathogenic, low penetrance/Established risk allele; risk factor	2
Pathogenic/Likely pathogenic; association	2
Pathogenic/Likely pathogenic; drug response	2
Pathogenic/Pathogenic, low penetrance; other	2
Pathogenic; confers sensitivity	2
Uncertain significance; Pathogenic/Likely pathogenic	2
Showing 17 to 43 of 43 entries (filtered from 103 total entries), 2 total columns
```

#### Approach: Regex case-insensitivity was required because PhenotypeList entries are not consistently capitalized meaning "Cranio" and "cranio" both appear as valid substrings referring to the same phenotype category, so a case-sensitive match would silently drop legitimate craniofacial records depending on submitter formatting.

ClinicalSignificance filtering requires some thought. Full field exploration via `count(ClinicalSignificance, sort = TRUE)` returned 103 distinct values, most representing compound classifications separated by `;` or `/` (e.g., "Pathogenic; risk factor," "Pathogenic/Likely pathogenic/Pathogenic, low penetrance"). Assignment instructions specify inclusion criteria as "Pathogenic (include Pathogenic and Pathogenic/Likely pathogenic)" ... read as an exact-match specification naming two categories, not a substring-matching instruction.

Decision: filter retained via exact string match on `ClinicalSignificance %in% c("Pathogenic", "Pathogenic/Likely pathogenic")` rather than substring/regex matching on the word "pathogenic." 
- Several excluded categories are not equivalent to a pathogenic call despite containing the word: "Conflicting classifications of pathogenicity" (323,607 records) indicates submitter disagreement, not a pathogenic consensus. "Likely pathogenic" alone (236,466 records) is a distinct, lower-confidence ClinVar tier from "Pathogenic/Likely Pathogenic" which seems to indicate a higher degree/significance towards pathogenicity. Also was not named in the assignment's inclusion list.
- A small set of compound qualified strings containing "Pathogenic" (e.g., "Pathogenic, low penetrance," "Pathogenic; drug response," "Pathogenic; risk factor") were also excluded under the exact-match approach. Combined, these account for approximately 560-600 records, so under 0.15% of the 481,063 records captured by the exact match.
- Clarification was requested from Hope instructor regarding whether qualified/compound pathogenic strings should be included. No response received prior to continuing; exact-match interpretation retained as the literal reading of assignment text, given negligible impact either way. The only significant thing that would necesitate redoing is the Likely pathogenic stand alone as it is a significant #. 
	- Likely pathogenic	236466  - 
	- Pathogenic/Likely pathogenic	78509
Filtering done with PR1_part2.R
```
library(tidyverse)

variants <- read_tsv("variant_summary.txt.gz")

cranio_variants = variants %>%
  filter(
    str_detect(PhenotypeList, regex("Cranio", ignore_case = TRUE)),
    ClinicalSignificance %in% c("Pathogenic", "Pathogenic/Likely pathogenic"),
    Assembly == "GRCh38"
  ) %>%
  select(Chromosome, Start, Stop, PhenotypeList) %>%
  arrange(Chromosome, Start) %>%
  mutate(Chromosome = paste0("chr", Chromosome))

write_tsv(cranio_variants, "Cranio_variants_sorted.tsv")
```

Verification performed post-filtering for a gut check:
```
pathogenictypes = variants %>%
+   count(ClinicalSignificance, sort = TRUE)
> View(pathogenictypes)
> length(readLines("Cranio_variants_sorted.tsv")) - 1
[1] 562
> variants %>%
+     filter(str_detect(PhenotypeList, regex("Cranio", ignore_case = TRUE))) %>%
+     nrow()
[1] 9273
> variants %>%
+     filter(str_detect(PhenotypeList, regex("Cranio", ignore_case = TRUE))) %>%
+     distinct(Assembly) %>%
+     count(Assembly)
# A tibble: 3 × 2
  Assembly     n
  <chr>    <int>
1 GRCh37       1
2 GRCh38       1
3 na           1

 variants %>%
+     filter(str_detect(PhenotypeList, regex("Cranio", ignore_case = TRUE))) %>%
+     count(Assembly)
# A tibble: 3 × 2
  Assembly     n
  <chr>    <int>
1 GRCh37    4644
2 GRCh38    4624
3 na           5

```
- Pre-filter "Cranio" match count (any assembly, any classification): 9,273 records.
- Assembly split within Cranio-matched records: GRCh37 = 4,644, GRCh38 = 4,624, na = 5 (sums correctly to 9,273; no records lost in filtering step). what would na even be? don't look and get lost. 
- Final filtered output (Cranio + Pathogenic/Pathogenic-Likely-pathogenic + GRCh38): 562 records, don't have a good grasp on general idea so good enough
- 
Uploaded to github, onto talaps and ready to move onto part 3. 


## Part 3 Data integration: 9/8/26 Talapas

Part 3 need, on Talapas:

- The RoCC output file from Part 1 (`PhyloP_RoCC_output_sorted.txt`)
- `Cranio_variants_sorted.tsv` from Part 2
- The 5 ATAC-seq narrowPeak files from `/projects/bgmp/shared/Bi623/ZoonomiaWorkshop/`
- bedtools installed via pixi
- All of those prepped into the format `bedtools multiinter` expects before running it... sorting nightmare why is this so mean. WHY!

Installed bedtools with pixi
```
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR1) 06:35 PM $ pixi init
✔ Created /gpfs/projects/bgmp/hodapp/bioinfo/Bi623/PR1/pixi.toml
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR1) 06:35 PM $ pixi add bedtools
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
 WARN cache for PypiMapping at /home/hodapp/.cache/rattler/cache/conda-pypi-mapping is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/conda-pypi-mapping for this run. Set [cache.pypi-mapping] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
✔ Added bedtools >=2.31.1,<3
```

Look at the provided datafiles to see if they are sorted int he default expected order for bedtools sort. multiinter expects all the files to be sorted the same. 

```
zcat /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/GSM7508786_CS18-12676-ATAC_peaks-q1.3.narrowPeak.gz | cut -f1 | uniq
chr1
chr10
chr11
chr12
chr13
chr14
chr15
chr16
chr17
chr18
chr19
chr2
chr20
chr21
chr22
chr3
chr4
chr5
chr6
chr7
chr8
chr9
chrX
chrY
zcat GSM7508790_CS23-12492-ATAC_peaks-q1.3.narrowPeak.gz | cut -f1 | uniq
SAME
```
confirmed the files are in the expected bedtools sort default order (by chromosome lexicographically, and then by start position numerically). so need to change my files to match those. R file will require getting rid of number line column, and both will need resorted. use either `bedtools sort -i input.bed > input.sorted.bed` OR `sort -k1,1 -k2,2n input.bed > input.sorted.bed`

```
pixi run bedtools sort -i /projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part1_RoCC/PhyloP_RoCC_output_sorted.txt > PhyloP_RoCC_multiinter_sorted.txt
```

There are column names on my Cranio file, so have to cut those. just start output from line 2 on...
```
tail -n +2 /projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part2/Cranio_variants_sorted.tsv | pixi run bedtools sort -i - > Cranio_multiinter_sorted.tsv
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
```

oh myyyyy gawd. the bedtools sort function (and the bash sort order described) take into account only columns 1, then 2. but guess what?!?! if there are ties, then it goes to the third column and if they are not ALSO sorted, then it errors out. fun.... resort 
```
sed -n '33,36p' /projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3/Cranio_multiinter_sorted.tsv
chr10   121517317       121517317       FGFR2-related craniosynostosis|Pfeiffer syndrome
chr10   121517318       121517319       FGFR2-related craniosynostosis
chr10   121517318       121517318       FGFR2-related craniosynostosis|FGFR2-related disorder
chr10   121517319       121517319       FGFR2-related craniosynostosis
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3) 07:19 PM $ sort -k1,1 -k2,2n -k3,3n -o Cranio_multiinter_sorted.tsv Cranio_multiinter_sorted.tsv
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3) 07:22 PM $ sed -n '33,36p' /projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3/Cranio_multiinter_sorted.tsv
chr10   121517317       121517317       FGFR2-related craniosynostosis|Pfeiffer syndrome
chr10   121517318       121517318       FGFR2-related craniosynostosis|FGFR2-related disorder
chr10   121517318       121517319       FGFR2-related craniosynostosis
chr10   121517319       121517319       FGFR2-related craniosynostosis
```

```
sort -k1,1 -k2,2n -k3,3n
```
rerun, with success. Ran /projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3/me_run_multiinter.sh

Usage summary:

```
 WARN cache for Repodata at /home/hodapp/.cache/rattler/cache/repodata is on a network/parallel filesystem (NFS/SMB/FUSE/BeeGFS/Lustre/GPFS/CephFS), redirected to /tmp/pixi-cache-hodapp/repodata for this run. Set [cache.repodata] in config.toml or PIXI_CACHE_DIR to override, or [cache.netfs-redirect] = "never" to keep the original path.
	Command being timed: "pixi run bedtools multiinter -i /projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3/PhyloP_RoCC_multiinter_sorted.txt /projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3/Cranio_multiinter_sorted.tsv /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/GSM7508786_CS18-12676-ATAC_peaks-q1.3.narrowPeak.gz /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/GSM7508787_CS18-12695-ATAC_peaks-q1.3.narrowPeak.gz /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/GSM7508788_CS19-12696-ATAC_peaks-q1.3.narrowPeak.gz /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/GSM7508789_CS22-12498-ATAC_peaks-q1.3.narrowPeak.gz /projects/bgmp/shared/Bi623/ZoonomiaWorkshop/GSM7508790_CS23-12492-ATAC_peaks-q1.3.narrowPeak.gz -names RoCC Cranio GSM7508786 GSM7508787 GSM7508788 GSM7508789 GSM7508790"
	Percent of CPU this job got: 96%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:06.32
	Maximum resident set size (kbytes): 21252
	Exit status: 0
```

got final output .bed file. 

looked at the overlap, frequency of how often the files overlap
```
awk '{print $4}' multiinter_output.bed | sort | uniq -c
 809871 1
 111253 2
  69557 3
  47147 4
  25316 5
   6347 6
     15 7
```

15 areas where all of the data files (all 7 inputs) overlap. Seems like the best way to narrow it down for further analysis as all the signal coming from this analysis support/indicate that these regions are likely important to cranio development.


```
TALAPAS login2 (/projects/bgmp/hodapp/bioinfo/Bi623/PR1/palimpsestkenlyn-Bi623-Project-1/Part3) 08:16 PM $ awk '$4 == 7' multiinter_output.bed
chr19   42217702        42217737        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42242544        42242577        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42242582        42242625        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42255089        42255248        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42268252        42268292        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42268401        42268435        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42268449        42268471        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42268523        42268543        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42268704        42268727        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42269187        42269213        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42269226        42269267        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42283777        42283812        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42283872        42283945        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr19   42283971        42283994        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
chr22   42615319        42615321        7       RoCC,Cranio,GSM7508786,GSM7508787,GSM7508788,GSM7508789,GSM7508790      1       1       1       1       1       1       1
```
## Part 4 visualization: 9/10/26- R

installed packages:
```
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("plotgardener")
BiocManager::install("plyranges")
BiocManager::install("grid")
BiocManager::install("TxDb.Hsapiens.UCSC.hg38.knownGene")
BiocManager::install("org.Hs.eg.db")
```

    
So i have my 15 areas where all of the data files (all 7 inputs) overlap from part 3, i will now plot ONE of these across the various files/inputs so i can see how they overlap.

### Reasoning

#### Background and region selection
Selected a region on chr19 based on the Part 3 bedtools multiinter output. This region showed depth 7 overlap (all 7 input files: RoCC, ClinVar Cranio variants, and all 5 ATAC-seq narrowPeak files) across multiple sub-intervals spanning roughly chr19:42217702-42283994.

Started with the middle section with most overlap, 42268252- 42269267 in the middle, thinking I can look and expand if needed. Graphs look confusing. Research and initial gene lookup at the multiinter coordinates pointed to CIC (Capicua Transcriptional Repressor, chr19:42268530-42295801). CIC's own disease associations in the literature are a neurodevelopmental disorder (MRD45) and a sarcoma gene fusion, not a craniofacial condition. Kept this as the working window regardless since the RoCC/ClinVar/ATAC convergence itself was the point of interest, not CIC's own literature. But it is meant to relate to cranio facial development importance??? Try a different range. Also tried collapse = TRUE and not included, but the. multiple tracks per data type look weird? 

Tried 42217702- 42269267 which shows a much expanded view, still lots of info but capturing the tiny CIC from above, a gap, then a stretch of interesting genes/data. Pulling the actual ClinVar records that fall in this stretch turned up ERF (ETS2 Repressor Factor, chr19:42247569-42255128), located about 13kb upstream of CIC. ERF causes Craniosynostosis 4 (CRS4), a well documented autosomal dominant craniofacial disorder. 26 ClinVar records cluster tightly at this locus (42248843-42255043), all point mutations. This is a much stronger, literature-supported craniofacial gene than CIC and is the more defensible choice to build the case around.

Narrowed to 42217702- 42255248 to focus only on this already large area and cut off gap and CIC info.

#### Data preparation
Downloaded the 5 ATAC-seq narrowPeak.gz files from Talapas to my computer No unzipping required, worked with .gz.

Read files into R as GRanges objects using plyranges:
- read_narrowpeaks() for the 5 ATAC files, since narrowPeak is a defined format plyranges parses directly
- read_bed() for the RoCC file and ClinVar Cranio file, since these are custom tab separated files rather than strict BED format

#### plotgardener setup
Installed plotgardener v1.18.0 (Bioconductor release matching R 4.6/Bioc 3.23). Confirmed hg38 is plotgardener's default assembly and matches all my input files' GRCh38 coordinates.

Basic structure follows: pageCreate() first to set up a blank canvas with defined width/height in inches, then a pgParams() object holding the shared genomic region (chrom, chromstart, chromend, assembly), then individual plotRanges()/plotGenes() calls placed at specific x/y coordinates, each referencing the shared params object so all tracks stay aligned to the same region.

Required loading TxDb.Hsapiens.UCSC.hg38.knownGene and org.Hs.eg.db separately for plotGenes() to have annotation data to draw from. Without these loaded, plotGenes() runs without error but produces an empty track.

plotRanges() defaults to pileup mode, stacking overlapping features into multiple rows rather than one. Used collapse = TRUE on every track to force a single row per file, since I want one clean row per dataset, as the multiple tracks looks intense. 

#### Issues encountered
Had confusing troubleshooting around a track (RoCC) appearing to go missing between renders. Turned out to be inconsistent rendering in the RStudio Plots pane depending on window/pane size, maybe? Render directly to a PDF at fixed dimensions using pdf()/dev.off() creates its own issues. Just leaving smaller dimensions. 

Widened the plotted window to look at more of the region (42217702-42268727) to get better context. This revealed 4 genes in view (ZNF526, DEDD2, GSK3A, ERF) rather than just CIC.

#### QUESTIONS?
Should i filter out the large clinvar tracks in my graphs?- not needed, is real signal depends on what I want to focus on. 
Should I be doing the collapse true? - yes

Ok so the clinvar data is proving to be very tricky given all that i included
```
chr19	41952441	42266625	Syndromic craniosynostosis
chr19	42032860	42297536	Syndromic craniosynostosis
chr19	42248842	42248842	Lambdoidal craniosynostosis
chr19	42248910	42248911	not provided|Lambdoidal craniosynostosis|Neurodevelopmental disorder|TWIST1-related craniosynostosis|Chitayat syndrome|ERF-related disorder|Noonan Syndrome-like developmental disorder|Noonan-like syndrome|Noonan syndrome
chr19	42249039	42249040	not provided|TWIST1-related craniosynostosis
chr19	42249078	42249115	TWIST1-related craniosynostosis
chr19	42249091	42249091	TWIST1-related craniosynostosis
chr19	42249199	42249201	TWIST1-related craniosynostosis|not provided
chr19	42249220	42249221	Lambdoidal craniosynostosis|TWIST1-related craniosynostosis|See cases|Inborn genetic diseases|not provided|Chitayat syndrome|Chitayat syndrome;Lambdoidal craniosynostosis|Noonan-like syndrome
chr19	42249255	42249256	TWIST1-related craniosynostosis
chr19	42249379	42249379	TWIST1-related craniosynostosis
chr19	42249415	42249415	Inborn genetic diseases|not provided|Lambdoidal craniosynostosis|Noonan Syndrome-like developmental disorder
chr19	42249460	42249460	not provided|TWIST1-related craniosynostosis|Noonan Syndrome-like developmental disorder|Inborn genetic diseases
chr19	42249493	42249493	TWIST1-related craniosynostosis|Inborn genetic diseases|not provided|Noonan Syndrome-like developmental disorder
chr19	42249545	42249546	TWIST1-related craniosynostosis|Lambdoidal craniosynostosis
chr19	42249565	42249565	Lambdoidal craniosynostosis|TWIST1-related craniosynostosis|Lambdoidal craniosynostosis;Chitayat syndrome|Noonan-like syndrome|not provided
chr19	42249685	42249685	Inborn genetic diseases|TWIST1-related craniosynostosis|not provided
chr19	42249927	42249928	TWIST1-related craniosynostosis
chr19	42250332	42250332	Lambdoidal craniosynostosis|not provided|TWIST1-related craniosynostosis|ERF-related disorder|Inborn genetic diseases
chr19	42250335	42250335	TWIST1-related craniosynostosis
chr19	42250365	42250365	TWIST1-related craniosynostosis
chr19	42250444	42250444	TWIST1-related craniosynostosis
chr19	42250467	42250467	TWIST1-related craniosynostosis
chr19	42250485	42250485	Common craniosynostosis syndromes
chr19	42250517	42250517	TWIST1-related craniosynostosis|Lambdoidal craniosynostosis;Chitayat syndrome
chr19	42250567	42250567	Lambdoidal craniosynostosis
chr19	42254967	42255043	TWIST1-related craniosynostosis
chr19	42254999	42254999	Lambdoidal craniosynostosis|not provided
chr19	45357482	45357484	Craniopharyngioma|Xeroderma pigmentosum|Cerebrooculofacioskeletal syndrome 2|not provided|Xeroderma pig
```

so these two first entries:
```
chr19	41952441	42266625	Syndromic craniosynostosis
chr19	42032860	42297536	Syndromic craniosynostosis
```

are huge and seem to cover my entire region of interest. its making the clinvar bar a solid red when i collapse it.... it seems like these are different TYPES of information being captured in clinvar...copy-number variants or large structural deletions/duplications associated with syndromic craniosynostosis which are these HUGE chunks a(~314 kb and ~264 kb) different variant class than the small point mutations likely showing up in the ERF gene. Both are legitimate ClinVar records, just fundamentally different kinds of evidence (a chromosomal-scale rearrangement vs. a single-nucleotide change in one gene). What am I trying to look at here? Well it depends on the scientific question, both things are real signal. In this case doing more of an exploration, but it was interesting to do both. 

Plot(s) many are finished. Plotgardner is a living nightmare where nothing makes sense (make the dimensions of the canvas/page bigger and you see less data!?!?!?!) but it is done now. Project complete, no report for this project. 


