.. highlight:: rst

Quickstart
++++++++++

Installation
*************

To install the software with pip, use the following two commands: ::

       pip install TumorDecon
       pip install git+https://github.com/kristyhoran/singscore

To load the module into python, use:

.. code-block:: python

       import TumorDecon as td


Tutorial
*********

Here we provide a basic tutorial for importing a dataset and running each of the 4 methods provided in TumorDecon:

**Step 1:** Read in gene expression profiles. Profiles can either be downloaded directly from cBioPortal or UCSC Xena, or loaded in from a local CSV file:

.. code-block:: python

    # Location of sample data (included with the TumorDecon package):
    >>> data_loc = td.get_td_Home()+"data/"
    # Read in sample data (original source - Colorectal Adenocarcinoma RNA Seq v2 from cBioPortal.org):
    >>> rna = td.read_rna_file(data_loc+'coadred_data_RNA_Seq_v2_expression_median.txt')
    >>> print(rna)
                 TCGA-3L-AA1B-01  TCGA-4N-A93T-01  TCGA-4T-AA8H-01  ...  TCGA-AG-A02N-01  TCGA-AG-A02X-01  TCGA-AG-A032-01
    Hugo_Symbol                                                     ...
    A1BG                 22.1470         171.2680          20.9980  ...         6.936515        16.165630        22.836611
    A1CF                220.9870         100.6290         174.0080  ...       174.525498       191.864642       159.905526
    A2BP1                 2.4178          10.1597           0.7311  ...         1.183092        75.017749        23.387740
    A2LD1               177.4080         371.3640         295.6750  ...       111.407100        70.135640       141.760142
    A2M               15911.5000        1494.3300        1333.5700  ...      2198.884111      1163.266249      2785.679374
    ...                      ...              ...              ...  ...              ...              ...              ...
    ZYG11A                3.3849           0.4838           0.0000  ...         0.265865        -0.402069         0.898260
    ZYG11B              543.0370         290.7600         669.7130  ...       914.956269       544.076931       685.383167
    ZYX                6259.1900        4653.1200        4460.6100  ...      2768.765832      5001.453757      4643.428584
    ZZEF1              1358.3200        1220.1300        3002.0100  ...      1151.317795      1822.211921      2125.966458
    ZZZ3                798.3560         333.8170         530.0680  ...       996.284836       735.466401       343.951649

    [17493 rows x 592 columns]

    # Can alternatively download additional data sets directly from cBioPortal, and fetch any missing Hugo Symbols from NCBI website:
    >>> rna_cbio = td.download_by_name('cbio', 'Colorectal Adenocarcinoma', fetch_missing_hugo=True)
    Downloading data from cbioportal to /Users/RachelAronow/testenv/lib/python3.9/site-packages/TumorDecon/data/downloaded//coadread_tcga_pan_can_atlas_2018.tar.gz...
    100% [......................................................................] 221792847 / 221792847
    Fetching missing Hugo Symbols for genes by Entrez ID...
    Found  1 / 13 of the missing values

    >>> print(rna_cbio)
                 TCGA-3L-AA1B-01  TCGA-4N-A93T-01  TCGA-4T-AA8H-01  ...  TCGA-AG-A02N-01  TCGA-AG-A02X-01  TCGA-AG-A032-01
    Hugo_Symbol                                                     ...
    UBE2Q2P2             15.7640           4.2767          11.3032  ...         2.815511         5.598328        13.724609
    HMGB1P1             144.4000         142.6610         143.1990  ...       333.491605       373.021521       409.240194
    LOC155060           441.9730         522.0130         288.0640  ...       138.619145       170.620655       301.792311
    EFCAB8                2.4178           2.4190           2.9245  ...         0.866737         2.568721         1.408164
    SRP14P1               2.9014           2.9028           3.6556  ...         9.461217         3.138180         6.967566
    ...                      ...              ...              ...  ...              ...              ...              ...
    ZYG11A                3.3849           0.4838           0.0000  ...         0.265865        -0.402069         0.898260
    ZYG11B              543.0370         290.7600         669.7130  ...       914.956269       544.076931       685.383167
    ZYX                6259.1900        4653.1200        4460.6100  ...      2768.765832      5001.453757      4643.428584
    ZZEF1              1358.3200        1220.1300        3002.0100  ...      1151.317795      1822.211921      2125.966458
    ZZZ3                798.3560         333.8170         530.0680  ...       996.284836       735.466401       343.951649

    [17498 rows x 592 columns]

    # Or from UCSC Xena:
    >>> rna_xena = td.download_by_name('xena', 'Colon and Rectal Cancer')
    Downloading data from Xena Hub to /Users/RachelAronow/testenv/lib/python3.9/site-packages/TumorDecon/data/downloaded//HiSeqV2.gz...
    100% [........................................................................] 23048575 / 23048575

    >>> print(rna_xena)
                 TCGA-CA-5256-01  TCGA-AZ-6599-01  TCGA-AA-3655-01  ...  TCGA-AF-A56K-01  TCGA-DC-6154-01  TCGA-AG-3592-01
    Hugo_Symbol                                                     ...
    ARHGEF10L        1144.371583      2226.484998      2082.076345  ...      1685.246812      2598.827162      1751.623022
    HIF3A              12.151906         4.076028         3.245104  ...       104.595167        67.693420        33.003233
    RNF17               0.000000         0.000000         0.463578  ...         0.000000         0.000000         0.000000
    RNF10            1432.871476      3297.970130      2907.783085  ...      2209.719087      3005.355704      2588.575669
    RNF11            1463.000164      1334.347953       976.802985  ...       778.173067       856.623127       619.978711
    ...                      ...              ...              ...  ...              ...              ...              ...
    PTRF              712.712881       486.041197      1630.066627  ...      5007.628873      2691.256217      1859.016882
    BCL6B              53.360997        28.531853       129.345774  ...       393.587105       237.575155       132.333917
    GSTK1            6002.965618      3106.847504      5442.428906  ...      2451.776390      3604.041120      4599.898773
    SELP               14.265469        16.813511        64.439962  ...       108.205371        36.614057        74.656913
    SELS             1901.525401      1535.572368      1308.772019  ...      1163.181273      1722.588903      1359.197095

    [20530 rows x 434 columns]

