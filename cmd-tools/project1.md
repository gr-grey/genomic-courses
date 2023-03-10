## Project 1: unix basic

Files needed for the project are [here](https://github.com/gr-grey/genomic-courses/blob/46af89aa31b5e30a75b834fcc6cf81c8a153f98c/cmd-tools/files/project1/gencommand_proj1_data.tar.gz)
for download and extraction. 

```bash
tar -xvf gencommand_proj1_data.tar.gz
```

### Q1, in apple.genome, each chromosome starts on a line with ">", how many chromosomes are there?

```bash
grep -c ">" apple.genome
# A: 3
```

### Q2-3, in apple.genes, the first column stores gene, the second column stores transcript, what's the number of unique genes and unique transcripts in the file?

For genes, cut out first column and `sort -u` to find uniq values, then count.
For transcripts, same operation for the second column.


```bash
# Solution 1, with cut, change to cut -f 2 for transcripts
cut -f 1 apple.genes | sort -u | wc -l 
# Solution 2, with awk, change to $2 for transcripts
awk '{print $1}' apple.genes | sort -u | wc -l 
# A: genes 5453, transcript 5456
```

### Q4-5, a gene can have multiple transcripts (two lines can have the same value in first column and differ in the second column), how many genes have only one transcripts, how many genes have more than one transcripts?

Cut out the first column and count the how many time each genes occurs, which gives the number of their transcripts. 
Then 

```bash
# genes with 1 transcript
cut -f 1 apple.genes | sort | uniq -c | grep -c " 1 "
cut -f 1 apple.genes | sort | uniq -c | awk '$1 == 1' | wc -l

# genes with >1 transcripts
cut -f 1 apple.genes | sort | uniq -c | grep -v " 1" | wc -l
cut -f 1 apple.genes | sort | uniq -c | awk '$1 > 1' | wc -l

# A: 1 transcript 5450, more than 1 transcripts 3
```

### Q6-7, in apple.gene, column 4 stores strand (+ or -) information, how many genes are on + strand, how many on - strand?
### Q8-13, column 3 stores chromosome information, count the number of genes on chr1, chr2, chr3; count the number of transcripts on chr1/2/3.

Question 6 through 13 are about filtering out lines that match certain value in a column. The solutions below covers the situation of counting genes. To count transcripts, change genes (column 1) to transcripts (col2).  

- Solution 1: cut out both gene column and filter column (strand or chromosome), uniq the combination, then count how many +/- strand or chromosomes there are. A problem with this method is if one gene maps to 2 strands/chromosomes (which shouldn't occur here), it'll count the gene twice. 
- Solution 2: similar to solution 1, after getting the uniq combination, we can also get the counts by `grep -c`
- Solution 3: awk can be used to filter value in a column. `awk '$4=="var"'` will return lines where column 4 matches "var". We can filter to get matching values, then count the number of unique genes.

To me, solution 3 is the most intuitive and easiest to read one, awk is a command worth learning for handling column based files.   

```bash
# solution 1
cut -f 1,4 apple.genes | sort -u | cut -f 2 | sort | uniq -c

# solution 2 with grep, 
#-w means work match with regular expression pattern 
# $ is end of line in regular expression.
cut -f 1,4 apple.genes | sort -u | grep -c -w "+$"
cut -f 1,4 apple.genes | sort -u | grep -c -w "\-$" # '-' is special char in reg exp

# solution 3 with awk
awk '$4 == "+"' apple.genes | cut -f 1 | sort -u | wc -l
awk '$4 == "-"' apple.genes | cut -f 1 | sort -u | wc -l

# for chromosome filters, change column 4 to col 3
cut -f 1,3 apple.genes | sort -u | cut -f 2 | sort | uniq -c
cut -f 1,3 apple.genes | sort -u | grep -c -w "chr1"
awk '$3 == "chr1"' apple.genes | cut -f 1 | sort -u | wc -l
# change to chr2/3, then for transcripts change column 1 to column

# A: genes on +/- strand: 2662/2791 
# genes on chr1/2/3: 1624/2058/1771
# transcripts on chr1/2/3: 1625/2059/1772
```

### Q14-17, column 1 in apple.conditionA/B/C stores gene, find out the number of genes exists (1) in both condition A and B, (2) only in A, (3) only in B, (4) in A,B,C.

First we need to cut out the gene columns in those 3 files for further comparison.

`comm file1 file2` compares two *sorted* files, returns 3 columns - lines only in file1; only in file2; in both file1 and file2.

We can pass in the number of columns to be *omitted* in the output, e.g. `comm -12 file1 file2` only returns the third column with common lines.

The last question asks to compare 3 files, we can output common genes of A and B, then compare it with C. 
Or a whacky way to achieve this is to concatenate genes from all 3 files, then count the genes that occur 3 times.

```bash
# prepare sorted files for comparison
cut -f 1 apple.conditionA | sort -u > condA_genes
cut -f 1 apple.conditionB | sort -u > condB_genes
cut -f 1 apple.conditionC | sort -u > condC_genes

# comm 3 column outputs: only in A; only in B; in both A and B 
# pass the column you want to omit as parameters
comm -12 condA_genes condB_genes | wc -l # genes in both A and B
comm -23 condA_genes condB_genes | wc -l # genes in only A
comm -13 condA_genes condB_genes | wc -l # genes in only B

# output common in A and B and compare with C
comm -12 condA_genes condB_genes > AB_genes
comm -12 AB_genes condC_genes| wc -l
# alternative solution, cat all three genes and count genes that occur 3 times
# cat file{1,2,3} is the same as saying cat file1 file2 file3
cat cond{A,B,C}_genes | sort | uniq -c | grep -c " 3 "

# A: genes in A&B 2410; only A 1206; only B 1243; ABC 1608
```

