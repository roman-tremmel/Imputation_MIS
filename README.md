# Imputation using the Michingan Imputation Server

## Preimputation

### Quality Control

    wget http://www.well.ox.ac.uk/~wrayner/tools/HRC-1000G-check-bim-v4.2.7.zip
    wget ftp://ngs.sanger.ac.uk/production/hrc/HRC.r1-1/HRC.r1-1.GRCh37.wgs.mac5.sites.tab.gz
    
    
### Convert ped/map to bed

    plink --file <input-file> --make-bed --out <output-file>

### Create a frequency file

    plink --freq --bfile <input> --out <freq-file>

### Execute script

    perl HRC-1000G-check-bim.pl -b <bim file> -f <freq-file> -r HRC.r1-1.GRCh37.wgs.mac5.sites.tab -h
    
Replace "make-bed" with "--recode vcf" in the script     
    
    # try good all sed -i 's/make-bed/--recode vcf/g'
    sh Run-plink.sh
    
 ### Sort using bcftools   
  
     ls -1 | grep vcf | awk {'print "bcftools sort " $1  " -Oz -o " $1 ".gz" '} > Run_bcf.sh 
     Run_bcf.sh
     
## Imputation 
https://imputationserver.sph.umich.edu/index.html

## Postimputation

### Filter R2>0.8

     ls -1 | grep dose | sed 's/vcf.gz//g' | awk {'print "bcftools view -i '\''R2>.8'\'' -Oz " $1"vcf.gz > " $1 "filtered.vcf.gz;tabix -p vcf " $1 "filtered.vcf.gz"'} > RUN_FILTER.sh 
     
optional filter for MAF

    bcftools view -i 'AF>.01' chr22.dose.filtered.vcf.gz | grep ^# -v | wc -l
     
### Number of variants

    zcat chr*.dose.filtered.vcf.gz | grep ^# -v | wc -l
    
    
### add rsID using snpSift/snpEff

java -jar -Xmx16024m SnpSift.jar annotate dbSnp132.vcf variants.vcf > variants_annotated.dvcf
     
