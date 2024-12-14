# Semi-automated Pipeline for Structural Analysis of Compressed Spinal Cord in DCM

## Description

This repository contains the code for pre-processing structural T2-weighted, T2-star, and magnetization transfer MRI images. The primary goal is to estimate biomarkers such as spinal cord atrophy, gray matter atrophy, and white matter injury in patients with degenerative cervical myelopathy (DCM). This pipeline is designed to provide reproducible, standardized, and localized measures of spinal cord injury.
## Objectives

1. Reproducible Analysis Pipeline: A user-friendly pipeline for batch processing of spinal cord morphometrics.
2. Standardized Measures: Generate standardized and normalized morphometric using the PAM50 spinal cord template measures for comparison between patients and controls.
3. Clinical Insight: Provide clinical insights into white matter changes, with magnetization transfer (MT) being particularly sensitive to white matter changes in non-compressed regions.
4. Localized Evaluation: Provide spinal cord level-specific metrics to enable precise localization of spinal cord pathology.

## Data collection and organization
### OU Spine dataset
The OU Spine dataset was acquired using the https://spine-generic.readthedocs.io. The study is ongoing, involving patients diagnosed with DCM and a control cohort of healthy subjects (HC). All MRI scans were acquired using a 3T MR750 GE scanner.
Due to the ongoing nature of the study, patient data is still being collected and analyzed. However, sample patient and control data are available in the Example data folder. Full datasets are available upon reasonable request to the senior author.

### Data Format and Organization
- All MRI datasets were converted from DICOM to NIFTI format and are organized following the Brain Imaging Data Structure (BIDS) format.
  TODO: what was used for data conversion?
- Spinal cord files are renamed according to BIDS standard https://bids.neuroimaging.io.

Here is an example of the BIDS data structure: # TODO adjust with your dataset
~~~
uk-biobank-processed
│
├── dataset_description.json
├── participants.json
├── participants.tsv
├── README
├── sub-1000032
├── sub-1000083
├── sub-1000252
├── sub-1000498
├── sub-1000537
├── sub-1000710
│   │
│   └── anat
│       ├── sub-1000710_T1w.json
│       ├── sub-1000710_T1w.nii.gz
│       ├── sub-1000710_T2w.json
│       └── sub-1000710_T2w.nii.gz
└── derivatives
    │
    └── labels
        └── sub-1000710
            │
            └── anat
                ├── sub-1000710_T1w_seg-manual.nii.gz  <---------- manually-corrected spinal cord segmentation
                ├── sub-1000710_T1w_seg-manual.json  <------------ information about origin of segmentation
                ├── sub-1000710_T1w_labels-manual.nii.gz  <------- manual vertebral labels
                ├── sub-1000710_T1w_labels-manual.json
                ├── sub-1000710_T1w_pmj-manual.nii.gz  <------- manual pmj label
                ├── sub-1000710_T1w_pmj-manual.json
                ├── sub-1000710_T2w_seg-manual.nii.gz  <---------- manually-corrected spinal cord segmentation
                └── sub-1000710_T2w_seg-manual.json
