Idea: we have CT/CBCT only HaN from Motol and Pancreatic-CT-CBCT-SEG. On the call with Denis 13.05.2026 we discussed that to proceed further in our goal(DIR training) we only have this data and there are two scenarious:
1. Get the data 
	1. Ask Motol for more
	2. Get more data from the company(Denis knows the name).
2. As the backup plan focus only on the HaN during whole our pipeline.
	1.  Additional motivation to say in the report: while visiting Motol together with Abhiram became to know that they are doing replanning for HaN region more that for any other region due to the high volatility of this region.

## Current HaN Motol data
Source: A:\DATA\Image_Registration\HaN_Motol\data-han
The source contain 4475 registration files(TODO: confirm with DD), probably it means that we have 4475 unique pairs CT/CBCT to train DIR. 
Training BundleV3: Regarding using this data in annotation: the data were used to train BundleV3 https://cloud.ujp.cz/apps/files/files/6505244?dir=/AI_Segmentace/bundle_onnx/BundleV3/models/SC_Brainstem&editing=false&openfile=true model(at least this one).
Preliminary correction: TODO check if there are preliminary corrected data. There are some HaN folder A:\DATA\Segmentation\CONTOUR_CORRECTION_RS_preliminary_correction\HaN
## Getting the data
For getting the data from Motol we should ask Ondřej Konček. 
Or if we wanted to buy we should ask Denis for the contact.
## My conclusion

I need to confirm if the Pancreatic-CT-CBCT-SEG is usable for us. 
TODO: ask Denis

And due to the time constraint I would now proceed with HaN for every module we should have till the end of phase 2(09/30/2026).
So in order to do that as a first priority we should prioritize the labeling of the data for HaN.
And while doing that we may ask use Pancreatic-CT-CBCT-SEG  and ask for the data.

## Additional info about folder demo_generation_nii
I did not found any info about what is the source of the data, DD does not know as well. I did not found it on Helios as well.
