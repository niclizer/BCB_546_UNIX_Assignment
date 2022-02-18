# UNIX Assignment

## Data Inspection

#### Used code below in inspect the file size of both fang_et_al and snp txt files

```
du -h snp_position.txt fang_et_al_genotypes.txt
```
By doing this I learned that `snp_position` is `41k` and `fang_et_al_genotypes` is `6.6M`.

I then wanted to learn the number of columns in each file. Code Utilized was -

```
tail -n +6 fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
```
and
```
tail -n +6 snp_position.txt | awk -F "\t" '{print NF; exit}'
```

fang and snp have `986` columns and `15` columns respectively. 

I then wanted to know the number of lines, words and characters in each file. I used the following command.

```
wc fang_et_al_genotypes.txt snp_position.txt
```

This gave me -
| File Name |Line | Word | Character |
|----|----|-----|---|
|fang_et_al|2783|2744038|11051939|
|snp_position|984|13198|82763|

Following the assignment guidance I then used the transpose.awk command to transpose the fang_et_al then used the above column and word count commands to make the table look like this -

This gave me -
| File Name |Line | Word | Character | Column|
|----|----|-----|---|---|
|fang_et_al|2783|2744038|11051939|986
|snp_position|984|13198|82763|15
|transposed_genotypes|986|2744038|11051939|2783

So the column and lines change places but the words and character counts remain the same.



# Data Processing

## Maize Data

### subset of three maize genotypes
```
grep -E "(ZMMIL|ZMMLR|ZMMR|Group)" fang_et_al_genotypes.txt > maize_3_geno.txt
```

This will grab every line that has either ZMMIL, ZMMLR, or ZMMR, plus the first line (this has all the SNP-ID data) and output them to a new file. The same was done of the teosinte genotypes.
### Transpose

```bash
awk -f transpose.awk maize_3_geno.txt > transposed_maize.txt
```

The above command transposed the maize genotype file useing the awk command provided.

### Sorting

Next the transposed file needed to be sorted as well as the snp_positions file. To avoid sorting the header I used the following command -
```
 (head -n 1 snp_position.txt && tail -n +2 snp_position.txt | sort -k1,1 ) > sorted_snp.txt

```
It looked only slightly different for the maize and teosinte sorting -
```
(head -n 3 transposed_maize.txt && tail -n +4 transposed_maize.txt | sort -k1,1 ) >sorted_trans_maize.txt

(head -n 3 transposed_teosinte.txt && tail -n +4 transposed_teosinte.txt | sort -k1,1 ) >sorted_trans_teosinte.txt
```

I also double checked that they were sorted with the command below; with no output meaning it was successfully sorted.

```
sort -c 
echo $?
```

Next I wanted to remove the header from the genotype files as it was no longer needed and might interfer with the sorting in the next step.

```
tail -n+4 sorted_trans_maize.txt > headless_maize.txt
tail -n+4 sorted_trans_teosinte.txt > headless_teosinte.txt
```
### Cutting out SNP-ID, Chromosome, and Position columns

Then I removed the other columns in the snp_positions file aside from column 1,3,4 which contains the SNP-ID, chromosome, and position data respectively.
```
cut -f 1,3,4 sorted_snp.txt > sorted_snp_cut.txt
```

### Joining

Next is the command to join the sorted snp_positions file with the transposed>sorted>headless maize and teosinte genotype files.

```
join -1 1 -2 1 -t $'\t' sorted_snp_cut.txt headless_maize_sort.txt > snp_maize_3.txt
join -1 1 -2 1 -t $'\t' sorted_snp_cut.txt headless_teosinte_sort.txt > snp_teosinte_3.txt
```
### Grabbing just individual chromosomes, sorting based on position, and replacing missing values

I then shuffled these files into folders for each grouping of data (maize increasing, decreasing, unknown, and multi). Same with teosinte.

To pull out just the indiviual chromosome I used an awk command
```
awk '$2 == "1"' snp_maize_3.txt >maize_chrom1.txt 
```
and to confirm I used -
```
cut -f 2 maize_chrom1.txt | sort | uniq -c
```
It only showed a single number, the chromosome I selected for.

### Sort based on Position and replacing missing values
Then sorted based on Position (column 3) which was done with -
```
sort -k 3,3n maize_chrom1.txt > maize_chrom1_sorted.txt
```
then replaced the missing values with a ? with-
```
's/ /?/g' maize_chrom_1.txt > maize_chrom_1_replaced.txt
```
All this piped out looks like this-
```
awk '$2 == "1"' snp_maize_3.txt | sort -k 3,3n | sed 's/ /?/g' > teosinte_chrom_1_replaced.txt
```
The only change for the each number is replacing the "1" with "n" for what ever chromosome you want to grab and changing then output file name.

For the sencond folder of 10 with the missing data encoded with a - and in decreasing order
```
awk '$2 == "1"' snp_maize_3.txt | sort -k 3,3nr | sed 's/ /-/g' > maize_chrom1_de.txt
```
and finally for the unknown and multi

```
'$2 == "unknown"' snp_maize_3.txt > maize_unknown.txt
'$2 == "multiple"' snp_maize_3.txt > teosinte_multiple.txt
```
## Teosinte Data

I did everything the same for the teosinte data as I did for the maize group. Most of the code I used is in the above section, but I'll add some here for good measure.

```
awk '$2 == "1"' snp_teosinte_3.txt | sort -k 3,3nr | sed 's/ /-/g' > teosinte_chrom1_de.txt
awk '$2 == "1"' snp_teosinte_3.txt | sort -k 3,3nr | sed 's/ /-/g' > teosinte_chrom1_de.txt
awk '$2 == "unknown"' snp_teosinte_3.txt > teosinte_unknown.txt
awk '$2 == "multiple"' snp_teosinte_3.txt > teosinte_multi.txt
