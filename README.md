# CXR-LC: Deep learning using Chest Radiographs to Identify High-risk Smokers for Lung Cancer Screening: Development and Validation of a Prediction Model

[![CXR-Risk Grad-CAM](/images/fig_small3.png)](https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2738349)

To appear in Annals of Internal Medicine.

## Overview
Chest radiographs (x-rays or CXR) are the most common diagnostic imaging test in medicine. Radiographs are intended to address a specific clinical question (e.g. rule out pneumonia), and most are considered "normal." We hypothesized that normal radiographs also contain "hidden" information about health and longevity that can be extracted using a convolutional neural network (CNN). 

The result is CXR-Risk, a CNN that predicts the risk of 12-year all-cause mortality based on a chest radiograph image. Persons in the highest CXR-risk category had a 53% 12-year mortality rate, 18-fold higher than the lowest risk. Prognostic value was independent of the radiologists' diagnostic findings (e.g. lung nodule) and standard risk factors (e.g. age, sex, diabetes). Please refer to the [JAMA Open manuscript](https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2738349) for details.


**Kaplan-Meier survival based on CXR-Risk category**
[![CXR-Risk Kaplan-Meier Survival](/images/fig2.png)](https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2738349)

This repo contains data intended to promote reproducible research. It is not for clinical care or commercial use. 

## Installation
This inference code was tested on Ubuntu 16.04 LTS, anaconda version 4.7.10, python 3.7.3, fastai 1.0.55, cuda 10.1, pytorch 1.0.1 and cadene pretrained models 0.7.4. A full list of dependencies is listed in `environment.yml`. 

Inference can be run on the GPU or CPU, and should work with ~4GB of GPU or CPU RAM. For GPU inference, a CUDA 10 capable GPU is required.

This example is best run in a conda environment:

```bash
cd location_of_repo
conda env create -n cxrrisk -f environment.yml
conda activate cxrrisk
python3 cxr-risk_inference.py
```

Dummy data files are provided in `development/dummy_dataset/dummy_dataset.csv` and `development/dummy_dataset/dummy_valid.csv`; dummy images are provided in the `development` and `test_images` folders. Weights for the CXR-Risk model are in `development/models/cxr-risk_v1.pth` 

A Docker image is planned -- watch this repo for details.

## Datasets
PLCO (NCT00047385) data used for model development and testing are available from the National Cancer Institute (NCI, https://biometry.nci.nih.gov/cdas/plco/). NLST (NCT01696968) testing data is available from the NCI (https://biometry.nci.nih.gov/cdas/nlst/) and the American College of Radiology Imaging Network (ACRIN, https://www.acrin.org/acrin-nlstbiorepository.aspx). Due to the terms of our data use agreement, I cannot distribute the original data. Please instead obtain the data directly from the NCI and ACRIN.

The `data` folder provides the image filenames, split of PLCO into independent training/tuning/testing datasets, and the CXR-Risk output probabilities:
* `PLCO_dataset_split.csv` describes the split of individuals into training, tuning, and testing datasets. filenamepng refers to the original TIF filename converted to a PNG. "dataset" is 1 for training, 2 for tuning, and 3 for testing. 
* `PLCO_probs.csv` contains the probabilities (cxr-risk_prob) generated by CXR-risk in the PLCO testing dataset. Probabilities were put into ordinal categories (cxr-risk_cat), with 1 corresponding to very low risk and 5 corresponding to very high risk as described in the manuscript.
* `NLST_probs.csv` contains the probabilities (cxr-risk_prob) and ordinal probability categories (cxr-risk_cat) for the NLST testing dataset. The format for filenamepng is the (original participant directory)_(original DCM filename).png


## Image processing
PLCO radiographs were provided as scanned TIF files by the NCI. TIFs were converted to PNGs with a minimum dimension of 512 pixels with ImageMagick v6.8.9-9. 

Many of the PLCO radiographs were rotated 90 or more degrees. To address this, we developed a CNN to identify rotated radiographs. First, we trained a CNN using the resnet34 architecture to identify synthetically rotated radiographs from the [CXR14 dataset](http://openaccess.thecvf.com/content_cvpr_2017/papers/Wang_ChestX-ray8_Hospital-Scale_Chest_CVPR_2017_paper.pdf). We then fine tuned this CNN using 11,000 manually reviewed PLCO radiographs. The rotated radiographs identified by this CNN in `preprocessing/plco_rotation_github.csv` were then corrected using ImageMagick. 

```bash
cd path_for_PLCO_tifs
mogrify -path destination_for_PLCO_pngs -trim +repage -colorspace RGB -auto-level -depth 8 -resize 512x512^ -format png "*.tif"
cd path_for_PLCO_pngs
while IFS=, read -ra cols; do mogrify -rotate 90 "${cols[0]}"; done < /path_to_repo/preprocessing/plco_rotation_github.csv
```

NLST radiographs were provided as DCM files by ACRIN. We chose to first convert them to TIF using DCMTK v3.6.1, then to PNGs with a minimum dimension of 512 pixels through ImageMagick to maintain consistency with the PLCO radiographs:

```bash
cd path_to_NLST_dcm
for x in *.dcm; do dcmj2pnm -O +ot +G $x "${x%.dcm}".tif; done;
mogrify -path destination_for_NLST_pngs -trim +repage -colorspace RGB -auto-level -depth 8 -resize 512x512^ -format png "*.tif"
```


The orientation of several NLST chest radiographs was manually corrected:

```
cd destination_for_NLST_pngs
mogrify -rotate "90" -flop 204025_CR_2000-01-01_135015_CHEST_CHEST_n1__00000_1.3.51.5146.1829.20030903.1123713.1.png
mogrify -rotate "-90" 208201_CR_2000-01-01_163352_CHEST_CHEST_n1__00000_2.16.840.1.113786.1.306662666.44.51.9597.png
mogrify -flip -flop 208704_CR_2000-01-01_133331_CHEST_CHEST_n1__00000_1.3.51.5146.1829.20030718.1122210.1.png
mogrify -rotate "-90" 215085_CR_2000-01-01_112945_CHEST_CHEST_n1__00000_1.3.51.5146.1829.20030605.1101942.1.png
```


## Acknowledgements
I thank the NCI and ACRIN for access to trial data, as well as the PLCO and NLST participants for their contribution to research. I would also like to thank the fastai and Pytorch communities. A GPU used for this research was donated as an unrestricted gift through the Nvidia Corporation Academic Program. The statements contained herein are mine alone and do not represent or imply concurrence or endorsements by the above individuals or organizations.



<b>This work will be presented at the [Symposium on Artificial Intelligence for Learning Health Systems (SAIL)](https://sail.health).</b>