**Step 2-A:** Running Linear Methods (cibersort and DeconRNASeq)

Linear methods require a signature matrix, and output cell fractions:

.. code-block:: python

    # read in LM22 file (the default) as the signature matrix
    >>> sig = td.read_sig_file()
    >>> print(sig)
                 B cells naive  B cells memory  Plasma cells  ...  Mast cells activated  Eosinophils  Neutrophils
    Hugo_Symbol                                               ...
    ABCB4           555.713449       10.744235      7.225819  ...             34.113659    15.030530    14.906888
    ABCB9            15.603544       22.094787    653.392328  ...             23.615746    29.786442    33.679147
    ACAP1           215.305951      321.621021     38.616872  ...             73.607932   696.442802   596.025961
    ACHE             15.117949       16.648847     22.123737  ...             20.501249    40.414927    22.766494
    ACP5            605.897384     1935.201479   1120.104684  ...            323.381277   860.563374   307.142798
    ...                    ...             ...           ...  ...                   ...          ...          ...
    ZNF204P          77.876897       42.946213     41.075062  ...             25.206700    65.487837    27.485546
    ZNF222            9.800664        7.203218     11.691368  ...            211.493742   346.556451    26.825597
    ZNF286A         737.859904      331.908761     73.882022  ...             25.833293     7.473171   441.313469
    ZNF324           28.211994       23.464424     21.173326  ...             45.775175    35.293284    35.424698
    ZNF442           19.713197       22.447533     19.391583  ...              3.646156    40.296509    20.157522

    [547 rows x 22 columns]

    # Can also use a custom signature matrix, such as the example "kmeans_signature_matrix_qval.txt" from the Signature Matrix tutorial.
    # Note this signature matrix uses Ensembl Gene IDs instead of Hugo Symbols - specify this in order to have them converted to Hugo Symbols
    >>> custom_sig = td.read_sig_file(td.get_td_Home()+'data/kmeans_signature_matrix_qval.txt', geneID='Ensembl_Gene_ID')
    >>> print(custom_sig)
                 CD8_subtype_1  CD8_subtype_2  CD4_subtype_1  ...  Fibroblast_subtype_2  Neutrophils_subtype_1  Neutrophils_subtype_2
    Hugo_Symbol                                               ...
    FGR             116.000139      24.380169       8.453629  ...              5.412057             558.770835             739.358363
    NFYA             22.503403       0.752393      15.056201  ...              2.073508              46.759730              37.213350
    LASP1           154.161176      36.456214     110.461637  ...             10.793872             259.252004             250.373201
    TSR3             39.389183      48.998150      16.131921  ...              0.371443               2.038300               3.715013
    NADK             16.429699      12.887361       9.760582  ...              1.503814             871.354356            1006.496017
    ...                    ...            ...            ...  ...                   ...                    ...                    ...
    TRBJ1-3         239.011558     239.012712     281.496628  ...              0.898271               0.000000               0.000000
    TRBJ1-5         607.825543     607.828500     734.535240  ...              1.117383               0.000000               0.000000
    TRBJ1-1         379.176464     379.177563     378.034351  ...              2.021114               0.000000               0.000000
    LINC01002         0.673294       0.928260      12.979473  ...              5.542829            1121.606395            1058.730521
    MIR1248         925.340683     925.015091      49.996583  ...              1.854341               0.000000               0.000000

    [972 rows x 16 columns]

    # Run cibersort method on ALL patients:
    # optional argments include:
    #    'scaling': how to scale the data (rna mixture & signature matrix) before applying Cibersort (must be either 'None', 'zscore', or 'minmax')
    #    'scaling_axis': whether to scale data by normalizing each cell types / patient individually (0) or each gene individually (1). default is 0
    #    'nu','C','kernel': all arguments to be passed to sklearn's implementation of NuSVR. See sklearn documentation for details
    #       Additionally, can pass in 'nu':'best' to choose the nu from [0.25, 0.5, 0.75] that minimizes the root mean square error
    #       Defaults are nu='best', C=1.0, kernel=linear
    #   'print_progress': whether to print patient ID as cibersort works through each patient (default: False)


