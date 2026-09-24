Main sources: 
/mnt/lustre/helios-shared/UJP/alekseve/experiments_with_patching/240122.login1/
/mnt/lustre/helios-home/alekseve/Projects/artlab_as/scripts_local/patch_sampling_compare.py

## How the classes are being picked in RandCropByLabelClassesd?

`RandCropByLabelClassesd`: each call performs an  weighted draw (`np.random.choice`, `p = ratios/sum(ratios)`) over class indices, with no history tracked. Consequently, class selection follows the categorical distribution defined by `ratios` — expected sampling frequency per class equals `ratio_i / sum(ratios)`, with replacement, so a class can recur within the same `num_samples` batch.
## How the ratio strategies inverse_freq and inverse_sqrt_freq are working?
1. First it builds `dataset_voxel_totals_per_roi` by summing every patient's per-class voxel counts across the whole loaded cohort (900 patients in your case).
2. The ratio computed is `ratio = 1.0 / dataset_voxel_totals_per_roi`
Example: 
weights[Bag_Bowel] = 1 / 405,349,208  ≈ 2.47e-9
weights[Lung_R] = 1 / 298,255,620 ≈ 3.35e-9
...
weights[Pituitary] = 1 / 19,829 ≈ 5.04e-5

Same idea for inverse_sqrt_freq:
`ratio = 1.0 / sqrt(dataset_voxel_totals_per_roi)`

## What is a probability for a voxel of a given class to be selected?

#### From what the probability depends on
1. Number of voxels per structure;
2. Patching approach (`RandCropByPosNegLabeld`, `RandCropByLabelClassesd`);
3. For `RandCropByLabelClassesd`: probability depends on the ratio strategy.
   4. Ratio strategies can be:
      1. **Uniform** — every class gets the same ratio (`ratio_i = 1`), so each class is equally likely to be picked as a center regardless of its voxel count.
      2. **Inverse-freq** — `ratio_i = 1 / count_i`, so rare classes are boosted in exact proportion to how underrepresented they are.
      3. **Inverse-sqrt-freq** — `ratio_i = 1 / sqrt(count_i)`, a softer boost that sits between uniform and inverse-freq, avoiding over-correcting for very rare classes.
5. For `RandCropByPosNegLabeld`: from the pos/neg ratio.
   Currently `pos = 3` (75%), `neg = 1` (25%). It means that 3 patch centers will belong to foreground, 1 to background (air).
6. For `RandCropByPosNegLabeld`, within the foreground draw there is no explicit per-class ratio — all foreground voxels across classes are pooled into one index array, so the class actually landed on is implicitly weighted by that class's voxel count (large/common structures dominate).
![Patch sampling comparison](Patch%20sampling%20investigation%20artifacts/Pasted%20image%2020260923101718.png)
*Important: The number of voxels per structure we can not control. 
Ratio for RandCropByPosNegLabeld(pos = 3, neg =1) we are fixing for our experiments.*


### What means "to be selected"?
It can be two meanings:
1. Given class be selected as a center voxel of patching approach.
2. A given class to be present in the selected patch.
   
Why both of this is important to us:
By setting per-class ratios we directly control only the probability that a class is chosen as the crop center (meaning 1). Presence in the patch (meaning 2) is at least that large, since a centered class is always present. It is also raised by the ratios of anatomically neighboring classes and by the patch's physical size, because a class can fall inside a patch centered on another structure.
1. **The ratios of _neighboring_ classes.** If a small organ sits next to a large one, raising the large organ's ratio also raises the small organ's presence. The small organ comes along inside patches centered on its neighbor.
2. **Patch size and voxel spacing.** A larger physical field of view increases how often nearby classes are caught.
3. **Background centers** (`neg` in `pn`). These patches can still contain foreground classes near the edges of organs.
   
So in order to cover both of the points I introduce:
 - `pn_presence_frac`/`lc_presence_frac` = P(this class has **≥1 voxel anywhere inside the patch box**, regardless of where the center landed), per one crop. Answers the question: _"How often does this class actually presented in the patches?". This value depends from strategy and amount of selected patches.
