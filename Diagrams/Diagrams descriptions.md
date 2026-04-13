1. Main workflow - the current system workflow for image processing and getting daily CT and segmentation masks for it from CBCT. Workflow includes deformable image registration, synthetic CT generation from CBCT
Both theoretical workflows rely on the idea of high quality sCT.
2. Theoretical workflow 2 - the idea is treating the sCT as deformed CT and using basic image registration for which we do not need segmentation for sCT and just perform image registration bettwen pCT and sCT.
3. Theoretical workflow 2 - the idea is treating the sCT as deformed CT and perform the segmentaton on the sCT.