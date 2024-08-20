# Protein-Abundance-Prediction

This is a random forest machine learning model that can be used to predict protein abundance. There are 6 required pieces of information: 1. mRNA sequences (.fa.gz format), 2. 3' and 5' UTR regions (.bed file), 3. RNA abundance (.csv file), 4. Translation information (.csv file), 5. Protein abundance (.csv file), and 6. File matching cell line to its experimental alias (.csv file).

## Notes on Use

This is a Quarto script (.qmd), so I would recommend downloading the file and opening it on RStudios for best ease of use. The code is split into several chunks, and you are able to run each chunk individually. However, you can also run all chunks at once using the ‘Run All’ option in the ‘Run’ drop down menu. Just be sure that all the files are in the correct format and named correctly as described in the NOTES found within the Load Libraries and Data section of the script. In all of the test runs I have done, I have not encountered an error in getting the script to work as long as all the needed information is provided. This code should take around 3.5 hours to execute.

## Clean Data

I structured the code so that all of the data matches the gene order as found in the RNA abundance file. Making sure that all data sets are in the same order in respect to genes is imperative to ensuring the correct binding of columns when all data sets are merged to feed to the random forest model for prediction.

In the case of a gene having several CDSs, the program will only keep the largest sequence. Each kept CDS is then fragmented into its respective codons.

The same logic was applied if there is a gene with more than one 5’ UTR. If there are genes without 5' information, these genes have a default 5’ UTR of sequence ‘NNN.’ Using the 5’ UTR, I extracted information related to the Kozak sequence and GC content. For Kozak sequences, I performed a local alignment between the 2 ideal Kozak sequences and the 5’ UTR, and I kept the highest of the 2 alignment scores for each gene. For GC abundance, I simply calculated the proportion of nucleotides G and C in the 5’ UTR for each gene.

## Normalize Data, Separate by Cell Line, Compile Data Sets by Cell Line

Codon counts: codon counts are normalized by finding the relative frequency of each codon with each gene’s CDS. This would decrease the bias that may be introduced by larger CDSs having more codons in general than smaller CDSs.

Kozak alignment scores: alignment scores of the Kozak sequences were normalized using Z-standardization. Z-standardization allows the learning model to make its predictions based on the number of standard deviations each alignment score is away from the mean.

GC Content: since the content was already reported in a translatable metric of proportion, no further normalization is performed.

RNA Abundance and Translation Information: both measures are normalized using counts per million followed by log +1 transformation. This log transformation was meant to deal with the skewness of the counts per million distribution. +1 was to avoid any computations with log(0), which would return an error.

After normalization, each normalized data set is subsetted by cell line, as identified by experimental aliases. After subsetting, codon counts, alignment scores, RNA abundance, translation information, and protein abundance were all compiled, with each cell line receiving their own data set.

## Cross-Validation and Random Forest Model Generation for Each Cell Line

10-fold cross validation is performed, and a random forest model with 100 trees is generated. 

## Implementation and Results

The test indices from each fold were used to run each model 10 times per cell line (each run testing a different fold). The results from each fold were combined either into a common data frame of predicted and actual protein abundances or mean values of RMSE, MAE, and R2. This way, each cell line could have one scatterplot, RMSE, MAE, and R2 encompassing the results from all genes.

