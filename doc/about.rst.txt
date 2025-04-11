.. highlight:: rst

About TumorDecon
++++++++++++++++

TumorDecon software includes four deconvolution methods (DeconRNAseq [Gong2013], CIBERSORT [Newman2015], ssGSEA [Şenbabaoğlu2016], Singscore [Foroutan2018]) and several signature matrices of various cell types, including LM22. The input of this software is the gene expression profile of the tumor, and the output is the relative number of each cell type. Users have an option to choose any of the implemented deconvolution methods and included signature matrices or import their own signature matrix to get the results. Additionally, TumorDecon can be used to generate customized signature matrices from single-cell RNA-sequence profiles.

TumorDecon is available on Github (https://github.com/ShahriyariLab/TumorDecon) and PyPI (https://pypi.org/project/TumorDecon/).

If you use the package or some parts of codes, please cite:

Rachel A. Aronow, Shaya Akbarinejad, Trang Le, Sumeyye Su, Leili Shahriyari, "TumorDecon: A digital cytometry software", SoftwareX, Volume 18, 2022, 101072, ISSN 2352-7110, https://doi.org/10.1016/j.softx.2022.101072.
