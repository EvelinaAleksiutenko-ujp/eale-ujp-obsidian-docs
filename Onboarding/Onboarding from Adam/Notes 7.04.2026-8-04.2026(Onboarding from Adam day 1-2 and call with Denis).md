### Finalising the SW handbook

As mentioned in SW handbook we have different modalities: 
1. CT
2. CBCT
3. MRI
Now we are currently working n two of them CT and CBCT.
Right now we are buying only CTs. 

The part of dosage calculation is deterministic.
### Regarding full workflow: 
**ADD DIAGRAM**
Our goal for now is to have PCT and masks on them and hav ecbct but we do not have masks on them so we do sct and do segmentation on it  treating it as CBCT masks.
#### Regarding goals of the project:
We do not have any requirements from the grant regarding metrics and benchmarks.
Even thought we do not have any requirements regarding the amount of classes of OARs that we need to be able to process.
But Denis recommends to move in the direction:
Add set up the monitoring and add benchmarks for metrics dice_score, Hausdorff 95, and additionally Surface distance.
For now it is already set up in MLFlow. 
*Note*: Hausdorff 95(hausdorff ==TODO==: CHECK it is 90 or 95).

*Note*: there will be no need to write any paper at the end of the project.
### Regarding segmentation: 
The classes that we are trying to segment is only "Organs at risk(**OARs**)" and here is the list of them.  
We are not trying to segment the tumor due to this raw reasons:
1. The tumor location is differs from patient to patient.
2. The tumor size differs as well as shape.
3. We want to keep doctors at high awareness and do it by their own.
### Regarding the code