.. code-block:: python

    >>> ciber_freqs = td.tumor_deconvolve(rna, 'cibersort',  patient_IDs='ALL', sig_matrix=sig, args={'nu':'best', 'scaling':'minmax'})
    Running CiberSort...
    CiberSort has completed!
    >>> print(ciber_freqs)
    Patient_ID       B cells naive  B cells memory  Plasma cells  ...  Mast cells activated  Eosinophils  Neutrophils
    TCGA-3L-AA1B-01       0.076577        0.000000      0.019106  ...              0.066274     0.000000          0.0
    TCGA-4N-A93T-01       0.004291        0.054434      0.045281  ...              0.185916     0.007008          0.0
    TCGA-4T-AA8H-01       0.013573        0.000000      0.118549  ...              0.160762     0.033994          0.0
    TCGA-5M-AAT4-01       0.000000        0.066355      0.000000  ...              0.266957     0.000000          0.0
    TCGA-5M-AAT5-01       0.022618        0.000000      0.024259  ...              0.359011     0.021282          0.0
    ...                        ...             ...           ...  ...                   ...          ...          ...
    TCGA-AG-A026-01       0.000000        0.022986      0.003191  ...              0.253747     0.006140          0.0
    TCGA-AG-A02G-01       0.021680        0.000175      0.000000  ...              0.169971     0.000374          0.0
    TCGA-AG-A02N-01       0.016291        0.007369      0.004204  ...              0.179287     0.009278          0.0
    TCGA-AG-A02X-01       0.005250        0.000000      0.003403  ...              0.181726     0.004704          0.0
    TCGA-AG-A032-01       0.072213        0.000000      0.034568  ...              0.216468     0.009628          0.0

    [592 rows x 22 columns]

    # Save results to a file:
    >>> ciber_freqs.to_csv("cibersort_results.csv", index_label='Patient_ID')

    # Run DeconRNASeq:
    # optional argments include:
    #    'scaling': Same as in cibersort
    #    'scaling_axis': Same as in cibersort
    #    'check_sig': whether to check the condition number of the signature matrix before solving (default: False)
    #    'print_progress': whether to print the results of each fit while function is running (default: False)
    >>> decon_freqs = td.tumor_deconvolve(rna, 'DeconRNASeq',  patient_IDs='ALL', sig_matrix=sig, args={'scaling':'minmax', 'print_progress':False})
    Running DeconRNASeq...
    DeconRNASeq has completed!

    >>> print(decon_freqs)
    Patient_ID       B cells naive  B cells memory  Plasma cells  ...  Mast cells activated   Eosinophils   Neutrophils
    TCGA-3L-AA1B-01   2.582540e-02    0.000000e+00      0.165577  ...          3.088621e-17  0.000000e+00  3.234649e-17
    TCGA-4N-A93T-01   2.305311e-02    1.298978e-16      0.141491  ...          0.000000e+00  5.750904e-18  1.934592e-17
    TCGA-4T-AA8H-01   1.053894e-02    5.177404e-17      0.159989  ...          7.645319e-18  1.235998e-02  5.827587e-19
    TCGA-5M-AAT4-01   5.394090e-17    1.991418e-02      0.109398  ...          3.574299e-17  4.039499e-17  2.120351e-17
    TCGA-5M-AAT5-01   1.089422e-02    4.913046e-17      0.144355  ...          2.883490e-17  6.280517e-18  9.021213e-18
    ...                        ...             ...           ...  ...                   ...           ...           ...
    TCGA-AG-A026-01   2.113573e-02    1.137586e-17      0.141309  ...          1.324951e-17  1.269026e-18  5.087319e-17
    TCGA-AG-A02G-01   1.517414e-02    0.000000e+00      0.131320  ...          0.000000e+00  2.747775e-18  1.425387e-17
    TCGA-AG-A02N-01   2.555726e-02    6.160640e-17      0.130763  ...          0.000000e+00  0.000000e+00  3.678833e-17
    TCGA-AG-A02X-01   2.959905e-02    5.615945e-18      0.182780  ...          1.647317e-17  4.197899e-18  1.486629e-17
    TCGA-AG-A032-01   3.053816e-02    4.980554e-17      0.140349  ...          1.588356e-17  4.061692e-17  0.000000e+00

    [592 rows x 22 columns]

