## On which dataset the Bundlev3 were trained
It can be found by following the pathes from config pathes on Helios
## Model names and ROI names matching
1. C:\Users\aleksiutenko\Projects\auto_segmentace\defined_structures.xlsx here can be found the strcuture codes and preprocessing transformations for every organ.
# BundleV3 VAG
## Investigation training logs 
1. VAG:
	   1. 2024-10-01 19:04:18,291 - INFO - [CONFIG]: n_max_samples: 550
		   1. What n_max_samples means?
		   2. 2024-10-01 19:04:23,762 - INFO - Training dataset size: 27
		And the amount of epochs: 500 :) and what was the rule to resume training? Is the current model best is from 500?
		3. It seems to be strange that first 50(equaly 50) epochs model shows 0 for mean_dice. 
		4. Are we sure it is unet as in config I found model_type: UNet and swinunetr_num_heads: [3, 6, 12, 24]
			2024-10-01 19:04:18,290 - INFO - [CONFIG]: swinuter_feature_size: 24
			2024-10-01 19:04:18,290 - INFO - [CONFIG]: unetr_num_heads: 12
			2024-10-01 19:04:18,290 - INFO - [CONFIG]: unetr_hidden_size: 768
			2024-10-01 19:04:18,290 - INFO - [CONFIG]: unetr_mlp_dim: 3072
			2024-10-01 19:04:18,290 - INFO - [CONFIG]: unetr_feature_size: 24
			So if the model is Unet it is  strange that rifts 50 epochs it has no progression in dice score.
	2. I was trying to find the tensorboads from bundlev3  training but i did not found it in helios. In logs it is written that it might be in 2024-10-01 19:04:18,291 - INFO - [CONFIG]: tensorboard_dir: /mnt/lustre/helios-home/dudasden/Female_Pelvis/vagina/general_model/run1_o_holesov/tensorboard
		1. TODO: ask Denis to give it to you. Ask: Am I getting right that there was no mlflow set up while bundlev3 training?
		2. 
# BundleV3 MC_HRLLLR_V3
## Val and Train loss bundlev3 
![[Pasted image 20260525193432.png]]
![[Pasted image 20260525193504.png|697]]
![[Pasted image 20260525193534.png]]
![[Pasted image 20260525193549.png]]
Epoch[12] Metrics -- val_Heart_mean_dice: 0.8692 val_Lung-Left_mean_dice: 0.9290 val_Lung-Right_mean_dice: 0.9357 val_hausdorff_distance: 28.8178 val_loss: 0.1103 val_mean_dice: 0.9113 val_mean_surface_distance: 2.0842 

 Epoch[12] Metrics -- learning_rate: 0.0005 train_Heart_mean_dice: 0.8085 train_Lung-Left_mean_dice: 0.8906 train_Lung-Right_mean_dice: 0.8748 train_hausdorff_distance: 45.0698 train_loss: 0.1709 train_mean_dice: 0.8580 train_mean_surface_distance: 3.1634

 Epoch[120] Metrics -- val_Heart_mean_dice: 0.9359 val_Lung-Left_mean_dice: 0.9212 val_Lung-Right_mean_dice: 0.8810 val_hausdorff_distance: 12.9106 val_loss: 0.1276 val_mean_dice: 0.9127 val_mean_surface_distance: 0.8554 

 Epoch[120] Metrics -- learning_rate: 0.0005 train_Heart_mean_dice: 0.8783 train_Lung-Left_mean_dice: 0.9279 train_Lung-Right_mean_dice: 0.9113 train_hausdorff_distance: 16.4625 train_loss: 0.1289 train_mean_dice: 0.9059 train_mean_surface_distance: 2.0072  
  

Epoch[126] Metrics -- val_Heart_mean_dice: 0.9318 val_Lung-Left_mean_dice: 0.9674 val_Lung-Right_mean_dice: 0.9722 val_hausdorff_distance: 12.1409 val_loss: 0.0624 val_mean_dice: 0.9571 val_mean_surface_distance: 1.1634 

