
# Edna_Unix Assignment
##Inspection of data files
Inside Unix Assignment folder there are 2 files to inspect

 fang_et_al_genotypes.txt 
 snp_position.txt  

##File sizes (fang_et_al_genotypes.txt and snp_position.txt)

	du -h fang_et_al_genotypes.txt snp_position.txt

	11M     fang_et_al_genotypes.txt
	84K     snp_position.txt

##Types of files
	file *
	fang_et_al_genotypes.txt: ASCII text, with very long lines
	snp_position.txt:ASCII text

##Perform a  word count using wc
	wc -l fang_et_al_genotypes.txt snp_position.txt
 
	2783 fang_et_al_genotypes.txt
 	984 snp_position.txt
 	3767 total

	or $ wc *

 	2783  2744038 11051939 fang_et_al_genotypes.txt
      
 	984    13198    82763 snp_position.txt. Lines, words and character respectively
      
##number of columns
	 awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt

	986

	 awk -F "\t" '{print NF; exit}' snp_position.txt

	15

###To Combine all the data inspection process the loop code "good" 

	 for filename in fang_et_al_genotypes.txt snp_position.txt ; do echo "N of lines at: $(wc -l 	$filename)"; echo  "N of columns at: $(awk -F "\t" '{print NF; exit}' $filename)"; echo "File 	size of: $(du -h $filename)"; done

	Returned 
	N of lines at: 2783 fang_et_al_genotypes.txt
	N of columns at: 986Ekmb2014
	File size of: 11M       fang_et_al_genotypes.txt
	N of lines at: 984 snp_position.txt
	N of columns at: 15
	File size of: 84K       snp_position.txt

##Data Processing
###Extract the files

	awk '$3 ~ /ZMMIL/ || $3 ~ /ZMMLR/ || $3 ~ /ZMMMR/' fang_et_al_genotypes.txt > maize_genotypes.txt

	 awk '$3 ~ /ZMPBA/ || $3 ~ /ZMPIL/ || $3 ~ /ZMPJA/' 	fang_et_al_genotypes.txt > teosinte_genotypes.txt

###check if all the data extracted is correct
	 awk -F "\t" '{print NF; exit}' maize_genotypes.txt teosinte_genotypes.txt
	
	986 columns for each file

	wc maize_genotypes.txt teosinte_genotypes.txt
      
	1573  1550978  6240114 maize_genotypes.txt
	975   961350  3873338 teosinte_genotypes.txt
	2548  2512328 10113452 total

### Extract the headers and create a new file for the headers and join these headers with the extracted files

	grep "Group" fang_et_al_genotypes.txt > header.txt 
	 cat header.txt maize_genotypes.txt > maize_genotypes_header.txt
 	 cat header.txt teosinte_genotypes.txt > teosinte_genotypes_header.txt

### Remove columns 1 and 2  in maize_genotypes.txt & teosinte_genotypes.txt

	 cut -f 3-968 maize_genotypes_header.txt  > filtered_maize_genotypes.txt

	  cut -f 3-968 teosinte_genotypes_header.txt > filtered_teosinte_genotypes.txt

###check with wc -l and awk -F "\t" '{print NF; exit}' filtered_teosinte_genotypes.txt

###Transpose the files
	
 awk -f transpose.awk filtered_teosinte_genotypes.txt > transposed_teosinte_genotypes.txt

 awk -f transpose.awk filtered_maize_genotypes.txt > transposed_maize_genotypes.txt

## Process the snp_position.txt by extracting the three most informative columns of data & removing the headers before sorting
	Cut -f 1,3,4 snp_position.txt > formatted_snp_positions.txt

 	grep -v "SNP_ID" formatted_snp_positions.txt > noheader_snp_positions.txt	

## remove also the headers in the transposed maize_genotypes.txt and transposed_teosinte.txt files to make it easier to join the files

	grep -v "Group" transposed_maize_genotypes.txt  > noheader_transposed_maize_genotypes.txt
	
	grep -v "Group" transposed_teosinte_genotypes.txt  > noheader_transposed_teosinte_genotypes.txt
  
##Sorting the first column of each of the three files by the common column
	sort -k1,1 noheader_snp_positions.txt > snp_position_sorted.txt
 	sort -k1,1 noheader_transposed_maize_genotypes.txt > maize_genotypes_sorted.txt
 	sort -k1,1 noheader_transposed_teosinte_genotypes.txt > teosinte_genotypes_sorted.txt

###checking if all well sorted well use echo $? 
	example code: sort -k1,1 -c snp_position_sorted.txt | echo $?)
	All returned 0

##Join the genotypes files with snp positions using the common column

	join -t $'\t' -1 1 -2 1 snp_position_sorted.txt teosinte_genotypes_sorted.txt > joined_teosinte_genotypes.txt

	join -t $'\t' -1 1 -2 1 snp_position_sorted.txt maize_genotypes_sorted.txt > 	joined_maize_genotypes.txt
	Inspections: awk -F "\t" '{print NF; exit}' joined_maize_genotypes.txt & wc *

##Separating data files based on chromosome:
### steps 
1. Replace missing data with ? 
 
	sed 's/<TAB>/?/g' joined_maize_genotypes.txt > sub_maize_genotypes.txt
 
	sed 's/<TAB>/?/g' joined_teosinte_genotypes.txt > sub_teosinte_genotypes.txt

2. Extracted SNPs for each  based on increasing positions

	awk '$2 == /1/ {print}' sub_maize_genotypes.txt | sort -k3,3n > maize_genotypes_chr1.tx
	
	awk '$2 ~/7/ {print}' sub_teosinte_genotypes.txt | sort -k3,3n > teosinte_genotypes_chr7.txt

3. Extracted SNPs for each  based on decreasing positions using a for loop

	for i in {1..10}; do awk '$2=='$i'' sub_maize_genotypes.txt | sed 's/?/-/g'| sort -k3,3nr | tee maize_genotypes_chr$i.txt; done
  
  	for i in {1..10}; do awk '$2=='$i'' sub_teosinte_genotypes.txt | sed 's/?/-/g'| sort -k3,3nr | tee teosinte_genotypes_chr$i.txt; 	done
4. Extracting SNPs with unknown and multiple  position

 	awk '$2 ~ /unknown/ {print}' sub_teosinte_genotypes.txt | sort -k3,3n > unknown_teosinte_genotypes.txt
 	
	awk '$2 ~ /multiple/ {print}' sub_teosinte_genotypes.txt | sort -k3,3n > multiple_teosinte_genotypes.txt
	
	awk '$2 ~ /multiple/ {print}' sub_maize_genotypes.txt | sort -k3,3nr > multiple_maize_genotypes.txt
	
	awk '$2 ~ /unknown/ {print}' sub_maize_genotypes.txt | sort -k3,3nr > unknown_maize_genotypes.txt

## my intermediary files are in a sub directory./Processed_files
## Individual chromosomes are in respective subdirectories
 












