
## *UNIX Assignment**

**Data Inspection**

**Attributes of `fang_et_al_genotypes`**
```
$ du -h Fang_et_al_genotypes
$ column -s "," -t fang_et_al_genotypes.txt | (head -n 2; tail -n 2)| less
$ tail -n +3 fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
$ wc fang_et_al_genotypes. txt
```

By inspecting this file I learned that:

1. Using the du -h command I found out the file is 11M in size
2. Using the column -s command I was able to look at the header and the tail of file. I looked at the first two lines and the last two lines. I also piped this information to less
3. While lookin at the tail, I combines the awk command to print out the number of columns. I found out there were 986 columns in this documents
4. Using the wc command I found there are 2783 lines, 2744038 words and 11051939 characters/bytes in the file


**Attributes of `snp_position.txt`**

```
$ du -h snp_postion.txt
$ (head -n 3; tail -n 3) < snp_position.txt |less
$ tail -n +3 snp_position.txt | awk -F "\t" '{print NF; exit}'
$ wc snp_position.txt
```

By inspecting this file I learned that:

1. du -h command showed the file is 81k in size
2. While combining both head and tail command I was able to get glimpse of the files
3. Using the tail and awk command I found out there were 15 columns in the file. I found out that the columns I require for the next step are on columns 1, 3 and 4. 
4. Using the wc command I found that there are 984 lines, 13198 words and 82763 characters/bytes 

**Data Processing**

**Creating a maize and teosinte file**

```
$ cut -f 1,3,4 snp_position.txt > new_snp.txt
$ grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > fang_et_al_zea.txt
$ grep -E "Sample_ID" fang_et_al_genotypes.txt > Sample_ID.txt
$ cp Sample_ID.txt > Sample_ID_teosinte.txt
$ cat fang_et_al_zea.txt >> Sample_ID.txt  
$ awk -f transpose.awk > Sample_ID.txt > transposed_fang_zea2.txt
$ sed 's/Sample_ID/SNP_ID/' transposed_fang_zea2.txt | sort -k1,1 > sorted_fang_zea.txt
```
 1. Using the cut command I created a new file (new_snp.txt) containing with the 1st columns (SNP_ID), 3rd columns (Chromosome) and 4th columns (position).
 2. I created the maize file by using grep to select key words and and sent the standard out to fang_et_al_zea.txt. 
 3. To bring the headers headers into the new file, I used grep to copy the header into a new file Sample_ID.txt. I then duplicated this file, to use in teosinte. 
 4. I then used the cat command to append the maize genotypes into the file with headers. I then transposed using awk command
 5. I used the sed command to change the sample_ID name on the header to SNP_ID to match the header in the SNP file followed by sorting the file on the first column. 

**Teosinte**

```
$ grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > fang_et_teosinte.txt
$ cat fang_et_teosinte.txt >> Sample_ID_teosinte.txt|fang_et_teosinte2.txt
$ awk -f transpose.awk fang_et_teosinte2.txt  > transposed_fang_teosinte2.txt
$ sed 's/Sample_ID/SNP_ID/' transposed_fang_teosinte2.txt | sed 's/Sample_ID/SNP_ID/'>  transposed_fang_teosinte.txt

```
1. Using grep I cut I created a teosinte file based on the key words given
2. I used the cat command to append the  new file in the file with the genotype headers to create fang_et_teosinte2.txt
3. Using awk command I transposed the new file. 
4. With a combination of two command, I created a new file with the Sample_ID changed to SNP_ID using sed, sorted by the first column using sort. 


**Joining each file with the SNP file**
```
$ join -1 1 -2 1 -t $'\t' sorted_snp.txt sorted_fang_zea.txt > join_fang_zea.txt
$ join -1 1 -2 1 -t $'\t' sorted_snp.txt sorted_fang_teosinte.txt > join_fang_teosinte.txt
```
I joined each respective file with sorted_snp.txt using join command. The command joined the two based on the similarities on the first column in the sorted_snp.txt and first column of respective genotype file. The option -t $'\t' is to use tab as a field separator

**Extracting chromosomes for teosinte and mays files**
```
$ awk '$2 ~ /^1$/' all_chrom_maize.txt > chr1_maize.txt 
$ grep "unknown" join_fang_zea.txt > unknown_snp_maize.txt
$ grep "multiple" join_fang_zea.txt > multiple_pst_maize.txt
$ awk '$2 ~ /^1$/' join_fang_teosinte.txt  > chr1_teosinte.txt
grep "unknown" join_fang_teosinte.txt > unknown_snp_teosinte.txt
grep "multiple" join_fang_teosinte.txt > multiple_pst_teosinte.txt
```
1. I created 20 chromosome files for maize and teosinte. I used awk to select on the 2nd field and used (/^1$/') option to select everything that matches with that chromosome number.  
2. To conform that the files are what I was looking for, I checked the file sizes. I expected that size of each file should decrease in size from the first chromosome to the tent. This was confirmed.
3. I used grep to create files containing SNPs with unknown position and another with for SNPs with multiple positions 

**Sorting chromosome files**
```
$ sort -k3,3n maize_chr1.txt > maize_chr1_ascending.txt
$ sed 's/?/-/g' maize_chr1.txt | sort -k3,3nr > maize_chr1_descending.txt
$ sort -k3,3n chr1_teosinte.txt > chr1_teosinte_ascending.txt
$ sed 's/?/-/g' chr1_teosinte.txt | sort -k3,3nr > chr1_teosinte_descending.txt
```
1. Before sorting I used grep command to check for the missing data either '-' or '?' . The '-' was missing in all files. I used the sed command to change the '?' to '-' and piped to sorting based on the 3rd column. The files files are sorted based on position in reverse
2. I sorted also sorted the file with '?' for missing data on increasing position
