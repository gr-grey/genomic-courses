## Project 2:

Files needed for the project are [here](https://github.com/gr-grey/genomic-courses/blob/474475e7a1e95277e73314bac412c75676f7febc/cmd-tools/files/project2/gencommand_proj2_data.tar.gz)

```bash
tar -xvf gencommand_proj2_data.tar.gz
```

### Q1-5 Inspect the "athal_wu_0_A.bam" file with samtools and answer the following questions:
  - How many alignments are there in the bam file? (Q1)
  - For SAM format, column 7 is the "RNEXT" field, which contains the reference name of the mate/next read, a value of "=" in this field means the mate is mapped to the same chromosome (most common for pair end sequencing); whereas a "*" means the mate is unmapped. In the given bam file, find out the number of alignments that (a) with a mate mapped to the same chromosome (Q4); and (b) with an unmapped mate (Q2)? 
  - How many alignments have a deletion (have 'D' in its CIGAR string / column 6)? (Q3)
  - How many alignments have a intron (have 'N' in its CIGAR string)? (Q5)

```bash
# Q1, samtools flagstat shows an overview of the file, including the number of alignment
samtools flagstat athal_wu_0_A.bam

# Q2 and Q4, cut out column 7 
# and count the occurrences of "=" (same chromosome) and "*" (unmapped)

# Solution 1: use uniq to count instances of "=" and "*"
samtools view athal_wu_0_A.bam | cut -f7 | sort | uniq -c
# Solution 2: use awk to count occurrences of "=" and "*"
samtools view athal_wu_0_A.bam | awk '$7 == "="' | wc -l # same with "*"
# Solution 3: bioawk supports $rnext variable, without having to remembering it's column 7
samtools view -h -o sam_athal.sam athal_wu_0_A.bam # covert bam to same file
bioawk -c sam '$rnext=="="' sam_athal.sam | wc -l # rnext is column 7

# Q3 and Q5, cut out column 6 and count the strings that contain 'D' (Q3) and 'N' (Q5)
# Solution 1: use grep to count instances strings that have 'D' in it
samtools view athal_wu_0_A.bam | cut -f6 | grep -c "D"
# Solution 2: use awk to match patterns with 'D' in it
samtools view athal_wu_0_A.bam | awk '$6 ~ /D/' | wc -l
# Solution 3: use bioawk to match patterns with 'D' in it
bioawk -csam '$cigar ~ /D/' sam_athal.sam | wc -l

# A: Q1 221372; Q2 65521; Q3 2451; Q4 150913; Q5 0
```

### Q6-10 from the original "athal_wu_0_A.bam" file, extract alignments with in the range of Chr3:11777000-1179400 to a new bam file, then answer the same set of question as Q1-5 for the new bam file.

```bash
# Extract alignments within the range and store them in range.bam
samtools view -b athal_wu_0_A.bam "Chr3:11777000-11794000" > range.bam
# repeat commands from Q1 to Q5 with the new range.bam file

# A: Q6 7081; Q7 1983; Q8 31; Q9 4670; Q10 0
```

Note: the bam file here are already sorted and indexed (the index file "athal_wu_0_A.bam.bai" is included).
However, in the case where a bam file is not sorted, you need to sort and index it before you can extract the range
```bash
# sort bam file
samtools sort athal_wu_0_A.bam -o mysorted.bam
# index, will create mysorted.bam.bai file
samtools index mysorted.bam
# extract range as before
samtools view -b mysorted.bam "Chr3:11777000-11794000" > range.bam
```

### Q11-13: check the header of the "athal_wu_0_A.bam" file to get more information
  - How many reference genomes were used to align the sequences? In header, @SQ SN lines has ref sequence info. (Q11)
  - Among all ref genomes above, what's the length of the first reference genome? (Q12)
  - What alignment program/tools was used? (In header, @PG shows the program used.) (Q13)

```bash
# Reference genome and alignment program can be found in the header
samtools view -H athal_wu_0_A.bam 
```

```
@HD     VN:1.0  GO:none SO:coordinate
@SQ     SN:Chr1 LN:29923332     AS:wu_0.v7.fas  SP:wu_0.v7.fas
@SQ     SN:Chr2 LN:19386101     AS:wu_0.v7.fas  SP:wu_0.v7.fas
@SQ     SN:Chr3 LN:23042017     AS:wu_0.v7.fas  SP:wu_0.v7.fas
@SQ     SN:Chr4 LN:18307997     AS:wu_0.v7.fas  SP:wu_0.v7.fas
@SQ     SN:Chr5 LN:26567293     AS:wu_0.v7.fas  SP:wu_0.v7.fas
@SQ     SN:chloroplast  LN:154478       AS:wu_0.v7.fas  SP:wu_0.v7.fas
@SQ     SN:mitochondria LN:366924       AS:wu_0.v7.fas  SP:wu_0.v7.fas
@RG     ID:H100223_GAII05_0002  PL:SLX  LB:wu_PII       PI:400  DS:wu_0_Genome  SM:wu_0
@RG     ID:Wii_PER01    PL:SLX  LB:wu_phaseI    PI:400  DS:wu_0_Genome  SM:wu_0
@RG     ID:Wii_PER02    PL:SLX  LB:wu_phaseI    PI:400  DS:wu_0_Genome  SM:wu_0
@RG     ID:Wii_SR03     PL:SLX  LB:wu_phaseI    PI:400  DS:wu_0_Genome  SM:wu_0
@PG     ID:stampy       VN:1.0.3_(r627) CL:-g /lustre/scratch103/sanger/xcg/tmp/tmp.zYfXz26246 -h /lustre/scratch103/sanger/xcg/tmp/tmp.zYfXz26246 --readgroup=ID:Wii_PER01,LB:wu_phaseI,SM:wu_0,PI:400,PL:SLX,DS:wu_0_Genome --bwaoptions=-q10 /lustre/scratch103/sanger/xcg/wu_0.v7.fas -o /lustre/scratch103/sanger/xcg/wu_0/A/aln_A1.sp0.sam -M /lustre/scratch103/sanger/xcg/wu_0/read_1_1.sp0.fq.gz /lustre/scratch103/sanger/xcg/wu_0/read_1_2.sp0.fq.gz
@PG     ID:samtools     PN:samtools     PP:stampy       VN:1.16.1       CL:samtools view -H athal_wu_0_A.bam
@CO     TM:Fri, 17 Sep 2010 12:20:13 BST        WD:/lustre/scratch103/sanger/xcg/wu_0/self     HN:bc-16-1-07.internal.sanger.ac.uk     UN:xcg
```

```bash
# Reference genome has SN: in the pattern
samtools view -H athal_wu_0_A.bam | awk '/SN:/' | wc -l
# or use grep
samtools view -H athal_wu_0_A.bam | grep -c "SN:"
# the length can be read off of SN:Chr1 LN:29923332
# the line @PG shows program used
samtools view -H athal_wu_0_A.bam | awk '/@PG/'

# A: Q11 7; Q12 29923332; Q13 stampy
```

### Q14-15, check the first alignment in the "athal_wu_0_A.bam" file and find out (a) the name/identifier (column 1) and (b) the start position of its mate (column 7, 8).
```bash
# display first alignment in the sam file
samtools view athal_wu_0_A.bam | head -1
samtools view athal_wu_0_A.bam | awk 'NR==1' # NR is line number in aw
# for bioawk, the first line has NR of 16 for some reason
bioawk -csam 'NR==16' sam_athal.sam

# cut out column 1 for name, col 7&8 for mate's position
samtools view athal_wu_0_A.bam | head -1 | cut -f1
samtools view athal_wu_0_A.bam | head -1 | cut -f7,8

# A: Q14 GAII05_0002:1:113:7822:3886#0; Q15 Chr3:11700332
```

### Q16-20 use bedtools to investigate overlapping intervals between the given "athal_wu_0_A_annot.gtf" file and the range.bam file in Q6-10, and answer the following questions
  - How many overlapping intervals (output with `-wo` option) are there between the 2 files? (Q16)
  - The length of the overlaps can be found in column 22 with the `-wo` option. How many overlaps has a length of 10 bases or longer? (Q17)
  - How many alignments in the bam file overlap with an interval in the gtf file?  (Q18)
  - Of all the intervals in gtf file that overlap with an alignment in the bam file, how many of them are exons? (Q19) 
  - How many unique transcripts (in column 9, first part of the string) are there in the gtf file (Q20)
 
```bash
# Q16 when use a bam file as input, need to add the -bed flag for outputting in bed format
bedtools intersect -a range.bam -b athal_wu_0_A_annot.gtf -wo -bed | wc -l

# Q17 Count the lines where column 22 has a value >= 10
bedtools intersect -a range.bam -b athal_wu_0_A_annot.gtf -wo -bed | bioawk -t '$22 >=10' | wc -l
# bioawk -t sets the delimiter to tab
# when use awk, need to specify awk -F'\t' -v OFS="\t" 
# F sets input sep, OFS stands for output field separator

# Q18 the "-c" option counts how many intervals in file B overlaps with each intervals in file A
# The counts are stored in column 7, count the number of alignments that has >0
bedtools bamtobed -i range.bam > range.bed
bedtools intersect -a range.bed -b athal_wu_0_A_annot.gtf -c | bioawk -t '$7 >0' | wc -l
# I ran into some problem with the -c flag with bam file, so I converted to bed file first

# Q19 similar to 18, use the gtf as file a, either bam or bed file as b
# the counts are stored in column 10 since gtf came with more columns
bedtools intersect -a athal_wu_0_A_annot.gtf -b range.bam -bed -c | bioawk -t '$10>0' | wc -l

# Q20 cut out column 9 with both transcript and gene name as a string
# then get only the transcripts part and count uniq instances.
cut -f9 athal_wu_0_A_annot.gtf | cut -d ' ' -f1,2 | sort -u | wc -l

# A: Q16 3101; Q17 2899; Q18 3101; Q19 21; Q20 4
```
