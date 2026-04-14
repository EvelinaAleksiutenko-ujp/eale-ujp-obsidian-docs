#### Segmentace(why in CZ?)
1. "Our task is focused **only on organs at risk (OAR)**" - should be OAR's
2. Should say about RTS more maybe "RTSTRUCT" what is the form of it
3. I am not quite sure regarding the input extension type(not sure that it is .dcm)
4. ''Right now this step is not added yet!' Can be "note:".
#### Registration
1. "cca"?
2. I am not sure that the rigid registration s hardly reproducible
3. ''DIR models anatomical changes rather than only position differences.' it looks like some word are missing here
4. ''deformation field' might be better if written "Deformation Vector Field(DVF)" and maybe transformation parameters -> deformation matrix(should be confirmed)
5. optionally resampled aligned image - not done yet - as i understand it is another operation after getting the output from registration.
6. in '## **Current Product Architecture**': 'Model for deformable registration performs poorly - it was experiment to train model without any segmented masks' - *should be a note*; maybe add that for now we are working on synthetic ct for creating a segmented masks(I am not sure that the right place to add it it is here but somewhere it should be, maybe in page regarding syngen)
7. 'Modules are executed depending on clinical workflow requirements.' - about which clinical workflows we are talking about, I understand, but somebody why reads it first time might not
8. I am not sure that the info in chapter ## **Registration Algorithms (Conceptual Overview)** is enough.
9. 'And it is not possible to at least evaluate results of model' - did not get it
10. 'Fixed–moving image pairs generated' - probably it is not 'generated' but more like 'prepared' or etc.
11. Are the '**Dataset Preparation Workflow**' is written to show how the VoxelMorph was trained? Or why the workflow do not include labels preparation?
12. What 'Data validated for spatial consistency' means?
13. Name '**Proposed Preparation Workflow**' - I would made a note under Dataset Preparation Workflow that for now the current approach is below or etc. 
14. Segmentation maps generated - we are not generating them, we annotate the images, it seems so.
15. if possible generate maps from images directly, else generate synthetic CT (sCT) from them instead - segment sCT - used these segmented maps instead' - if I did not know what is being conveyed I would not understand
16. 'with problem of deformable registration' - probably better write more concrete that it will help with training DIR itself;
17. 'its evaluation for multimodal registration task' - not clear to me what does it mean

##### Synthetic CT
1. In chapter **Why Synthetic CT is Important in Radiotherapy**  would mention another usecase as well that we need to generate sCT from CBCT in order to have the CT from the same machine from which the treatment will be delivered.
2. And in the same chapter I wuld also mention another usecase and mark is as a main our approach is to use sCT in order to get segmented CBCT in order to train DIR.
General note: 
3. Sometimes the list options ends with different punctuation sign ',' or '.', or sometimes without it al all. There should become consistency in it, just for prettier view.
4. 'patient coordinate system compatibility.' - did not get it
5. 'Currently in development' - should be as note or should be in the name of the chapter.
		Example: *Note*: *Currently in development*
6. '- _Different models may exist for:_
- _MRI → CT generation,_
- _CBCT → CT correction._
- _CT→ CT correction._' sould probably be after '_modality-specific models_'
7. 'Model is stored and executed using:' I am not sure that we can use word 'executed' here.
8. 'easier integration into clinical systems..' - delete last dot