

### Notes from https://sw-handbook.ujp.cz/en/ml/adaptivni_radioterapie/ and https://sw-handbook.ujp.cz/en/ml/modalities/

##### Difference between curative and palliative radiotherapy
|Aspect|Curative Radiotherapy|Palliative Radiotherapy|
|---|---|---|
|Intent|Cure the cancer|Relieve symptoms|
|Dose|High|Lower|
|Duration|Longer (weeks)|Shorter (days–1 week)|
|Target|Entire tumor|Symptom-causing areas|
|Outcome|Long-term control or cure|Comfort and quality of life|
##### CT vs CBCT
| Feature            | CT             | CBCT                         |
| ------------------ | -------------- | ---------------------------- |
| Beam shape         | Fan beam       | Cone beam                    |
| Image acquisition  | Slice-by-slice | Whole volume in one rotation |
| Soft tissue detail | Excellent      | Limited                      |
| Bone detail        | Good           | Excellent                    |
| Radiation dose     | Higher         | Lower                        |
| Typical setting    | Hospital       | Dental / radiotherapy suite  |
![[Pasted image 20260402161848.png]]

By the default:
- Computed Tomography delivers a **higher radiation dose** than Cone Beam Computed Tomography
- CBCT still uses **ionizing radiation**, but typically less than CT
- MRI uses **no ionizing radiation**
###### Questions:
1. Page **https://sw-handbook.ujp.cz/en/ml/adaptivni_radioterapie/**: "number of fractions". Fractions of what?
2. **CTV** – Clinical Target Volume
3. In the step 3 we are assuming that CBCT or MRI are:
	1. Patient anatomy **changes between fractions** (organ motion, filling, tumor shrinkage), and these changes can **affect dose distribution**;
	2. Daily CBCT or MRI images can be **registered and compared** to the planning CT with sufficient accuracy.
	3. That CBCT and/or MRI images can display same amount of valuable information that planning CT
	4. The **clinical benefit** of detecting anatomical changes and adapting treatment **outweighs the additional radiation dose** from daily CBCT imaging.
	5. There may not be a situation that the treatment should be declined due to critical change in Daily CT that we did not notice while doing CBCT or MRI.
4. "rigid or deformable registration or synthetic CT can be used instead"
5. Why  "Potential dose escalation"? 
	1. Ability to respond to anatomical changes == Reduced margins around tumor, Better protection of healthy tissue
6. LINAC with CBCT imaging, Synthetic CT \= sCT generation

### Notes from https://sw-handbook.ujp.cz/en/ml/projekty/
####Auto Segmentace 
Questions:
1.  In  https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/home Currently best models are able to download with their training .log files are on [UJP cloud](https://cloud.ujp.cz/apps/files/files/6440647?dir=/AI_Segmentace/bundle_onnx/BundleV3) ink is not working. Where the models and .log files from training are stored?
2. 