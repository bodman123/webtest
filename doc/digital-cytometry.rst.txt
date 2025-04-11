.. highlight:: rst

Running Digital Cytometry
+++++++++++++++++++++++++

All digital cytometry in TumorDecon is done with the ``tumor_deconvolve()`` function:

.. function:: tumor_deconvolve (mixture_data, method, patient_IDs='ALL', sig_matrix=None, up_genes=None, down_genes=None, args={})

   Given a digital cytometry method, cell signature, and a mixture (RNA Sequence) dataset, return the estimated frequencies/ranks of each cell type in each patient.

   :param mixture_data: Pandas DataFrame of rna gene expression data. Rows are genes (indexed by 'Hugo_Symbol') and columns are patients
   :param method: Method for tumor deconvolution. Must be either 'CIBERSORT', 'DeconRNASeq', 'ssGSEA', or 'SingScore'
   :param patient_IDs: Either a list of patient IDs to run digital cytometry on, or the string 'ALL' (to run for all patients). The default is 'ALL'
   :param sig_matrix: Pandas DataFrame of signature gene expression values for given cell types. Rows are genes (indexed by 'Hugo_Symbol') and columns are cell types *** Required for 'CIBERSORT' and 'DeconRNASeq' methods *** (ignored for 'ssGSEA' and 'SingScore' methods)
   :param up_genes: Dictionary. Keys are cell types, vaues are a list of up-regulated genes (given by Hugo Symbol) for that cell type *** Required for 'ssGSEA' and 'SingScore' methods *** (ignored for 'CIBERSORT' and 'DeconRNASeq' methods)
   :param down_genes: Dictionary. Keys are cell types, vaues are a list of down-regulated genes (given by Hugo Symbol) for that cell type *** Optional for 'SingScore' method *** (ignored for 'CIBERSORT', 'DeconRNASeq', and 'ssGSEA' methods)
   :param args: Dictionary. Additional arguments to pass on to the given method. For a list of valid arguments for each method, see the subsections below.

   :return: Pandas DataFrame. Frequencies/scores for each specified patient and cell type in the signature matrix, as solved for using the specified method



**Example 1:** Run digital cytometry on all patients in the RNA mixture data, using the cibersort method, and save the output to a csv file.

Since CiberSort is a linear method, we need to pass in a signature matrix for the ``cell_signature`` argument, in the form of a Pandas DataFrame. Using the ``read_sig_file()`` function with no argument will read in LM22 by default. ::

  >>> rna = td.read_rna_file('uvm_tcga_pan_can_atlas_2018/data_RNA_Seq_v2_expression_median.txt')
  >>> sig = td.read_sig_file()
  >>> freqs = td.tumor_deconvolve(rna, 'cibersort', sig_matrix=sig)
  Running CiberSort...
  CiberSort has completed!
  >>> freqs.to_csv("cibersort_results.csv", index_label='Patient_ID')

**Example 2:** Run digital cytometry on just a subset of patients in the RNA mixture data, using the ssGSEA method. Then save output to a cvs file:

Since ssGSEA is a rank-based method, we need to pass in a list for the ``up_genes`` argument. Using the ``read_geneset()`` function with no argument will read in the default up-regulated gene set (derived from single-cell reference profiles of LM22, in Le et al. (2020)). We also define the list of patients we want to calculate scores for in a variable named ``patient_subset``::

  >>> rna = td.read_rna_file('uvm_tcga_pan_can_atlas_2018/data_RNA_Seq_v2_expression_median.txt')
  >>> up_reg_genes = td.read_geneset()
  >>> patient_subset = ['TCGA-RZ-AB0B-01', 'TCGA-V3-A9ZX-01', 'TCGA-V3-A9ZY-01', 'TCGA-V4-A9E5-01', \
                        'TCGA-V4-A9E7-01', 'TCGA-V4-A9E8-01', 'TCGA-V4-A9E9-01', 'TCGA-V4-A9EA-01', \
                        'TCGA-V4-A9EC-01', 'TCGA-V4-A9ED-01',	'TCGA-V4-A9EE-01', 'TCGA-V4-A9EF-01']
  >>> ssgsea_scores = td.tumor_deconvolve(rna, 'ssGSEA',  patient_IDs=patient_subset, up_genes=up_reg_genes, args={'print_progress':True})
  Running ssGSEA...
  TCGA-RZ-AB0B-01
  TCGA-V3-A9ZX-01
  TCGA-V3-A9ZY-01
  TCGA-V4-A9E5-01
  TCGA-V4-A9E7-01
  TCGA-V4-A9E8-01
  TCGA-V4-A9E9-01
  TCGA-V4-A9EA-01
  TCGA-V4-A9EC-01
  TCGA-V4-A9ED-01
  TCGA-V4-A9EE-01
  TCGA-V4-A9EF-01
  ssGSEA has completed!
  >>> ssgsea_scores.to_csv("ssgsea_results.csv", index_label='Patient_ID')



