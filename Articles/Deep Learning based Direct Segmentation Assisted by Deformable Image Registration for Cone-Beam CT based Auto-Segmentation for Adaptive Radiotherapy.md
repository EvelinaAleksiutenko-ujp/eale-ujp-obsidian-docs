Source: https://arxiv.org/pdf/2206.03413
Document quality: AI generated
# Paper Summary & Comparison to Our Approach

## 1. Problem Statement

CBCT-based online adaptive radiotherapy (ART) requires fast and accurate auto-segmentation of OARs and tumor volumes. Two core challenges make this difficult:

- **Poor image quality** — CBCT has artifacts, noise, low soft-tissue contrast, and truncations compared to planning CT
- **Lack of labelled training data** — expert CBCT contours are not generated in routine clinical practice, making large annotated CBCT datasets rare and expensive to produce

---

## 2. Proposed Method — Three Components

### Component 1: Pseudo Label Generation via Multiple DIR Methods

- pCT is **rigidly pre-registered** to CBCT space using Velocity (Varian)
- Three DL-based DIR models are then applied to deformably register pCT → CBCT:
    - **FAIM**
    - **5-cascaded Voxelmorph**
    - **10-cascaded VTN**
- Each DIR model produces a DVF, which is applied to pCT expert contours → generating three sets of deformed pCT contours **(y1, y2, y3)** as pseudo labels on CBCT
- All three DIR models are trained **unsupervised** (NCC + DVF regularization loss) — no segmentation labels required
- During U-Net training, pseudo labels are **randomly switched** per iteration across y1/y2/y3 to mitigate systematic errors from any single DIR method

### Component 2: Influencer Volumes as Spatial Prior

- Deformed pCT contours from a **different** DIR method than the pseudo label are fed as **extra input channels** to the U-Net
- This provides the model with shape and location prior knowledge, compensating for poor CBCT soft-tissue contrast
- To avoid circularity: pseudo label GT and influencer volume always come from **different DIR methods** (i ≠ j, where i,j ∈ {1,2,3})
- This single addition rescues DSC from ~0.63 (pseudo labels only) back to ~0.85 (DIR-level performance)

### Component 3: Fine-tuning with Small True Label Set

- The model pre-trained on pseudo labels + influencer volumes is **fine-tuned** on a small set of 30 patients with expert CBCT contours
- Techniques used to prevent overfitting: **layer freezing**, **early stopping**, **lower learning rate**
- This final step pushes performance **beyond DIR-based methods**

---

## 3. Dataset

|Property|Detail|
|---|---|
|Total patients|137 H&N squamous cell carcinoma|
|pCT scanner|Philips, 1.17 × 1.17 × 3.00 mm³|
|CBCT scanner|Varian On-Board Imager, 0.51 × 0.51 × 1.99 mm³, 512×512×93|
|Patients with true CBCT labels|39 (expert-annotated by radiation oncologist)|
|Patients without CBCT labels|98 (pseudo label training only)|
|Fine-tuning split|30 patients|
|Test split|9 patients|
|Structures|19 OARs + target (brachial plexus, brainstem, parotids, SMGs, mandible, larynx, nGTV, spinal cord, etc.)|

---

## 4. Results

|Model|Description|Avg DSC|
|---|---|---|
|Model_DIR|DIR-based contour propagation (baseline)|~0.85|
|Model_pseudo|U-Net with pseudo labels only, no influencer|~0.63|
|Model_influencer|U-Net with pseudo labels + influencer volumes|~0.85|
|**Model_finetune**|**Fine-tuned on 30 true label patients**|**0.86**|

- **7 out of 19 structures** showed at least 0.02 DSC improvement over DIR baseline after fine-tuning
- Minimum DSC: 0.72 (L_BP) | Maximum DSC: 0.95 (Oral cavity)
- Key finding: **pseudo labels alone are insufficient** — influencer volumes are critical to reach DIR-level performance

---

## 5. Key Findings and Conclusions

- Direct segmentation on CBCT **without prior knowledge is infeasible** due to poor image quality and lack of labels
- Pseudo labels from DIR alone produce poor results (DSC ~0.63) — the model cannot learn reliable boundaries from CBCT images alone
- Influencer volumes provide the essential **spatial prior** that rescues performance to DIR level
- Fine-tuning on even a **small set of true labels** (30 patients) is enough to surpass DIR-based methods
- The DIR models used for pseudo label generation are **fully unsupervised** — true CBCT labels are only needed for the final fine-tuning stage