**Step 2-B:** Running Rank-Based Methods (ssGSEA and SingScore)

Rank-based methods require a list of up-regulated genes, and output scores/ranks for each cell type:

.. code-block:: python

  # Read in up-regulated gene set derived from LM22 (default):
  >>> up_geneset = td.read_geneset()
  # When running tumor_deconvolve(), the default is to apply the method on all patients in the rna data frame (patient_IDs='ALL').
  # To instead run the method on only a select list of patients, pass in a list of patient identifiers (column names)
  >>> patient_subset = ['TCGA-3L-AA1B-01', 'TCGA-4N-A93T-01', 'TCGA-4T-AA8H-01', 'TCGA-5M-AAT4-01', \
                      'TCGA-5M-AAT5-01', 'TCGA-5M-AAT6-01', 'TCGA-5M-AATA-01', 'TCGA-5M-AATE-01', \
                      'TCGA-A6-2675-01', 'TCGA-A6-2682-01', 'TCGA-A6-2684-01', 'TCGA-A6-2685-01']

  # Run ssGSEA on only a few patients:
  # optional arguments include:
  #   'alpha': Weight used in ssGSEA method (default: 1.0)
  #   'norm': whether to normalize enrichment scores, as done by Barbie et al., 2009, online methods, pg. 2
  #   'print_progress': whether to print patient ID as ssGSEA iterates through (default: False)
  >>> ssgsea_scores = td.tumor_deconvolve(rna, 'ssGSEA',  patient_IDs=patient_subset, up_genes=up_geneset, args={'alpha':0.5, 'norm':True, 'print_progress':False})