#### Training 
For now the idea is to check if the fine-tuning on the opensource CADS dataset will improve the performance but on the independent testing dataset which we are buying now.
Previously model was trained on the hospitals data just for PoC. But this data 
1. The training script are https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/master/monai_framework/training/training_pipeline.py?ref_type=heads
2. Current filenames from CADS opensourse dataset are [here]( https://cloud.ujp.cz/apps/files/files/10102277?dir=/AI_Segmentace/bundle_onnx/cads_551_baseline_model/logs&editing=false&openfile=true)
3. The limitation for Helious usage is 72 hours.
4. We are using MONAI as a basic lib to work with images.
5. Optuna tuning script can be [found](https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/master/monai_framework/tuning/tuning_pipeline.py?ref_type=heads): 
6. For the fullstack(Ondra) it is necessary to provide output only after [export script](https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/master/monai_framework/export_utils.py?ref_type=heads)
7. [Script for MLFlow monitoring]().[script](https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/master/monai_framework/handlers/mlflow_handler.py?ref_type=heads)
8. The current benchmark leaders are https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/Experiments/Thorax/Base-Model/config.yaml
	1. [ Example of the metrics for them](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/Experiments/Thorax/Base-Model/config.yaml)
	2. All these models were not train with OrientationD(transformation), [the infuence of it.](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/Experiments/CADS/Orientationd)
	3. https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/master/monai_framework/training/training_data.py?ref_type=heads#L26 in  row 81 there is a function which in the other branch feat-transfer-learining updates it with ability to use Orientationd(row 97 in the new branch.
	4. Adam were experimenting with nibabel as the monai orientation d is written with usage of the nibabel, but the results are nearly same to the offline orientation(default that we were using before). So I do not need to experiment with it and just use monai orientationd.
	5. Adam were using only CC BY Number licences(standart) for CADS dataset. Maybe there is a need to investigate which licences we actually can use.
	6. We are not sure which data were used for testing those leaders models(probably for the same subset).
	7. Plan: Adam recommends not to retrain the model with old hospital data and orientation d as we will not be able to use it for grant. ==TODO==: Check it with Denis(are we allowed to use shaddy data)
	8. CADS Data location on Helious: ![[Pasted image 20260408110811.png]] The commented folders are the folders with not the standart licence.
	9. Hospital old data location: ![[Pasted image 20260408111018.png|633]]
	10.  They were experimenting with UNETR on the old hospital data, but per each class they took 200 samples, so we need to experiment with the new data with bigger amount 
		[Info about experiment](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/Experiments/Thorax/Transformer-UNETR)
	11. For retrainng on the new data I should keep the transformations from that new branch. ==TODO==: go through all transformations and monai explanations and recomendation for it.
	12. *Before working with DCM data you need every time convert it to te nifti files. there is no need in the some metadata of the dcm*. *Only theree attributes in metadata should be while working wih niffi files. Use simpleitk*
	13. https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/feat--transfer-learning-update/monai_framework/data/data_container.py?ref_type=heads the data container is the input to the data loader. More info about data loaders(containers, cache dataset, data loader) here https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/feat--transfer-learning-update/monai_framework/training/training_data.py?ref_type=heads in setup_data_loaders
	14. Regarding postprocessing every organ in https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/feat--transfer-learning-update/defined_structures.xlsx?ref_type=heads has its own.
		==TODO==: Ask Denis how it was set up.
	15. Testing is in 153 https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/feat--transfer-learning-update/monai_framework/testing_utils.py?ref_type=heads run_testing
	16. There is cross validation but i should think if i need to use it. Adam were not using it
	17. To upload new data to helious: https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/feat--transfer-learning-update/monai_framework/utils/preliminary_correction_nifty_preparation.py?ref_type=heads
		1. `RT_STRUCT_BASE_PATH` - labels
		2. in dcm contain metadata which says which data modality it is 
		3. before training we conver dcm to nft and thats how we get whole ct and whole label
			1. Example of the label dcm file(last one) ![[Pasted image 20260408113718.png|561]]
	18. After convertion to nift files the images will contain 3 metadata labels(spacing, ,) they are needed for the transformations and the results of loading will be metatensor.
		1. ![[Pasted image 20260408114836.png]]
			
	19. https://dicom.innolitics.com/ciods/multi-frame-single-bit-secondary-capture-image/general-study/00200010 website with all dicom labels(probably you will never need it)
	20. ==TODO==: If i will load the nift files(as an example CADS) with nibabel I will see the atributes and those 3 labels gt_fdata() [the functions to use](https://nipy.org/nibabel/nibabel_images.html#the-image-object)
	21. The bundle is ![[Pasted image 20260408134958.png]] and this is the result of the model export after training
	22. the structure codes is the names of the bundles(models) ![[Pasted image 20260408135505.png]]
	23.  ==TODO==: Ask Denis why we can not train models for segmentation on CBCT
	24. CADS models only(no finetuning) https://cloud.ujp.cz/apps/files/files/10102225?dir=/AI_Segmentace/bundle_onnx/cads_551_baseline_model
	25. https://cloud.ujp.cz/apps/files/files/10102531?dir=/AI_Segmentace/bundle_onnx/finetuned_cads finetuned cads model wit different learing rates+ written in wiki
	26. Demo folders ![[Pasted image 20260408144111.png]] is the folders on which we ere testing the trained model on old hospital data
	27. Demo Calisto works good as it is similar to our data, but others not.(TODO: Ask Jana where the report in  redmine is)
	28. ![[Pasted image 20260408144743.png]]
	Here is how the difference in our files looks like to the demo files
	29. Registration is not working as we need the segments for CBCT. After getting the data and  generating nice sCT we can test existing models, classic approaches as well for DR. So it is can be kinda rewritten.
	30. The deformable registration repository should follow the same structure as auto_segmentace.
Optuna examples:
9. https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/Experiments/Thorax
10. Adam were not using the Optuna. 
11. [The current config for Optuna.](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/Experiments/Thorax/Base-Model/config.yaml)

# TODO
To be added: 
1. Mutual information loss for DFRG task.
2. Read the paper https://arxiv.org/pdf/2102.00590
3. Read the wiki for segmentation

### Recommended materials:
1. https://youtu.be/toOGDXg7gwM?si=P0xYwTRn9Gz_UAgA - for general understanding, wording, etc.
2. https://www.phiro.science/article/S2405-6316(25)00089-2/fulltext
3. Take a look in competitors(Philips, Siemense, Vi..?, franch something, https://www.therapanacea.eu/our-products/mr-box/)
4. Read CycleCut paper.
5. 1. [https://medical.spectronic.se/page-2/page6/index.html](https://medical.spectronic.se/page-2/page6/index.html)
6. [https://www.siemens-healthineers.com/radiotherapy/software-solutions/autocontouring](https://www.siemens-healthineers.com/radiotherapy/software-solutions/autocontouring)
7. [https://www.therapanacea.eu/our-products/annotate/](https://www.therapanacea.eu/our-products/annotate/) (actively working on AI based ART)