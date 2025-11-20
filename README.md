# noaa-afsc-pinksalmon_herring_stomachs

1. created a single folder with raw sequence data (from 2023 and 2025 library prep and sequencing runs)

2. working in the terminal, remove primers from demultiplexed amplicon reads using cutadapt 
- cd fastq
- conda activate cutadptenv
- DATA=/Users/kimberly.ledger/Documents/noaa-afsc-pinksalmon_herring_stomachs/fastq
- NAMELIST=$(ls ${DATA} | sed 's/e*_L001.*//' | uniq)
- echo "${NAMELIST}"
- mkdir trimmed
- for i in ${NAMELIST}; do
   cutadapt --discard-untrimmed -g GCCGGTAAAACTCGTGCCAGC -G CATAGTGGGGTATCTAATCCCAGTTTG -o trimmed/${i}_R1.fastq.gz -p trimmed/${i}_R2.fastq.gz "$DATA/${i}_L001_R1_001.fastq.gz" "$DATA/${i}_L001_R2_001.fastq.gz";
done
- pigz -d trimmed/*.gz
- cd trimmed 
- mkdir filtered
- mkdir filtered/outputs

3. process reads using DADA2 by running '1_sequence_filtering.Rmd'

4. filter ASVs using LULU by running "2_lulu.Rmd"

5. perform taxonomic assignment using MIDORI2 database 

note: i previously made a blastdb for the metaprobe project, so just going to point to that folder... but if needing to create a blastdb, run the following:
makeblastdb -in MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST.fasta \
            -dbtype nucl \
            -out MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db

blastn -query outputs/1_sequence_filtering/myasvs.fasta \
       -db /Users/kimberly.ledger/Documents/metaprobes_BeringSea2024/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db \
       -out outputs/3_blastn_taxonomy/blastnresults \
       -perc_identity 96 -qcov_hsp_perc 100 \
       -outfmt "6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore sscinames staxids" \
       -num_threads 10
       
6. filter blastn results to finalize taxonomic assignments using "3_blastn_taxonomy.Rmd"

7. 