Cibersort
*********

The Cibersort method uses NuSVR from sklearn to solve for the cell type frequencies. Since Cibersort is a linear method, the ``sig_matrix`` argument is required. If a value for ``up_genes`` is also declared, it will be ignored. When using this method, the following additional arguments may be passed to the function via the ``args`` Dictionary:

  - **print_progress**: Boolean, whether to print patient ID as cibersort iterates through. Default is False
  - **scaling**: String, must be either 'None', 'zscore', or 'minmax'. Determines how to scale the mixture data and signature matrix before applying CIBERSORT. Default is MinMax scaling
  - **scaling_axis**: 0 or 1. Whether to scale mixture data and signature matrix by normalizing each column (patient/celltype) individually (scaling_axis=0) or each row (gene) individually (scaling_axis=1). Default is 0.
  - **nu**: see `sklearn's documentation for NuSVR <https://scikit-learn.org/stable/modules/generated/sklearn.svm.NuSVR.html>`_. If nu='best' (default), cibersort will use the nu in the set {0.25, 0.5, 0.75} that minimizes root mean square b/w data and prediction. If nu a float, will compute cibersort for that specific nu
  - C: see `sklearn's documentation for NuSVR <https://scikit-learn.org/stable/modules/generated/sklearn.svm.NuSVR.html>`_
  - kernel:: see `sklearn's documentation for NuSVR <https://scikit-learn.org/stable/modules/generated/sklearn.svm.NuSVR.html>`_
  - shrinking: see `sklearn's documentation for NuSVR <https://scikit-learn.org/stable/modules/generated/sklearn.svm.NuSVR.html>`_

The output for this method is a Pandas DataFrame (num_celltypes x num_patients) that contains cell type frequencies for each patient in the given 'patient_IDs' list. Rows are indexed by cell type, column headings are patient IDs

**Example 3:** Run CiberSort on all patients in the RNA mixture data, using the :ref:`custom signature matrix <custom-signature>` "kmeans_signature_matrix_qval.txt", and specifying an explicit scaling and value of ``nu``. ::

  >>> rna = td.read_rna_file('uvm_tcga_pan_can_atlas_2018/data_mrna_seq_v2_rsem.txt')
  >>> sig = td.read_sig_file("kmeans_signature_matrix_qval.txt", geneID='Ensembl_Gene_ID')
  >>> freqs = td.tumor_deconvolve(rna, 'cibersort', sig_matrix=sig, args={'nu':0.5, 'scaling':'minmax'})
  Running CiberSort...
  CiberSort has completed!
  >>>print(freqs)
  Patient_ID       CD8_subtype_1  CD8_subtype_2  CD4_subtype_1          ...            Fibroblast_subtype_2  Neutrophils_subtype_1  Neutrophils_subtype_2
  TCGA-RZ-AB0B-01       0.000000       0.340832       0.179790          ...                        0.012600               0.000000               0.000436
  TCGA-V3-A9ZX-01       0.028684       0.146691       0.152585          ...                        0.010037               0.000000               0.006936
  TCGA-V3-A9ZY-01       0.000000       0.488760       0.108457          ...                        0.001606               0.000000               0.000814
  TCGA-V4-A9E5-01       0.000000       0.399474       0.190793          ...                        0.003778               0.000000               0.000999
  TCGA-V4-A9E7-01       0.000000       0.455365       0.217109          ...                        0.000000               0.000228               0.000000
  ...                        ...            ...            ...          ...                              ...                   ...                    ...                    ...
  TCGA-YZ-A980-01       0.009343       0.284902       0.178078          ...                        0.015250               0.000000               0.000000
  TCGA-YZ-A982-01       0.000000       0.269736       0.150146          ...                        0.000000               0.000000               0.006947
  TCGA-YZ-A983-01       0.000000       0.592702       0.109028          ...                        0.000000               0.000000               0.002387
  TCGA-YZ-A984-01       0.000000       0.353790       0.140766          ...                        0.001984               0.011312               0.000000
  TCGA-YZ-A985-01       0.000000       0.408330       0.153663          ...                        0.000720               0.000000               0.003510

  [80 rows x 16 columns]