- `pn_center_prob`/`lc_center_prob` = P(this class is the exact class the crop's **center voxel** was drawn from), for **one single crop** — a closed-form approximation, computed from `dataset_voxels`/`ratio`. Answers the question: "if I draw one crop right now, what's the chance its center lands on _any_ voxel of class `c`?"
- **`pn_center_hit_prob`/`lc_center_hit_prob` = P(this class was the center voxel of **at least one** crop, across _all_ `total_draws` crops drawn now)** Answers the question: Did the sampler ever deliberately target this class, anywhere, during all draws?"

### Results of the analysis

#### How big is every class?

The biggest class by voxel counts is Bag_Bowel - 405349208 voxels, the smallest is Pituitary - 19829. This significant difference, as well as ratios between other classes can be visible the first diagram below(blue one).

![coverage_ratio_inverse_freq_n1](Patch%20sampling%20investigation%20artifacts/coverage_ratio_inverse_freq_n1.png)

#### Results for uniform strategy for `RandCropByPosNegLabeld`, `RandCropByLabelClassesd`

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 1;
3. Amount of patches = 2;
Final amount of total draws is: 1800

*Important: pn - means by using RandCropByPosNegLabeld approach, lc - by using RandCropByLabelClassesd;*

![class_selection_prob_uniform_n1](Patch%20sampling%20investigation%20artifacts/class_selection_prob_uniform_n1.png)
First diagram shows: What is the probability of class being present in the patch for all current draws(total draws) of patches;
Second diagram shows: what is the probability of a class to be a crop center; 
Third diagram: P(this class was the center voxel of **at least one** crop, across _all_ `total_draws` crops drawn)**

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 10;
3. Amount of patches = 2;
Final amount of total draws is: 18000
![class_selection_prob_uniform_n10](Patch%20sampling%20investigation%20artifacts/class_selection_prob_uniform_n10.png)

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 30;
3. Amount of patches = 2;
Final amount of total draws is: 54000
![class_selection_prob_uniform_n30](Patch%20sampling%20investigation%20artifacts/class_selection_prob_uniform_n30.png)

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 50;
3. Amount of patches = 2;
Final amount of total draws is: 90000
![class_selection_prob_uniform_n50](Patch%20sampling%20investigation%20artifacts/class_selection_prob_uniform_n50.png)

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 100;
3. Amount of patches = 2;
Final amount of total draws is: 180000
![class_selection_prob_uniform_n100](Patch%20sampling%20investigation%20artifacts/class_selection_prob_uniform_n100.png)


##### Conclusion
By following the changes of the first diagram: 
	- the probability of class being present in patch for different amount of draws stay nearly same; 
	- the probability of the class being present in patch are higher for underrepresented classes with RandCropByLabelClassesd;

By following the changes of the second diagram: 
	- the uniform strategy with RandCropByPosNegLabeld approach is consequesing the larger classes to dominate in the patches; 
	- the uniform strategy with RandCropByPosNegLabeld approach is consequesing all classes to have same probability;

By following the changes of the third diagram: 
	- the probability of the underrepresented classes to be a center at least once is getting higher with amount of draws but not reaching 100% even with 180000 of pathes.
	- the probability of the highly represented classes to be selected as a center of the crop is 100% from the first draw(first patches);
	-  the probability of the middle represented classes to be selected as a center of the crop is getting higher and reaching 100%  till 10 epoch already(or earlier, more details in csv files);




#### Results for inverse strategy for `RandCropByPosNegLabeld`, `RandCropByLabelClassesd`
Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 1;
3. Amount of patches = 2;
Final amount of total draws is: 1800
![class_selection_prob_inverse_freq_n1](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_freq_n1.png)

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 10;
3. Amount of patches = 2;
Final amount of total draws is: 18000

![class_selection_prob_inverse_freq_n10](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_freq_n10.png)
Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 30;
3. Amount of patches = 2;
Final amount of total draws is: 54000

![class_selection_prob_inverse_freq_n30](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_freq_n30.png)

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 50;
3. Amount of patches = 2;
Final amount of total draws is: 90000
![class_selection_prob_inverse_freq_n50](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_freq_n50.png)
Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 100;
3. Amount of patches = 2;
Final amount of total draws is: 180000
![class_selection_prob_inverse_freq_n100](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_freq_n100.png)


##### Conclusion
By following the changes of the first diagram: 
	- the probability of class being present in patch for different amount of draws stay nearly same but for highly represented classes the probablity is lower that the uniform strategy for  RandCropByLabelClassesd;
	- the probability of the class being present in patch are higher for underrepresented classes with RandCropByLabelClassesd;

By following the changes of the second diagram: 
	- the inverse strategy with RandCropByPosNegLabeld approach is consequesing the underrepresented classes to dominate in the patches; 
	- the inverse strategy with RandCropByPosNegLabeld approach is consequesing highly represented classes to be less presented in patches than underrepresented and the difference in probabilities is significant;

By following the changes of the third diagram: 
	- the probability of the highly represented classes to be selected as a center of the crop  is low from the beginning and barely reaching 80%+ till the 100 draw with RandCropByLabelClassesd;
	-  the probability of the  underrepresented classes to be selected as a center of the crop  is low from the beginning and barely reaching 80%+ till the 100 draw with RandCropByPosNegLabeld but having a 100% prob from the beginning with RandCropByLabelClassesd;
	-  the probability of the middle represented classes to be selected as a center of the crop is high from the first draw and reaching 100%  till 3d draw(more in logs);


#### Results for inverse-sqrt strategy for `RandCropByPosNegLabeld`, `RandCropByLabelClassesd`

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 1;
3. Amount of patches = 2;
Final amount of total draws is: 1800
![class_selection_prob_inverse_sqrt_freq_n1](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_sqrt_freq_n1.png)

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 10;
3. Amount of patches = 2;
Final amount of total draws is: 18000
![class_selection_prob_inverse_sqrt_freq_n10](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_sqrt_freq_n10.png)
Parameters:
4. amount of CT studies = 900;
5. Amount of draws(imitating the epochs) = 30;
6. Amount of patches = 2;
Final amount of total draws is: 54000
![class_selection_prob_inverse_sqrt_freq_n30](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_sqrt_freq_n30.png)
Parameters:
7. amount of CT studies = 900;
8. Amount of draws(imitating the epochs) = 50;
9. Amount of patches = 2;
Final amount of total draws is: 90000
![class_selection_prob_inverse_sqrt_freq_n50](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_sqrt_freq_n50.png)

Parameters:
1. amount of CT studies = 900;
2. Amount of draws(imitating the epochs) = 100;
3. Amount of patches = 2;

Final amount of total draws is: 180000
![class_selection_prob_inverse_sqrt_freq_n100](Patch%20sampling%20investigation%20artifacts/class_selection_prob_inverse_sqrt_freq_n100.png)

##### Conclusion
By following the changes of the first diagram: 
	- the probability of class being present in patch for different amount of draws stay nearly same but for highly represented classes the probability is kinda same that the uniform strategy for  RandCropByLabelClassesd and higher than  inverse strategy for  RandCropByLabelClassesd;
	- the probability of the class being present in patch are higher for underrepresented classes with RandCropByLabelClassesd;

By following the changes of the second diagram: 
	- the inverse-sqrt strategy with RandCropByLabelClassesd approach is consequesing the underrepresented classes to dominate in the patches;
	- the inverse-sqrt  strategy with RandCropByLabelClassesd approach is consequesing higly represented classes to be less presented in patches than underrepresented and the difference in probabilities is not such big as in inverse strategy for RandCropByPosNegLabeld;

By following the changes of the third diagram: 
	- the probability of the highly represented classes to be selected as a center of the crop  is not so low as with inverse strategy and fastly reaching 100%(till 10 draw) with RandCropByLabelClassesd;
	-  the probability of the  underrepresented classes to be selected as a center of the crop  is low from the beginning and barely reaching 80% till last draw with RandCropByPosNegLabeld;
	-  the probability of the middle(more focused to the start) represented classes to be selected as a center of the crop is about 100%  for the first draw for RandCropByPosNegLabeld and RandCropByLabelClassesd;


### Final conclusion and recommendations
By following the changes across all the diagrams and all three ratio strategies:

- the probability of a class being present in patch is staying nearly the same for different amount of draws, no matter the strategy or the approach;
- the probability of the class being present in patch are higher for underrepresented classes with RandCropByLabelClassesd, and this effect is getting stronger the more the strategy is pushed toward inverse (uniform → inverse-sqrt → inverse);
- RandCropByPosNegLabeld approach is never consequesing any change in a class being selected as a center, no matter which ratio strategy is chosen, since this approach has no per-class ratio at all;
- the inverse strategy with RandCropByLabelClassesd approach is consequesing the underrepresented classes to dominate in the patches, and the inverse-sqrt strategy is consequesing the same effect but softer;
- the uniform strategy with RandCropByLabelClassesd approach is consequesing all classes to have the same probability to be a center;
- the probability of the highly represented classes to be selected as a center of the crop is low from the beginning and barely reaching 80%+ till the 100 draw with RandCropByLabelClassesd under the inverse strategy, but is not so low and fastly reaching 100% (till the 10th draw) under the inverse-sqrt strategy;
- the probability of the underrepresented classes to be selected as a center of the crop is low from the beginning and barely approaching but not quite reaching 80% till the last draw with RandCropByPosNegLabeld, no matter the ratio strategy chosen;
- the probability of the middle represented classes to be selected as a center of the crop is reaching close to 100% quite early for both approaches, and this happens faster the softer the ratio strategy is (uniform fastest, inverse-sqrt in the middle, inverse the slowest for the biggest classes);
- Aorta and SpinalCord are having high presence_frac not because of the ratio strategy but because they are long structures spanning most of the patient's scan, so the patch is catching them regardless of where the center landed;
- the center_prob formula is an approximation and can be slightly higher than the measured presence_frac for rare, anatomically-localized classes (like Pituitary), since it is pooled across the whole cohort and does not know some patients structurally cannot contain that class.



##### Recommendations

- if RandCropByPosNegLabeld approach is chosen, changing the ratio strategy will not help the underrepresented classes to be selected as a center more often, since this approach is not consequesing any ratio effect at all;
- if RandCropByLabelClassesd approach is chosen, the inverse-sqrt strategy is recommended over the inverse strategy, since it is consequesing the same underrepresented-classes boost but without punishing the highly represented classes as much;

Additional information:
Patching could not cause the 0 we see in or metrics for some classes.(more about this please take a look in /mnt/lustre/helios-shared/UJP/alekseve/experiments_with_patching/240122.login1/coverage_ratio_plots)

Open questions:
1. Why Aorta and SpinalCord are having such a high values of pn_presence_frac, lc_presence_frac?
