# Multi-Marker Metagenomic Assessment of Eukaryotic Communities in Najafgarh Drain Reveals Public Health and Ecological Risks
Example Kraken2 commands demonstrating the analysis workflow for detection of eukaryotic species from Whole Metagenome Sequences.

To enable accurate taxonomic classification while minimizing host contamination, we created two types of Kraken2-compatible databases: (i) a human genome database for removing human-derived reads, and (ii) custom clustered eukaryotic databases for downstream classification. We illustrate the workflow using sample ND1 as an example.

# Step 1. Building the Human Genome Database
The human genome was used to remove human contamination: (eg: human_genome_db)
```plaintext
kraken2-build --download-taxonomy --db human_genome_db
kraken2-build --download-library human --db human_genome_db
```

# Step 2. Building the Taxonomic Classification Databases
A clustered eukaryotic nr database was prepared for taxonomic classification: (eg: eukaryotic_db)  
Download NCBI taxonomy
```plaintext
kraken2-build --download-taxonomy --db eukaryotic_db
```

Add eukaryotic sequences
```plaintext
kraken2-build --add-to-library eukaryotic_nr.fasta --db eukaryotic_db
```

Build the database
```plaintext
kraken2-build --protein --threads 64 --db eukaryotic_db
```
The same procedure was followed to create a union database containing sequences from MIDORI2, COInr, SILVA, and plantITS, enabling broad eukaryotic taxonomic coverage. (eg: union_db)

As a result, two complementary Kraken2-compatible databases were generated:
1.	human_genome_db (for host read removal)
2.	eukaryotic_db & union_db (for taxonomic classification)

# Step 3. Removal of Human Reads
Raw paired-end reads were first mapped to the human genome database to filter human contamination:
```plaintext
kraken2 --db human_genome_db \
--threads 64 \
--paired raw_reads/ND1_1.fq raw_reads/ND1_2.fq > output_directory/result/ND1_result.txt \
--report output_directory/report/ND1_report.txt \
--unclassified-out output_directory/data/ND1_human_free_reads_#.fq
```

# Step 4. Classification Against the Clustered Eukaryotic Database
Reads free from human contamination were classified against the clustered eukaryotic nr database:
```plaintext
kraken2 --db eukaryotic_db \
--threads 64 \
--paired ND1_human_free_reads_1.fq ND1_human_free_reads_2.fq  > output/result/ND1_database1_result.txt \
--report output/report/ND1_database1_report.txt \
--unclassified-out output/data/ND1_unclassified_#.fq
```

# Step 5. Classification Against the Union Database
Unclassified reads from the previous step were further classified against the union database (MIDORI2, COInr, SILVA, plantITS):
```plaintext
kraken2 --db union_db \
--threads 64 \
--paired ND1_unclassified_1.fq ND1_unclassified_2.fq > output/result/ND1_database2_result.txt \
--report output/report/ND1_database2_report.txt \
```

# Step 6. Merging Reports
Finally, classification reports (ND1_database1_report.txt  and ND1_database2_report.txt) from Step 4 and Step 5 were merged using the combine_kreports.py script from KrakenTools (available at https://github.com/jenniferlu717/KrakenTools) for further analysis. 
