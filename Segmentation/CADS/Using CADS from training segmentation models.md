There is two main goals to use [CADS](https://huggingface.co/datasets/mrmrx/CADS-dataset) dataset: 
1. To train standalone model just on CADS dataset;
2. To train a standalone model on CADS dataset and use transfer learning to adapt it to our data(as an example full fine-tuning of the network with our data).
## Taking a look on the CADS dataset usage for segmentation
I can see that Adam were training only for the ROI from [HRLLLR_V3](https://cloud.ujp.cz/apps/files/files/6440914?dir=/AI_Segmentace/bundle_onnx/BundleV3/models/MC_HRLLLR_V3) model.
He was experimenting with two approaches: 
1. He was training model just on CADS dataset;
2. Trained model on CADS he fine-tuned on same dataset as [HRLLLR_V3](https://cloud.ujp.cz/apps/files/files/6440914?dir=/AI_Segmentace/bundle_onnx/BundleV3/models/MC_HRLLLR_V3)
## Training baseline CADS model
After taking a look on the training log file https://cloud.ujp.cz/apps/files/files/10102273?dir=/AI_Segmentace/bundle_onnx/cads_551_baseline_model/logs

| Experiment name | Model name | Train dataset size | Validation dataset size | lr   | Number of epochs planned | Real number of epochs |
| --------------- | ---------- | ------------------ | ----------------------- | ---- | ------------------------ | --------------------- |
| Cads baseline   | U-net      | 1200               | 300                     | 1e-4 | 1000                     | 51                    |
The model seems to be not overfitted, but better to build the diagrams to see it better.
 `Epoch[51] Metrics -- learning_rate: 0.0001 train_Adrenal_gland_L_mean_dice: 0.6352 train_Adrenal_gland_R_mean_dice: 0.6797 train_Aorta_mean_dice: 0.8575 train_Gallbladder_mean_dice: 0.4815 train_Inferior_vena_cava_mean_dice: 0.7161 train_Kidney_L_mean_dice: 0.7170 train_Kidney_R_mean_dice: 0.6858 train_Liver_mean_dice: 0.7947 train_Lower_lobe_of_lung_L_mean_dice: 0.8899 train_Lower_lobe_of_lung_R_mean_dice: 0.8980 train_Middle_lobe_of_lung_R_mean_dice: 0.8008 train_Pancreas_mean_dice: 0.6477 train_Portal_vein_and_splenic_vein_mean_dice: 0.5091 train_Spleen_mean_dice: 0.7536 train_Stomach_mean_dice: 0.6918 train_Upper_lobe_of_lung_L_mean_dice: 0.8247 train_Upper_lobe_of_lung_R_mean_dice: 0.7172 train_hausdorff_distance: 29.9840 train_loss: 0.3732 train_mean_dice: 0.7236 train_mean_surface_distance: 2.5250`


## Comparison of CADS+ finetuning on bundlev3 data vs Bundle_v3 training:

| Experiment name    | Model name | Number of epochs planned | Real number of epochs | Train dataset size | Validation dataset size | lr   | train_Heart_mean_dice 50 epoch | train_Lung_L_mean_dice<br>50 epoch | train_Lung_R_mean_dice<br>50 epoch | train_Heart_mean_dice result | train_Lung_L_mean_dice<br>result | train_Lung_R_mean_dice<br>result |
| ------------------ | ---------- | ------------------------ | --------------------- | ------------------ | ----------------------- | ---- | ------------------------------ | ---------------------------------- | ---------------------------------- | ---------------------------- | -------------------------------- | -------------------------------- |
| CADS + FN bundlev3 | U-net      | 50                       | 50                    | 400                | 100                     | 1e-4 | 0.8929                         | 0.9422                             | 0.9551                             | 0.8929                       | 0.9422                           | 0.9551                           |
| CADS + FN bundlev3 | U-net      | 300                      | 89                    | 400                | 100                     | 5e-4 | 0.0125                         | 0.0244                             | 0.0094                             | 0.0113                       | 0.0219                           | 0.0107                           |
| Bundlev3           | U-net      | 800                      | 298                   | 400                | 100                     | 5e-4 | 0.8683                         | 0.9257                             | 0.9116                             | 0.8730                       | 0.9214                           | 0.9183                           |
|                    |            |                          |                       |                    |                         |      |                                |                                    |                                    |                              |                                  |                                  |
Conclusion: 
1. There is no any sense on the experiment CADS + FN BundleV3 because lr was too high and training was stopped on 86 epoch(out of 300 planned) probably due to the stopping rule.
2. Is the stopping rule will work if the model actually overfitting?
	1. TODO: investigate the stopping rule
	2. I think that bundlev3 models might be overfitted 
3. Probably they used cross-validation due to the small amount of data for raining for many classes;
Idea: it is not convenient to compare models if their logs are only in log files. We should have unified mlflow or tensorboard for all our experiments.
Idea BS: BS is too small and gradient may be too sensitive to the images in batch

## Questions
 1. How exactly Adam were doing transfer learning? Were he fine-tuning full network? Maybe we should try to freeze something?
 2. What 551 means for baseline model?
 3. Are the cross validation is included while training?