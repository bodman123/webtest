.. highlight:: rst

.. _custom-signature:

Signature Matrix Creation
+++++++++++++++++++++++++

.. function:: batch_correct_datasets (data_loc, cell_file_dict, outfile="batch_corrected_datasets.txt")

   Performs batch correction on raw data files, and saves the combined output to a single CSV file. All files must be stored in the same folder.

   :param data_loc: String. Path to folder that holds all data files
   :param cell_file_dict: Dictionary. Keys = Cell Types, Values = paths to all raw files containing data for the given cell type
   :param outfile: String. Name to save the batch corrected datasets file under. Default is "batch_corrected_datasets.txt"

   :return: None. Batch corrected datasets are written to a text file



.. function:: create_signature_matrix (infile, cell_types, clustered=False, max_clusters=10, intermfile=None, outfile="kmeans_signature_matrix_qval.txt")

   Given a batch corrected dataset as input (such as one created with ``batch_correct_datasets()``), runs clustering on each cell-type (using the silhouette method), does differential expression analysis to get the genes that are significantly expressed among each cluster (using adjusted p value), and outputs a signature matrix to a csv file.

   :param infile: String. Path to text file containing the batch corrected datasets
   :param cell_types: List of cell types (strings) to include in signature matrix
   :param clustered: Boolean. Whether or not the infile is pre-clustered
   :param max_clusters: Integer. Maximum value of K to try for k-means clustering. Ignored if ``clustered=True``
   :param intermfile: String. Where to save intermediate step of clustered batch corrected datasets. Ignored if ``clustered=True``. Default is ``infile+"_clustered.txt"``
   :param outfile: String. Name to save the finished signature matrix file under. Default is "kmeans_signature_matrix_qval.txt"

   :return: None. Signature matrix and (if ``clustered=False`` & ``intermfile != None``) text file of clustered batch corrected datasets are written to csv files


**Example:** Create a custom signature matrix from raw data files. The data used in this example are single cell profiles from the Gene Expression Omnibus (GEO) repository. They are available to download on the NCBI website, as well as from our GitHub page, at: https://github.com/ShahriyariLab/TumorDecon/tree/master/TumorDecon/data/sig_matrix_tutorial :

To begin, we define a dictionary where the keys are the cell types we wish to consider, and the values are all the raw files that contain data for that cell type: ::

   >>> cell_file_dict = {'CD4': ['GSE107011_Processed_data_TPM2.txt', 'ensembl_version_GSE97861.txt', 'ensembl_version_GSE97862.txt', \
                                 'ensembl_version_GSE113891.txt', 'ensembl_version_GSE114407.txt', 'ensembl_version_GSE115978.txt'], \
                                 'CD8': ['ensembl_version_GSE98638.txt', 'ensembl_version_GSE114407.txt', 'GSE107011_Processed_data_TPM2.txt'], \
                                 'B_': ['GSE107011_Processed_data_TPM2.txt', 'ensembl_version_GSE114407.txt', 'ensembl_version_GSE115978.txt'], \
                                 'mono': ['ensembl_version_GSE114407.txt', 'GSE107011_Processed_data_TPM2.txt'], \
                                 'NK': ['GSE107011_Processed_data_TPM2.txt', 'ensembl_version_GSE115978.txt'], \
                                 'Endo': ['ensembl_version_GSE102767.txt', 'ensembl_version_GSE113839.txt', 'ensembl_version_GSE115978.txt'], \
                                 'Fibro': ['ensembl_version_GSE113839.txt', 'GSE109448_rnaseq_gene_tpm.tsv', 'GSE109449_singlecell_rnaseq_gene_tpm.txt'], \
                                 'Neutro': ['GSE107011_Processed_data_TPM2.txt']}

Next, remove batch effects with the ``batch_correct_datasets()`` function. In the following example, we have saved all the GEO files to "datafolder" and we wish to save the batch corrected data under the name "batch_corrected_sample_data.txt": ::

  >>> td.batch_correct_datasets("datafolder", cell_file_dict, outfile="batch_corrected_sample_data.txt")
  Performing batch correction...
  Found 6 batches.
  Adjusting for 0 covariate(s) or covariate level(s).
  Standardizing Data across genes.
  Fitting L/S model and finding priors.
  Finding parametric adjustments.
  Adjusting the Data
  Found 3 batches.
  Adjusting for 0 covariate(s) or covariate level(s).
  Standardizing Data across genes.
  Fitting L/S model and finding priors.
  Finding parametric adjustments.
  Adjusting the Data

  [ ... output truncated for space ... ]

  Found 3 batches.
  Adjusting for 0 covariate(s) or covariate level(s).
  Standardizing Data across genes.
  Fitting L/S model and finding priors.
  Finding parametric adjustments.
  Adjusting the Data
  Saving batch corrected datasets to batch_corrected_sample_data.txt...

Once batch effects are removed, use the data to create a signature matrix: ::

  >>> cell_types_to_include = ["CD8", "CD4", "B_cell", "NK", "mono", "Endothelial", "Fibroblast", "Neutrophils"]
  >>> td.create_signature_matrix("batch_corrected_sample_data.txt", cell_types_to_include, outfile="kmeans_signature_matrix_qval.txt")
  Reading batch-corrected dataset file batch_corrected_sample_data.txt...
  Clustering batch-corrected datasets...
  CD8: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  CD4: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  B_cell: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  NK: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  mono: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  Endothelial: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  Fibroblast: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  Neutrophils: Finding optimal K for K-means (max # of clusters = 10)
  Optimal K = 2
  Saving clustered data sets to batch_corrected_sample_data_clustered.txt...
  Generating signature matrix from clustered batch-corrected datasets...
  Saving signature matrix to kmeans_signature_matrix_qval.txt...

Note that the ``create_signature_matrix()`` function expects a non-clustered data file as input. However, if your datafile is large, running these two processes (clustering & generation of signature matrix) in sequence can take a very long time, or you may run into memory issues. To avoid this, you can also input an already-clustered data file (saved as an intermediate step in a previous run), and include the argument ``clustered=True``, in order to skip the clustering step: ::

  >>> td.create_signature_matrix("batch_corrected_sample_data_clustered.txt", cell_types_to_include, clustered=True, outfile="kmeans_signature_matrix_qval_2.txt")
  Reading batch-corrected dataset file batch_corrected_sample_data_clustered.txt...
  Generating signature matrix from clustered batch-corrected datasets...
  Saving signature matrix to kmeans_signature_matrix_qval_2.txt...