2024-08-17 22:00:07,827 - INFO - Epoch[126] Metrics -- learning_rate: 0.0005 train_Heart_mean_dice: 0.8790 train_Lung-Left_mean_dice: 0.9203 train_Lung-Right_mean_dice: 0.9069 train_hausdorff_distance: 19.1280 train_loss: 0.1280 train_mean_dice: 0.9021 train_mean_surface_distance: 1.8305 

  

 Epoch[180] Metrics -- val_Heart_mean_dice: 0.9372 val_Lung-Left_mean_dice: 0.9668 val_Lung-Right_mean_dice: 0.9682 val_hausdorff_distance: 8.2997 val_loss: 0.0640 val_mean_dice: 0.9574 val_mean_surface_distance: 0.8981 

  Epoch[180] Metrics -- learning_rate: 0.0004 train_Heart_mean_dice: 0.8776 train_Lung-Left_mean_dice: 0.9232 train_Lung-Right_mean_dice: 0.9074 train_hausdorff_distance: 39.9906 train_loss: 0.1274 train_mean_dice: 0.9027 train_mean_surface_distance: 2.0054 

  Epoch[240] Metrics -- val_Heart_mean_dice: 0.9424 val_Lung-Left_mean_dice: 0.9698 val_Lung-Right_mean_dice: 0.9700 val_hausdorff_distance: 7.3891 val_loss: 0.0595 val_mean_dice: 0.9607 val_mean_surface_distance: 0.6745 

   Epoch[240] Metrics -- learning_rate: 0.0004 train_Heart_mean_dice: 0.8789 train_Lung-Left_mean_dice: 0.9196 train_Lung-Right_mean_dice: 0.9219 train_hausdorff_distance: 35.4752 train_loss: 0.1123 train_mean_dice: 0.9068 train_mean_surface_distance: 2.4643 

  

  Epoch[246] Metrics -- val_Heart_mean_dice: 0.9379 val_Lung-Left_mean_dice: 0.9696 val_Lung-Right_mean_dice: 0.9664 val_hausdorff_distance: 12.1915 val_loss: 0.0599 val_mean_dice: 0.9580 val_mean_surface_distance: 1.2353 

  Epoch[246] Metrics -- learning_rate: 0.0004 train_Heart_mean_dice: 0.8758 train_Lung-Left_mean_dice: 0.9251 train_Lung-Right_mean_dice: 0.9196 train_hausdorff_distance: 38.0666 train_loss: 0.1129 train_mean_dice: 0.9069 train_mean_surface_distance: 2.9612 


  Epoch[282] Metrics -- val_Heart_mean_dice: 0.9388 val_Lung-Left_mean_dice: 0.9689 val_Lung-Right_mean_dice: 0.9665 val_hausdorff_distance: 13.0380 val_loss: 0.0592 val_mean_dice: 0.9581 val_mean_surface_distance: 0.8890 

   Epoch[282] Metrics -- learning_rate: 0.0004 train_Heart_mean_dice: 0.8833 train_Lung-Left_mean_dice: 0.9241 train_Lung-Right_mean_dice: 0.9222 train_hausdorff_distance: 31.2469 train_loss: 0.1062 train_mean_dice: 0.9098 train_mean_surface_distance: 2.5380 

  Epoch[294] Metrics -- val_Heart_mean_dice: 0.9390 val_Lung-Left_mean_dice: 0.9691 val_Lung-Right_mean_dice: 0.9667 val_hausdorff_distance: 8.3544 val_loss: 0.0572 val_mean_dice: 0.9583 val_mean_surface_distance: 0.6651 

  Epoch[294] Metrics -- learning_rate: 0.0004 train_Heart_mean_dice: 0.8845 train_Lung-Left_mean_dice: 0.9287 train_Lung-Right_mean_dice: 0.9273 train_hausdorff_distance: 32.1588 train_loss: 0.1010 train_mean_dice: 0.9135 train_mean_surface_distance: 2.8280


Conclusion: 
1. val_loss is barely changing from epoch 240;
2.  val_hausdorff_distance is started to grow from 246 epoch; 
3. val_mean_surface_distance is jumping from epochs to epochs;
4. train_mean_surface_distance started increasing from 126 epochs
5. Val loss plateaued at epoch 126 and remained stable through epoch 294, with only marginal improvement from 0.0595 to 0.0572 after epoch 240.
6. Val hausdorff distance shows a clear decreasing trend from epoch 12 to 240 (28.8 → 7.39), but has started growing from epoch 246 onward, suggesting boundary precision begins to degrade after epoch 240..
    
7. Val mean surface distance follows a downward overall trend (2.08 → 0.67), confirming improving boundary quality, despite noisy oscillations between individual epochs.
    
8. Train mean surface distance reached its best value at epoch 126 (1.83) and has been increasing since that.
    
9. Val metrics consistently outperform train metrics from epoch 126 onward, *indicating the validation set is easier than the training set and the train/val split may not be fully representative.*
    
10. Heart segmentation is the weakest structure throughout all epochs
    
11. No single epoch is optimal across all metrics simultaneously — the best checkpoint should be selected based on a composite of val hausdorff distance and val mean surface distance rather than val loss alone.
Questions: 
12. What was the stopping rule for training?
13. Was the CrossValidation used for validating?
14.  Am I getting right that for bundlev3 you were taking model weights from the last epoch? Is there are any existing checkpoints from any other epochs for bundlev3?
## Wrong early stopping set up
Source: https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/master/monai_framework/training/training_handlers.py?ref_type=heads#L106

`    if config.early_stop:
        `train_handlers.append(`
            `EarlyStopHandler(`
                `patience=int(math.ceil(10 * math.log(config.epochs))),`
                `score_function=lambda x: -x.state.output[0]["loss"],`
            `)`
        `)`
        The Early stopping rule should be focused on validation loss(and maybe validation dice), not on training loss. That's wy in combination with 200+ epochs it gives overfitting(I guess so).
### Why crossvalidation is not enough?