~~~
## Analysis pipeline
### Dependencies
- Spinal Cord Toolbox [(SCT 6.1)](https://github.com/spinalcordtoolbox/spinalcordtoolbox/releases/tag/6.1): Required for spinal cord segmentation and analysis.
- Python 3.9: The processing scripts written in Python. (or analysis scripts? I don't see it in your batch script)
- [FSLeyes](https://open.win.ox.ac.uk/pages/fsl/fsleyes/fsleyes/userdoc/install.html) (FMRIB Software Library): Required for data visualization. (could be ITKsnap, 3Dslicer...)

### Installation
- Spinal Cord Toolbox, [SCT 6.1](https://github.com/spinalcordtoolbox/spinalcordtoolbox/releases/tag/6.1) : Follow the SCT installation guide for instructions on how to download and install SCT script for version 6.1, and integrate it with FSL. [https://spinalcordtoolbox.com/user_section/installation.html](https://spinalcordtoolbox.com/en/stable/user_section/installation.html)


Download this repository:
~~~
git clone [https://github.com/sct-pipeline/ukbiobank-spinalcord-csa.git](https://github.com/Mfauziyya/DCM_Neurosurgery_Practice.git)
~~~

### Usage
- Run the provided preprocessing script in batch mode.
```bash
    sct_run_batch -h
```
- This is the processing script that loops across all participant data. Use the help message to include the mandatory and optional arguments.

#### Example command
```bash
sct_run_batch -path-data /define/your/data/directory/sourcedata/ -jobs 50 -path-output /define/your/analysis/folder -script /specify/your/code/location/Preprocession_extraction.sh -exclude-list [ ses-brain ]
```
- `-path-data`: path to data folder to be processed in BIDS format.
- `-jobs`: Number of subject to run in parallel.
- `-path-output`: Path of analysis results.
- `-script`: preprocessing script (`DCM_Neurosurgery_Practice/Scripts/Preprocession_extraction.sh`).
- `-exclude-list`: list of subjects or session to exclude from the analysis.

### Preprocessing Steps
Spinal Cord MRI (T2 weighted, T2 star, MT) preprocessing include number of key steps 
#### T2 weighted
1. Spinal cord Segmentation
```bash
    sct_deepseg_sc -i ${file}.nii.gz -c t2 -qc qc
```
-    To segment the cervical spinal cord from surrounding neck tissues.
-    include the qc flag to generate the quality control report for this step
2. Vertebral labeling
```bash
   sct_label_vertebrae -i ${file}.nii.gz -s ${file_seg}.nii.gz -c t2 -qc qc
```
3. Registration to PAM50 template 
```bash
    sct_register_to_template -i ${file_t2w}.nii.gz -s ${file_t2_seg}.nii.gz -ldisc ${file_t2_labels_discs}.nii.gz -c t2 -qc qc
```
## Quality Control:
- After preprocessing, perform a QC check by reviewing the HTML files in the QC directory: `<path-out>/qc/index.html`.
- Inspect the T2-weighted and T2-star images for segmentation and vertebral level labeling errors:
<ADD EXAMPLE>

- If errors (e.g., segmentation leakage or under-segmentation) and/or labelling error are found, manually correct them and save them under derivatives/label.
- After corrections, re-run the batch analysis. The pipeline will automatically fetch manually corrected files from the designated folder(./BIDS/derivatives/label).
## Result Export:
- Morphometric and MTR (Magnetization Transfer Ratio) measurements will be exported as CSV files.
- These CSV files can be used for secondary analysis to assess metrics such as  T2-weighted morphometrics (CSA,AP, RL, eccentricity, solidity, MSCC) as well as GM and WM morphometrics, including signal intensity, and tract- and region-based MTR.
- Additionally, the exported data can facilitate group analysis to compare cohorts.

## Publications

- Muhammad F, Weber KA, Bédard S, Haynes G, Smith L, Khan AF, Hameed S, Gray K, McGovern K, Rohan M, Ding L, Van Hal M, Dickson D, Al Tamimi M, Parrish T, Dhaher Y, Smith ZA. Cervical spinal cord morphometrics in degenerative cervical myelopathy: quantification using semi-automated normalized technique and correlation with neurological dysfunctions, The Spine Journal (2024), https://doi.org/10.1016/j.spinee.2024.07.002

- Haynes G, Muhammad F, Weber KA II, Khan AF, Hameed S, Shakir H, Van Hal M, Dickson D, Rohan M, Dhaher Y, Parrish T, Ding L, Smith ZA. Tract-specific magnetization transfer ratio provides insights into the severity of degenerative cervical myelopathy. Spinal Cord (2024). https://doi.org/10.1038/s41393-024-01036-y




