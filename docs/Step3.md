# Step 4: Application of Ensemblex
- [Introduction](#introduction)
- [Ensemblex parameters](#ensemblex-parameters)
- [Applying the Ensemblex algorithm](#applying-the-ensemblex-algorithm)


 - - - -

## Introduction

In Step 4, we will process the output files from the constituent genetic demultiplexing tools with the Ensemblex framework. Ensemblex processes the output files in a three-step pipeline to identify the most probable sample label for each cell based on the predictions of the constituent tools:

**Step 1: Probabilistic-weighted ensemble** <br />
In Step 1,  Ensemblex utilizes an unsupervised weighting model to identify the most probable sample label for each cell. Ensemblex weighs each constituent tool’s assignment probability distribution by its estimated balanced accuracy for the dataset. The weighted assignment probabilities across all four constituent tools are then used to inform the most probable sample label for each cell.

**Step 2: Graph-based doublet detection** <br />
In Step 2, Ensemblex utilizes a graph-based approach to identify doublets that were incorrectly labeled as singlets in Step 1. Pooled cells are embedded into PCA space and the most confident doublets in the pool (nCD) are identified. Then, based on the Euclidean distance in PCA space, the pooled cells that surpass the percentile threshold (pT) of the nearest neighbour frequency to the confident doublets are labelled as doublets by Ensemblex. Ensemblex performs an automated parameter sweep to identify the optimal nCD and pT values; however, user can opt to manually define these parameters. 

**Step 3: Ensemble-independent doublet detection** <br />
In Step 3, Ensemblex utilizes an ensemble-independent approach to further improve doublet detection. Here, cells that are labelled as doublets by Demuxalot or Vireo are labelled as doublets by Ensemblex; however, users can nominate different tools to utilize for Step 3, depending on the desired doublet detection stringency. 


 - - - -

## Ensemblex parameters

Users can choose to run each step of the Ensemblex framework sequentially (Steps 1 to 3) or can opt to skip certain steps. While Step 1 is necessary to generate the ensemble sample labels, Steps 2 and 3 were implemented to improve Ensemblex's ability to identify doublets; thus, if users do not want to prioritize doublet detection, they may skip Steps 2 and/or 3. Nonetheless, we demonstrated in our **pre-print manuscript** that utilizing the entire Ensemblex framework is important for maximizing the demultiplexing accuracy. Users can define which steps of the Ensemblex framework they want to utilize in the adjustable parameters file.

The adjustable parameters file (`ensemblex_config.ini`) is located in `~/working_directory/job_info/configs/`. For a comprehensive description of how to adjust the analytical parameters of the Ensemblex pipeline please see [Execution parameters](reference.md). The following parameters are adjustable when applying the Ensemblex algorithm:

|Parameter|Default|Description|
|:--|:--|:--|
|<ins>**Pool parameters**</ins>|
|PAR_ensemblex_sample_size| NULL| Number of samples multiplexed in the pool.|
|PAR_ensemblex_expected_doublet_rate| NULL|Expected doublet rate for the pool. If using 10X Genomics, the expected doublet rate can be estimated based on the number of recovered cells. For more information see [10X Genomics Documentation](https://kb.10xgenomics.com/hc/en-us/articles/360059124751-Why-is-the-multiplet-rate-different-for-the-Next-GEM-Single-Cell-3-LT-v3-1-assay-compared-to-other-single-cell-applications).|
|<ins>**Set up parameters**</ins>|
|PAR_ensemblex_merge_constituents|Yes|Whether or not to merge the output files of the constituent demultiplexing tools. If running Ensemblex on a pool for the first time, this parameter should be set to "Yes". Subsequent runs of ensemblex (e.g., parameter optimization) can have this parameter set to "No" as the pipeline will automatically detect the previously generated merged file.|
|<ins>**Step 1 parameters: Probabilistic-weighted ensemble**</ins>|
|PAR_ensemblex_probabilistic_weighted_ensemble| Yes|Whether or not to perform Step 1: Probabilistic-weighted ensemble. If running Ensemblex on a pool for the first time, this parameter should be set to "Yes". Subsequent runs of ensemblex (e.g., parameter optimization) can have this parameter set to "No" as the pipeline will automatically detect the previously generated Step 1 output file.|
|<ins>**Step 2 parameters: Graph-based doublet detection**</ins>|
|PAR_ensemblex_preliminary_parameter_sweep| No|Whether or not to perform a preliminary parameter sweep for Step 2: Graph-based doublet detection. Users should utilize the preliminary parameter sweep if they wish to manually define the number of confident doublets in the pool (nCD) and the percentile threshold of the nearest neighour frequency (pT), which can be defined in the following two parameters, respectively. |
|PAR_ensemblex_nCD| NULL|Manually defined number of confident doublets in the pool (nCD). Value can be informed by the output files generated by setting PAR_ensemblex_preliminary_parameter_sweep to "Yes". |
|PAR_ensemblex_pT| NULL|Manually defined percentile threshold of the nearest neighour frequency (pT). Value can be informed by the output files generated by setting PAR_ensemblex_preliminary_parameter_sweep to "Yes".|
|PAR_ensemblex_graph_based_doublet_detection| Yes|Whether or not to perform Step 2: Graph-based doublet detection. If PAR_ensemblex_nCD and PAR_ensemblex_pT are not defined by the user (NULL), Ensemblex will automatically determine the optimal parameter values using an unsupervised parameter sweep. If PAR_ensemblex_nCD and PAR_ensemblex_pT are defined by the user, graph-based doublet detection will be performed with the user-defined values.   |
|<ins>**Step 3 parameters: Ensemble-independent doublet detection**</ins>|
|PAR_ensemblex_preliminary_ensemble_independent_doublet| No|Whether or not to perform a preliminary parameter sweep for Step 3: Ensemble-independent doublet detection. Users should utilize the preliminary parameter sweep if they wish to manually define which constituent tools to utilize for ensemble-independent doublet detection. Users can define which tools to utilize for ensemble-independent doublet detection in the following parameters.|
|PAR_ensemblex_ensemble_independent_doublet| Yes|Whether or not to perform Step 3: Ensemble-independent doublet detection.|
|PAR_ensemblex_doublet_Demuxalot_threshold| Yes|Whether or not to label doublets identified by Demuxalot as doublets. Only doublets with assignment probabilities exceeding Demuxalot's recommended probability threshold will be labeled as doublets by Ensemblex.|
|PAR_ensemblex_doublet_Demuxalot_no_threshold| No|Whether or not to label doublets identified by Demuxalot as doublets, regardless of the corresponding assignment probability.|
|PAR_ensemblex_doublet_Demuxlet_threshold| No|Whether or not to label doublets identified by Demuxlet as doublets. Only doublets with assignment probabilities exceeding Demuxlet's recommended probability threshold will be labeled as doublets by Ensemblex.|
|PAR_ensemblex_doublet_Demuxlet_no_threshold| No|Whether or not to label doublets identified by Demuxlet as doublets, regardless of the corresponding assignment probability.|
|PAR_ensemblex_doublet_Souporcell_threshold| No|Whether or not to label doublets identified by Souporcell as doublets. Only doublets with assignment probabilities exceeding Souporcell's recommended probability threshold will be labeled as doublets by Ensemblex.|
|PAR_ensemblex_doublet_Souporcell_no_threshold| No|Whether or not to label doublets identified by Souporcell as doublets, regardless of the corresponding assignment probability.|
|PAR_ensemblex_doublet_Vireo_threshold| Yes|Whether or not to label doublets identified by Vireo as doublets. Only doublets with assignment probabilities exceeding Vireo's recommended probability threshold will be labeled as doublets by Ensemblex.|
|PAR_ensemblex_doublet_Vireo_no_threshold| No|Whether or not to label doublets identified by Vireo as doublets, regardless of the corresponding assignment probability.|
|<ins>**Confidence score parameters**</ins>|
|PAR_ensemblex_compute_singlet_confidence| Yes|Whether or not to compute Ensemblex's singlet confidence score. This will define low confidence assignments which should be removed from downstream analyses. |


  - - - -

## Applying the Ensemblex algorithm
To apply the Ensemblex algorithm use the following code:

```
ensemblex_HOME=/path/to/ensemblex.pip
ensemblex_PWD=/path/to/working_directory

bash $ensemblex_HOME/launch_ensemblex.sh -d $ensemblex_PWD --step ensemblexing
```

If the ensemblex algorithm completed successfully, the following files should be available in `~/working_directory/ensemblex`

```
working_directory
└── ensemblex
    ├── confidence
    │   └── ensemblex_final_cell_assignment.csv
    ├── constituent_tool_merge.csv
    ├── step1
    │   ├── ARI_demultiplexing_tools.pdf
    │   ├── BA_demultiplexing_tools.pdf
    │   ├── Balanced_accuracy_summary.csv
    │   └── step1_cell_assignment.csv
    ├── step2
    │   ├── optimal_nCD.pdf
    │   ├── optimal_pT.pdf
    │   ├── PC1_var_contrib.pdf
    │   ├── PC2_var_contrib.pdf
    │   ├── PCA1_graph_based_doublet_detection.pdf
    │   ├── PCA2_graph_based_doublet_detection.pdf
    │   ├── PCA3_graph_based_doublet_detection.pdf
    │   ├── PCA_plot.pdf
    │   ├── PCA_scree_plot.pdf
    │   └── Step2_cell_assignment.csv
    └── step3
        ├── Doublet_overlap_no_threshold.pdf
        ├── Doublet_overlap_threshold.pdf
        ├── Number_Ensemblux_doublets_EID_no_threshold.pdf
        ├── Number_Ensemblux_doublets_EID_threshold.pdf
        └── Step3_cell_assignment.csv
```

For a comprehensive description of the Ensemblex algorithm output files, please see [Ensemblex outputs](outputs.md).