DeconRNASeq
***********

The DeconRNASeq method uses the Optimize method from SciPy to solve for the cell type frequencies. Since DeconRNASeq is a linear method, the ``sig_matrix`` argument is required. If a value for ``up_genes`` is also declared, it will be ignored. When using this method, the following additional arguments may be passed to the function via the ``args`` Dictionary:

  - **check_sig**: Boolean. Whether or not to check the condition number of the signature matrix before solving
  - **scaling**: String. Must be either 'None', 'zscore', or 'minmax'. Determines how to scale the signature matrix and mixture data before solving. Default is MinMax scaling.
  - **scaling_axis**: 0 or 1. Whether to scale mixture data and signature matrix by normalizing each column (celltype/patient) separately (scaling_axis=0) or each row (gene) separately (scaling_axis=1). Default is 0.
  - **formulation**: String. Must be either 'qp', 'ridge', or 'lasso'. Determines how to formulate the optimization problem:
          - **qp**: solve a QP (quadratic programming) problem min_x||Ax-b||^2 with strict condition x.sum()=1, xi>=0
          - **ridge**: solve a ridge regression problem min_x{||Ax-b||^2 + ||x||^2} with condition xi>=0
          - **lasso**: solve a lasso regression problem min_x{||Ax-b||^2 + Sum_i{|x_i|} with condition xi>=0
  - **reg_constant**: Float. Regularization constant for lasso/ridge regression. Ignored if formulation set to 'qp'
  - **print_progress**: Boolean. Whether or not to print the solver (scipy.optimize) results

The output for this method is a Pandas DataFrame (num_celltypes x num_patients) that contains cell type frequencies for each patient in the given 'patient_IDs' list. Rows are indexed by cell type, column headings are patient IDs

**Example 4:** Run DeconRNASeq on all patients in the RNA mixture data, using the default LM22 signature matrix. To evaluate our confidence in our solution, we explicitly check the condition number of the signature matrix, and print the solver results: ::

  >>> rna = td.read_rna_file('uvm_tcga_pan_can_atlas_2018/data_mrna_seq_v2_rsem.txt')
  >>> sig = td.read_sig_file()
  >>> freqs = td.tumor_deconvolve(rna, 'deconRNAseq', sig_matrix=sig, args={'check_sig':True, 'print_progress':True})
  Condition number of signature matrix = 12.43933600367927
  Running DeconRNASeq...

  TCGA-RZ-AB0B-01
     fun: 1.768742658372559
     jac: array([0.22816905, 0.2551588 , 0.20533609, 0.34062135, 0.22060739,
       0.20532408, 0.20530543, 0.20500609, 0.22956975, 0.32579079,
       0.21855029, 0.20513448, 0.20529102, 0.20514449, 0.2050613 ,
       0.32388473, 0.20581529, 0.20504086, 0.20532118, 0.2950649 ,
       0.20484114, 0.30095159])
  message: 'Optimization terminated successfully.'
     nfev: 243
      nit: 10
     njev: 10
   status: 0
  success: True
       x: array([4.79865365e-18, 0.00000000e+00, 3.24670795e-01, 4.81395017e-17,
       3.27398518e-17, 1.61659557e-02, 5.85440056e-02, 5.81593582e-02,
       9.57879942e-18, 4.25466532e-17, 4.63744135e-18, 3.15186488e-02,
       7.94907975e-02, 2.76273695e-01, 7.08312745e-04, 7.96339326e-17,
       0.00000000e+00, 6.24736273e-02, 8.95223995e-02, 1.22308564e-17,
       2.47240499e-03, 1.70412859e-17])


  TCGA-V3-A9ZX-01
    fun: 2.3380210562625794
    jac: array([0.08302617, 0.07165793, 0.07166326, 0.09838021, 0.11366048,
      0.07198009, 0.07167557, 0.07217818, 0.07183281, 0.09873474,
      0.07061729, 0.08603653, 0.08129218, 0.07147855, 0.07210144,
      0.07260156, 0.07217726, 0.07224679, 0.07091093, 0.16353929,
      0.12585488, 0.20857742])
  message: 'Optimization terminated successfully.'
     nfev: 242
      nit: 10
     njev: 10
   status: 0
  success: True
      x: array([0.00000000e+00, 5.13881657e-03, 2.38885934e-01, 0.00000000e+00,
      0.00000000e+00, 2.61113903e-02, 1.20781286e-02, 1.09609396e-02,
      6.60321613e-02, 1.75537745e-17, 2.11868602e-02, 6.54156460e-18,
      4.40696071e-18, 3.96846445e-01, 5.41708212e-02, 0.00000000e+00,
      1.21155050e-01, 6.53890726e-03, 4.08945470e-02, 1.69936550e-17,
      1.54190292e-18, 2.90338837e-17])

  [ ... output truncated for space ... ]

  DeconRNASeq has completed!

  >>> print(freqs)
  Patient_ID       B cells naive  B cells memory  Plasma cells   T cells CD8      ...       Mast cells resting  Mast cells activated   Eosinophils   Neutrophils
  TCGA-RZ-AB0B-01   4.798654e-18    0.000000e+00      0.324671  4.813950e-17      ...             8.952240e-02          1.223086e-17  2.472405e-03  1.704129e-17
  TCGA-V3-A9ZX-01   0.000000e+00    5.138817e-03      0.238886  0.000000e+00      ...             4.089455e-02          1.699366e-17  1.541903e-18  2.903388e-17
  TCGA-V3-A9ZY-01   0.000000e+00    3.518049e-17      0.283670  0.000000e+00      ...             6.147303e-02          0.000000e+00  2.351062e-18  0.000000e+00
  TCGA-V4-A9E5-01   0.000000e+00    0.000000e+00      0.258718  2.794810e-17      ...             7.591108e-02          0.000000e+00  3.969250e-18  1.350584e-17
  TCGA-V4-A9E7-01   1.937510e-17    1.667966e-03      0.258230  0.000000e+00      ...             6.860309e-02          7.706740e-18  4.057952e-18  2.948868e-17
  ...                        ...             ...           ...           ...      ...                      ...                   ...           ...           ...
  TCGA-YZ-A980-01   2.581715e-17    0.000000e+00      0.272212  5.280067e-18      ...             6.036439e-02          0.000000e+00  0.000000e+00  0.000000e+00
  TCGA-YZ-A982-01   1.078221e-17    3.272811e-18      0.283501  6.271637e-17      ...             8.536993e-02          0.000000e+00  0.000000e+00  4.242467e-17
  TCGA-YZ-A983-01   4.291653e-03    0.000000e+00      0.320183  0.000000e+00      ...             8.946226e-02          5.102526e-18  9.125998e-03  0.000000e+00
  TCGA-YZ-A984-01   1.928266e-17    2.356858e-03      0.258814  5.212959e-18      ...             6.631310e-02          3.597047e-17  7.543348e-18  0.000000e+00
  TCGA-YZ-A985-01   0.000000e+00    1.442125e-02      0.309380  9.133074e-17      ...             5.799046e-02          2.838456e-17  0.000000e+00  0.000000e+00

  [80 rows x 22 columns]


ssGSEA
******

Since ssGSEA is a rank-based method, the ``up_genes`` argument is required. If a value for ``sig_matrix`` is also declared, it will be ignored. When using this method, the following additional arguments may be passed to the function via the ``args`` Dictionary:

  - **print_progress**: Boolean. Whether to print patient ID as ssGSEA iterates through. Default is False
  - **alpha**: Float. Weight used in ssGSEA method. Default is 1.0
  - **ties_method**: Describes how to treat ties in rankings. See pandas.DataFrame.rank() for list of valid options.
  - **norm**: Boolean. Whether to normalize enrichment scores by using the entire data set, as indicated by Barbie et al., 2009, online methods, pg. 2. Default is False.

The output for this method is a Pandas DataFrame (num_celltypes x num_patients) that contains ssGSEA enrichment scores for each cell type declared in 'up_genes' dictionary, for each patient in the given 'patient_IDs' list. Rows are indexed by cell type, column headings are patient IDs

**Example 5:** Run ssGSEA on 5 individual patients in the RNA mixture data, using the default gene set derived from LM22 and alpha = 0.5, and then normalize the results ::

  >>> rna = td.read_rna_file('uvm_tcga_pan_can_atlas_2018/data_mrna_seq_v2_rsem.txt')
  >>> up_reg_genes = td.read_geneset()
  >>> patient_subset = ['TCGA-RZ-AB0B-01', 'TCGA-V3-A9ZX-01', 'TCGA-V3-A9ZY-01', 'TCGA-V4-A9E5-01', 'TCGA-V4-A9E7-01', 'TCGA-V4-A9E8-01', 'TCGA-V4-A9E9-01', 'TCGA-V4-A9EA-01']
  >>> ssgsea_scores = td.tumor_deconvolve(rna, 'ssGSEA',  patient_IDs=patient_subset, up_genes=up_reg_genes, args={'alpha':0.5, 'norm':True})
  Running ssGSEA...
  ssGSEA has completed!
  >>>> print(ssgsea_scores)
  Patient_ID       B cells naive  B cells memory  Plasma cells  T cells CD8     ...       Mast cells resting  Mast cells activated  Eosinophils  Neutrophils
  TCGA-RZ-AB0B-01       0.327240        0.249076      0.986881     0.326747     ...                 0.735592              0.490964     0.558033     0.435243
  TCGA-V3-A9ZX-01       0.276838        0.220086      0.982511     0.405832     ...                 0.681473              0.527347     0.565607     0.481490
  TCGA-V3-A9ZY-01       0.291156        0.239979      0.898351     0.489769     ...                 0.646602              0.451504     0.532254     0.477449
  TCGA-V4-A9E5-01       0.279829        0.238470      0.888191     0.521628     ...                 0.619921              0.447962     0.506716     0.467605
  TCGA-V4-A9E7-01       0.318732        0.155308      0.946145     0.452830     ...                 0.671831              0.468233     0.485592     0.453880
  TCGA-V4-A9E8-01       0.263715        0.196422      0.955722     0.547990     ...                 0.608545              0.456964     0.473649     0.514065
  TCGA-V4-A9E9-01       0.316531        0.217946      0.912949     0.542386     ...                 0.585311              0.455807     0.461873     0.447807
  TCGA-V4-A9EA-01       0.382925        0.229779      0.855658     0.572038     ...                 0.583329              0.438876     0.458896     0.458504

  [8 rows x 22 columns]


SingScore
*********

Since SingScore is a rank-based method, the ``up_genes`` argument is required. If a value for ``sig_matrix`` is also declared, it will be ignored.

If both the ``down_genes`` and ``up_genes`` arguments are defined, the ``tumor_deconvolve()`` function will run PySingScore's implementation of bidirectional singscore. If only ``up_genes`` is defined, it will run PySingScore's implementation of unidirectional singscore.

The output for this method is a Pandas DataFrame (num_celltypes x num_patients) that contains SingScore scores for each cell type declared in 'up_genes' dictionary, for each patient in the given 'patient_IDs' list. Rows are indexed by cell type, column headings are patient IDs

**Example 6:** Run both unidirectional and bidirectional SingScore using the default Gene Sets. To derive a down-regulated gene set, we use the ``td.find_up_down_genes_from_sig()`` function (see :ref:`Creating Gene Sets <creating-gene-set>`). We will create this gene set from the LM6 signature matrix, which is included in the package: ::

  >>> rna = td.read_rna_file('uvm_tcga_pan_can_atlas_2018/data_mrna_seq_v2_rsem.txt')
  >>> LM6 = td.read_sig_file('LM6.txt')
  >>> up_genes_LM6, down_genes_LM6 = td.find_up_down_genes_from_sig(LM6, down_cutoff=0.4, up_cutoff=4.0)

  >>> singscore_unidirectional = td.tumor_deconvolve(rna, 'singscore', up_genes=up_genes_LM6)
  Running SingScore...
  SingScore has completed!
  >>> print(singscore_unidirectional)
                    B cells  CD8 T cells  CD4 T cells  NK cells  Monocytes  Neutrophils
  Patient_ID
  TCGA-RZ-AB0B-01 -0.040879    -0.042245    -0.015696 -0.068842  -0.004216    -0.067957
  TCGA-V3-A9ZX-01 -0.016157     0.006778     0.025657 -0.024595   0.040129    -0.035982
  TCGA-V3-A9ZY-01 -0.047875    -0.054082    -0.005818 -0.085125   0.007932    -0.061282
  TCGA-V4-A9E5-01 -0.053520    -0.070746    -0.033882 -0.102350   0.001286    -0.067848
  TCGA-V4-A9E7-01 -0.077335    -0.070958    -0.043286 -0.098458  -0.035024    -0.090904
  ...                   ...          ...          ...       ...        ...          ...
  TCGA-YZ-A980-01 -0.027205    -0.025515     0.020872 -0.041668   0.000913    -0.065293
  TCGA-YZ-A982-01 -0.061571    -0.051486    -0.021256 -0.073456   0.018704    -0.058435
  TCGA-YZ-A983-01 -0.058798    -0.065018    -0.035282 -0.082523  -0.006307    -0.065574
  TCGA-YZ-A984-01 -0.068884    -0.046064    -0.007893 -0.079870  -0.008476    -0.075183
  TCGA-YZ-A985-01 -0.072839    -0.054141    -0.018756 -0.089481  -0.011167    -0.081205

  [80 rows x 6 columns]

  >>> singscore_bidirectional = td.tumor_deconvolve(rna, 'singscore', up_genes=up_genes_LM6, down_genes=down_genes_LM6)
  Running SingScore...
  SingScore has completed!
  >>> print(singscore_bidirectional)
                    B cells  CD8 T cells  CD4 T cells  NK cells  Monocytes  Neutrophils
  Patient_ID
  TCGA-RZ-AB0B-01  0.478281     0.489102     0.489548  0.485272   0.489559     0.484873
  TCGA-V3-A9ZX-01  0.488396     0.496139     0.495398  0.494335   0.491098     0.485794
  TCGA-V3-A9ZY-01  0.463718     0.468867     0.474098  0.466825   0.482648     0.472018
  TCGA-V4-A9E5-01  0.459967     0.460788     0.466172  0.457230   0.478536     0.463606
  TCGA-V4-A9E7-01  0.457835     0.465507     0.465669  0.463028   0.479901     0.471592
  ...                   ...          ...          ...       ...        ...          ...
  TCGA-YZ-A980-01  0.485412     0.491924     0.493563  0.491461   0.495019     0.491583
  TCGA-YZ-A982-01  0.455890     0.470598     0.471025  0.475132   0.484155     0.469897
  TCGA-YZ-A983-01  0.470838     0.479412     0.478998  0.482687   0.488062     0.481390
  TCGA-YZ-A984-01  0.460155     0.471427     0.473025  0.463111   0.480844     0.466355
  TCGA-YZ-A985-01  0.443784     0.463111     0.467764  0.459697   0.470359     0.456920

  [80 rows x 6 columns]