---

## 6. Comparison to Our Approach

### Overview of Our Pipeline

Our proposed pipeline extends the unsupervised DIR framework (Liang et al. 2021 — CycleGAN + 5-cascaded Voxelmorph) by introducing an AI segmentation component (U-Net) to generate pseudo labels on sCT, which are then used to compute a Dice loss during DIR training.

---

### Detailed Comparison Table

|Aspect|Paper (2206.03413)|Our Approach|
|---|---|---|
|**Goal**|CBCT direct segmentation|CT-to-CBCT DIR + contour propagation|
|**Pseudo label source**|3 DL-based DIR methods (FAIM, Voxelmorph, VTN)|AI U-Net segmentation on sCT|
|**Registration space**|pCT → raw CBCT (cross-modality)|pCT → sCT (same-modality) ✅ better|
|**DIR training**|Unsupervised (NCC + regularization)|Unsupervised + Dice loss (semi-supervised)|
|**Multiple pseudo labels**|✅ 3 sets, randomly switched per iteration|❌ Single U-Net output|
|**Influencer volumes**|✅ Critical component — rescues DSC from 0.63→0.85|❌ Not in current design|
|**Fine-tuning on true labels**|✅ 30 patients with expert CBCT contours|❌ Not planned|
|**Noise sources in pseudo labels**|1 (DIR registration error)|2 (CycleGAN error + U-Net error)|
|**True CBCT labels needed for training**|❌ No (only for fine-tuning)|❌ No|
|**True CBCT labels needed for evaluation**|✅ Yes (9 test patients)|✅ Yes|

---

### Key Risks in Our Approach vs the Paper

#### Risk 1 — Double Error Source in Pseudo Labels

The paper uses classical/DL DIR to generate pseudo labels — one source of noise. Our approach uses a U-Net applied to CycleGAN-generated sCT — introducing **two stacked error sources**:

```
CBCT → CycleGAN (error 1) → sCT → U-Net (error 2) → pseudo labels
```

This compounds uncertainty and may degrade DIR training quality.

#### Risk 2 — No Influencer Volumes

The paper demonstrates that pseudo labels alone drop DSC to ~0.63 — far below DIR baseline. The influencer volume mechanism is what recovers performance. Our current pipeline has no equivalent mechanism.

#### Risk 3 — No Fine-tuning Strategy

The paper explicitly fine-tunes on 30 true-label patients to correct DIR errors embedded in pseudo labels. Without this step, pseudo label noise cannot be corrected.

---

### Advantages of Our Approach vs the Paper

|Advantage|Explanation|
|---|---|
|Same-modality registration|Our pipeline uses pCT → sCT (same modality) which is more accurate than the paper's pCT → raw CBCT cross-modality registration|
|Jointly trained pipeline|CycleGAN and Voxelmorph improve each other during training — the paper trains components separately|
|No need for multiple DIR models|Our approach uses a single jointly trained model rather than 3 separate DIR algorithms|

---

## 7. Recommended Mitigations for Our Approach

Based on lessons from this paper, the following additions are recommended:

1. **Add influencer volumes** — feed pCT RTS as an additional input channel to the segmentation/DIR network alongside the CBCT image, providing spatial prior knowledge
2. **Use multiple pseudo label sources** — if possible, generate pseudo labels from both U-Net on sCT and a classical DIR method (e.g. Elastix), and randomly switch between them during training
3. **Keep Dice loss weight small (λ2 << λ1)** — limit the influence of noisy pseudo labels on DIR training
4. **Plan a fine-tuning stage** — if any true CBCT expert contours are available (even a small set of 10–30 patients), use them to fine-tune and correct pseudo label errors
5. **Validate pseudo label quality** — before using sCT RTS as pseudo GT, quantitatively assess U-Net segmentation quality on sCT vs expert pCT contours to understand the noise level

---

## 8. Summary

The paper (2206.03413) provides strong evidence that pseudo label-based training for CBCT segmentation is viable but requires careful design. The three-component approach — pseudo labels + influencer volumes + fine-tuning — is a well-validated framework that our pipeline partially replicates. The key gap in our current design is the absence of influencer volumes and a fine-tuning strategy, both of which this paper shows are critical for achieving competitive performance. Our use of sCT (same-modality registration) is an advantage over this paper, but the additional error introduced by U-Net pseudo label generation needs to be mitigated carefully.