.. code-block:: python

  Running ssGSEA...
  ssGSEA has completed!
  >>> print(ssgsea_scores)
  Patient_ID       B cells naive  B cells memory  Plasma cells  ...  Mast cells activated  Eosinophils  Neutrophils
  TCGA-3L-AA1B-01       0.259867        0.194624      0.885539  ...              0.476357     0.487919     0.347988
  TCGA-4N-A93T-01       0.279005        0.085207      0.876560  ...              0.487487     0.507909     0.403473
  TCGA-4T-AA8H-01       0.229115        0.094173      0.872763  ...              0.440532     0.515211     0.398734
  TCGA-5M-AAT4-01       0.213934        0.146207      0.838287  ...              0.484334     0.478675     0.329150
  TCGA-5M-AAT5-01       0.226495        0.083503      0.887130  ...              0.570072     0.490234     0.360500
  TCGA-5M-AAT6-01       0.220341        0.196759      0.928414  ...              0.495182     0.551787     0.444604
  TCGA-5M-AATA-01       0.264451        0.242712      0.843088  ...              0.562232     0.517319     0.385985
  TCGA-5M-AATE-01       0.255101        0.159893      0.860415  ...              0.487739     0.502279     0.389226
  TCGA-A6-2675-01       0.239517        0.216401      0.880584  ...              0.442438     0.552197     0.480925
  TCGA-A6-2682-01       0.215060        0.159228      0.934567  ...              0.572144     0.557205     0.490885
  TCGA-A6-2684-01       0.200944        0.187249      0.888014  ...              0.466842     0.530049     0.442364
  TCGA-A6-2685-01       0.174148        0.183445      0.901240  ...              0.512232     0.559692     0.475958

  [12 rows x 22 columns]

  # SingScore can be run with just an up-regulated gene set (unidirectional):
  >>> singscore_unidirectional = td.tumor_deconvolve(rna, 'singscore',  patient_IDs=patient_subset, up_genes=up_geneset)
  Running SingScore...
  SingScore has completed!
  >>> print(singscore_unidirectional)
                   B cells naive  B cells memory  Plasma cells  ...  Mast cells activated  Eosinophils  Neutrophils
  Patient_ID                                                    ...
  TCGA-3L-AA1B-01       0.043939        0.019919      0.390517  ...              0.143151     0.170208     0.057520
  TCGA-4N-A93T-01       0.042402       -0.018819      0.388269  ...              0.130023     0.159335     0.047293
  TCGA-4T-AA8H-01      -0.001167       -0.034557      0.383143  ...              0.121452     0.173807     0.054449
  TCGA-5M-AAT4-01      -0.000663       -0.034056      0.352491  ...              0.143960     0.147913     0.072633
  TCGA-5M-AAT5-01       0.001516       -0.026920      0.387989  ...              0.176622     0.160018     0.047570
  TCGA-5M-AAT6-01       0.033225        0.028672      0.410925  ...              0.153879     0.205684     0.129255
  TCGA-5M-AATA-01       0.023101        0.036137      0.361934  ...              0.177856     0.182411     0.094270
  TCGA-5M-AATE-01       0.028539       -0.005005      0.373973  ...              0.135569     0.170774     0.075416
  TCGA-A6-2675-01       0.024810        0.021903      0.382503  ...              0.141583     0.213163     0.167502
  TCGA-A6-2682-01      -0.001531       -0.011193      0.411903  ...              0.194219     0.203540     0.185292
  TCGA-A6-2684-01       0.011188        0.009044      0.387331  ...              0.149757     0.202112     0.137950
  TCGA-A6-2685-01       0.017267        0.014811      0.396467  ...              0.170028     0.221638     0.165198

  [12 rows x 22 columns]

  # or with both an up-regulated and down-regulated gene set (bidirectional):
  # Note we can derive up-regulated and down-regulated genes from an existing signature matrix, such as LM6:
  >>> LM6 = td.read_sig_file(data_loc+'LM6.txt')
  >>> up_geneset_LM6, down_geneset_LM6 = td.find_up_down_genes_from_sig(LM6, down_cutoff=0.4, up_cutoff=4.0)
  >>> singscore_bidirectional = td.tumor_deconvolve(rna, 'singscore',  patient_IDs=patient_subset, up_genes=up_geneset_LM6, down_genes=down_geneset_LM6)
  Running SingScore...
  SingScore has completed!
  >>> print(singscore_bidirectional)
                    B cells  CD8 T cells  CD4 T cells  NK cells  Monocytes  Neutrophils
  Patient_ID
  TCGA-3L-AA1B-01  0.499283     0.498565     0.499251  0.497644   0.498842     0.498363
  TCGA-4N-A93T-01  0.497755     0.496919     0.497468  0.494885   0.495496     0.494552
  TCGA-4T-AA8H-01  0.491771     0.493391     0.495615  0.491519   0.493491     0.491843
  TCGA-5M-AAT4-01  0.495321     0.496566     0.497346  0.495163   0.498167     0.497388
  TCGA-5M-AAT5-01  0.495463     0.495565     0.497145  0.494091   0.495991     0.494845
  TCGA-5M-AAT6-01  0.498709     0.498768     0.498831  0.498318   0.499212     0.498525
  TCGA-5M-AATA-01  0.497041     0.498285     0.498800  0.497634   0.498982     0.498472
  TCGA-5M-AATE-01  0.495452     0.495420     0.497194  0.493926   0.496085     0.494777
  TCGA-A6-2675-01  0.498108     0.498180     0.498828  0.497645   0.499401     0.498931
  TCGA-A6-2682-01  0.497307     0.497994     0.498878  0.497267   0.499530     0.499082
  TCGA-A6-2684-01  0.498561     0.498945     0.499311  0.498411   0.499402     0.498990
  TCGA-A6-2685-01  0.499290     0.499078     0.499545  0.498534   0.499574     0.499424

**Step 3:** Visualize Results (for LM22 cell types only).

.. code-block:: python

  # Select a subset of the patients to visualize:
  >>>results = ciber_freqs.loc[patient_subset]
  # Create and save plots:
  >>>td.cell_frequency_boxplot(results, save_as="boxplots.png")
  >>>td.cell_frequency_barchart(results, save_as="barcharts.png")
  >>>td.pair_plot(results, save_as="pairplots.png")
  >>>td.hierarchical_clustering(results, save_as="clustermaps.png")
  # Each of the plots above use matplotlib and seaborn, and users can customize plot parameters as in these packages. For example:
  >>>td.cell_frequency_boxplot(results, font_scale=0.75, rcParams={'figure.figsize':(5,3)}, save_as="customized-boxplot.png")
