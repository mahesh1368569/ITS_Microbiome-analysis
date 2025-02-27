1.	Import data. (# the folder should only have the fastq files . no other files should be there)

qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path ponto-manifest.tsv \
  --output-path paired-end-demux.qza \
  --input-format PairedEndFastqManifestPhred33V2
2.	Summary

qiime demux summarize \
  --i-data  paired-end-demux.qza \
  --o-visualization  demux-paired-end.qzv

3.	Trimming ITS samples with Q2-ITSxpress for Dada2 (It took 34 hrs on 10 nodes- Shadow cluter)
 

qiime itsxpress trim-pair-output-unmerged\
  --i-per-sample-sequences sequences.qza \
  --p-region ITS2 \
  --p-taxa F \
  --p-cluster-id 1.0 \
  --p-threads 2 \
  --o-trimmed trimmed_exact.qza
4. Use Dada2 to identify sequence variants

qiime dada2 denoise-paired \
  --i-demultiplexed-seqs trimmed.qza \
  --p-trunc-len-r 0 \
  --p-trunc-len-f 0 \
  --output-dir dada2out

qiime feature-table summarize \
  --i-table dada2out/table.qza \
  --o-visualization tableviz.qzv

5. Download reference data from UNITE for fungal classification

https://unite.ut.ee/repository.php

Import the latest UNITE data into QIIME 2:
qiime tools import \
  --type 'FeatureData[Sequence]' \
  --input-path sh_qiime_release_10.05.2021/sh_refs_qiime_ver8_dynamic_10.05.2021.fasta \
  --output-path unite.qza
Import the associated UNITE taxonomy file.
qiime tools import \
  --type 'FeatureData[Taxonomy]' \
  --input-format HeaderlessTSVTaxonomyFormat \
  --input-path sh_qiime_release_10.05.2021/sh_taxonomy_qiime_ver8_dynamic_10.05.2021.txt \
  --output-path unite-taxonomy.qza




Train the QIIME classifier
qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads unite.qza \
  --i-reference-taxonomy unite-taxonomy.qza \
  --o-classifier classifier.qza

Classify the sequence variants
qiime feature-classifier classify-sklearn \
  --i-classifier classifier.qza \
  --i-reads dada2out/representative_sequences.qza \
  --o-classification taxonomy.qza
Summarize the results
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv


Create an interactive bar plot figure

qiime taxa barplot \
  --i-table dada2out/table.qza  \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file ponto-metadata.tsv  \
  --o-visualization taxa-bar-plots.qzv

6. Alpha and beta diversity analysis

qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences dada2out/representative_sequences.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza

qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table dada2out/table.qza \
  --p-sampling-depth 27000 \
  --m-metadata-file ponto-metadata.tsv \
  --output-dir core-metrics-results


qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/faith_pd_vector.qza \
  --m-metadata-file ponto-metadata.tsv \
  --o-visualization core-metrics-results/faith-pd-group-significance.qzv
  
qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/evenness_vector.qza \
  --m-metadata-file ponto-metadata.tsv \
  --o-visualization core-metrics-results/evenness-group-significance.qzv

Microbiome Analyst

Creating a BIOM table with taxonomy annotations
qiime tools export \
--input-path table.qza \
--output-path exported
qiime tools export \
--input-path taxonomy.qza \
--output-path exported
Change the first line of taxonomy.tsv (i.e. the header) to this:
#OTUID	taxonomy	confidence

** make sure the tab spacing is  not lost when making the above change

biom add-metadata \
-i feature-table.biom \
-o table-with-taxonomy.biom \
--observation-metadata-fp taxonomy.tsv \
--sc-separated taxonomy 

Relative Abundance for PAST – RDA

1.	create a feature table that has taxonomy instead of feature ID
qiime taxa collapse \
  --i-table table.qza \
  --i-taxonomy taxonomy_its.qza \
  --p-level 2 \
  --o-collapsed-table phyla-table.qza


2.	convert this new frequency table to relative-frequency

qiime feature-table relative-frequency \
--i-table phyla-table.qza \
--o-relative-frequency-table rel-phyla-table.qza

3.	Export

qiime tools export \
  --input-path rel-phyla-table.qza \
  --output-path exported-tree

4.	Convert biom to txt
biom convert \
 -i feature-table.biom \
-o rel-phyla-table.tsv \
--to-tsv
