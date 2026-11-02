# Our current architecture with a patch_size 64

![[artlab-unet-architecture.svg]]

# Skip connection arguments size mismatch
So we have 4 strides by 2, it means that we shrinking the size of the input 4 tuimes by 2, 2^4 =  16. So our system should be dividable by 16 in order to have contactenation via skip connection have the same size.

Example:
If the input is  8x8x8:


Encoder stage 3:
spatial size 1³ ────────────── saved for the skip connection
       │                                      │
       ▼                                     │
Encoder stage 4:                         │
stride 2, but size stays 1³            │
       │                                      │
       ▼                                     │
Bottleneck: 1³                              │
       │                                       │
       ▼                                      │
Decoder upsamples: 1³ → 2³       │
       │                                       │
       └─── try to combine ────┘
		    2³ with 1³
                         ✗
         
![[unet-8-cube-failure.png]]How it will look like for 16^3  input size:
![[unet-16-cube-success.png]]
# BatchNorm needs more than one value per channel

 `16³` and less with batch 1 
 The deepest spatial map is `1×1×1`(as discussed above) we are inputting into the BatchNorm only one value - PyTorch will throw an error.
 If the batch is bigger than 1 with 16^3 it will not happen.


For 32^3 and batch size 1 it will not happen cause the deepest spatial map is 2x2x2 so BN has 8 values.

For `16×16×32` with batch 1: 1 × 1 × 2 = 2 spatial positions

As the result we will have:
![[astra-conclusion.png]]

## Conclusion

- Four stride-2 stages: each input axis must be a multiple of 16 for matching input/output dimensions.
- Minimum cubic input: `16x16x16`; training BatchNorm requires local patch batch >= 2.
- BatchNorm constraint: `B * (X/16) * (Y/16) * (Z/16) > 1`. `32^3` and `16x16x32` support batch 1.
- Patch dimensions refer to post-spacing voxels. Ablations: `64^3` -> `64x64x32` -> `48x48x32`, reducing Z then XY context.
- Architectural compatibility does not establish optimal segmentation performance; evaluate ROI retention, Dice, and false positives under fixed validation.