## Current structure
For now there are 3 types of data we are interested in while storing them in these three folders:
1. Original CT(now can be found in CONTOUR_CORRECTION).
2. RTSTRUCT segmented automatically by segmentation model(with suffics_BundleV3_organs).
3. RTSTRUCT segmented automatically and corrected by students(with suffics _BundleV3_organs_preliminary_correction)
*Note: there is no any 'CT data with few original segmented organs by doctors made during threatment' as it is written in a/folder_structure.md*

For now folder contain:
1. CONTOUR_CORRECTION: original CT, RS_BundleV3_organs(**duplicated** from CONTOUR_CORRECTION_RS)
2. CONTOUR_CORRECTION_RS: RS_BundleV3_organs, RS_BundleV3_organs_preliminary_correction(**duplicated** from CONTOUR_CORRECTION_RS_preliminary_correction folder)
3. CONTOUR_CORRECTION_RS_preliminary_correction: RS_BundleV3_organs_preliminary_correction
TODO: ask if there are any reason for duplication, the reason can be that such structure is needed for the visualization tools and the amount of the data is huge to be coppied.

## Proposed structure and recommendations
1.  CONTOUR_CORRECTION: original CT
2. CONTOUR_CORRECTION_RS: only RS_BundleV3_organs
3. CONTOUR_CORRECTION_RS_preliminary_correction: only RS_BundleV3_organs_preliminary_correction
4. Rename the folder CONTOUR_CORRECTION to the name of the source(e.g. Motol). TODO: Ask Denis about the source
5. Rename CONTOUR_CORRECTION_RS to RS_BundleV3
6. Rename CONTOUR_CORRECTION_RS_preliminary_correction to RS_BundleV3_organs_preliminary_correction
7. Add a note what preliminary_correction is.
## Meaning of additional  .txt, .csv, .xlsx files in CONTOUR_CORRECTION and copyRS_according_ID.py  in CONTOUR_CORRECTION_RS
1. TODO: Ask Denis what was the purpose of the script is the script copyRS_according_ID.py
2. TODO: Ask Denis about all additional(mentioned above in the title) files