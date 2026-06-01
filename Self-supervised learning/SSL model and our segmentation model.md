# Main Sources

- [https://arxiv.org/abs/2505.13584](https://arxiv.org/abs/2505.13584)
- [Conversation with Denis (20.05.2026)](https://docs.google.com/document/d/1wgFpAq3dvdR23c8oyqpRKRRghtLsXIQ0xlQAyFnk6Wg/edit?usp=sharing)

# Introduction

The main goal of self-supervised learning (SSL) in our work is to improve the performance of the segmentation model while having minimal the amount of labeled data (annotations) required for training. At the same time, we assume the availability of a sufficiently large amount of unlabeled image data.

One possible approach is to train a deep neural network capable of learning meaningful feature representations from unlabeled data. These learned representations can then be transferred to the downstream segmentation task.

Before training an SSL model for segmentation, three main factors should be defined:

1. The type of representations we want the model to learn and the corresponding pretext task capable of learning such representations.
2. The SSL model architecture.
3. The knowledge transfer strategy between the SSL model and the downstream segmentation model.

## Representation Learning and Pretext Tasks

The first and most important factor is the type of representations we want the model to learn. In SSL, the quality and characteristics of the learned feature space directly determine how useful the learned representations will be for the downstream segmentation task.

After defining the target representation types, we can analyze different pretext tasks(described in the survey paper above) and determine which of them are capable of learning the required features. 
## SSL Model Architecture

The second factor is the SSL model architecture. The selected architecture must be capable of extracting desired representations. Different architectures encode spatial and semantic information differently, which directly affects feature quality and downstream segmentation improvment.

In our work, the downstream segmentation model is based on the U-Net architecture. We also experimented with three additional architectures commonly used in segmentation research. The exact model names and implementation details can be found in the following the [link](https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/feat--transfer-learning-update/monai_framework/models/model_factory.py?ref_type=heads).

## Knowledge Transfer Strategy

The third factor is the knowledge transfer strategy between the SSL-pretrained model and the downstream segmentation model. 

Depending on the selected approach, the pretrained encoder may be fully fine-tuned, partially frozen, or used only as a feature extractor. The effectiveness of knowledge transfer depends on both the compatibility between the learned representations and the downstream task requirements, as well as the architectural alignment between the SSL model and the segmentation model.


While analyzing the paper I found different strategies of knowledge transfer:
1. Combining ssl and segmentation backbones(below - approaches 1, 2)
2. Using for ssl same backbone as segmentation model, no stacking(below - approach 3). Same as using just preinialized differently(not randomly, with a sense) current backbone of segmentation model.
3. Using different backbone then segmentation and task-specific head.
4. Bridging the weights from different layers of ssl backbone o another different layer of current segmentation model backbone.

Current approaches of knowledge transfer that we will focus on:
5. Combining SSL model backbone(already train on pretext task) with the segmentation model. Training segmentation model, while freezing the SSL part. So ssl component is used here as feature extractor only.
6. Combining SSL model backbone(already train on pretext task) with the segmentation model. Training segmentation model, without the SSL part.
7. Using ssl model with the same or different architecture as current segmentation model (without combination with current segmentation backbone) with task specific head. Also the ssl part can be freezed or not freezed while fine-tuning.
I personaly find this approaches very connected to the idea of transfer learning.