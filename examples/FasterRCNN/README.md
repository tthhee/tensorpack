# Faster R-CNN / Mask R-CNN on COCO
This example provides a minimal (2k lines) and faithful implementation of the
following object detection / instance segmentation papers:

+ [Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks](https://arxiv.org/abs/1506.01497)
+ [Feature Pyramid Networks for Object Detection](https://arxiv.org/abs/1612.03144)
+ [Mask R-CNN](https://arxiv.org/abs/1703.06870)
+ [Cascade R-CNN: Delving into High Quality Object Detection](https://arxiv.org/abs/1712.00726)

with the support of:
+ Multi-GPU / multi-node distributed training, multi-GPU evaluation
+ Cross-GPU BatchNorm (aka Sync-BN, from [MegDet: A Large Mini-Batch Object Detector](https://arxiv.org/abs/1711.07240))
+ [Group Normalization](https://arxiv.org/abs/1803.08494)
+ Training from scratch (from [Rethinking ImageNet Pre-training](https://arxiv.org/abs/1811.08883))

This is likely the best-performing open source TensorFlow reimplementation of the above papers.

## Dependencies
+ Python 3.3+; OpenCV
+ TensorFlow ≥ 1.6
+ pycocotools: `pip install 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI'`
+ Pre-trained [ImageNet ResNet model](http://models.tensorpack.com/FasterRCNN/)
  from tensorpack model zoo
+ [COCO data](http://cocodataset.org/#download). It needs to have the following directory structure:
```
COCO/DIR/
  annotations/
    instances_train201?.json
    instances_val201?.json
  train201?/
    # image files that are mentioned in the corresponding json
  val201?/
    # image files that are mentioned in corresponding json
```

You can use either the 2014 version or the 2017 version of the dataset.
To use the common "trainval35k + minival" split for the 2014 dataset, just
download the annotation files `instances_minival2014.json`,
`instances_valminusminival2014.json` from
[here](https://github.com/rbgirshick/py-faster-rcnn/blob/master/data/README.md)
to `annotations/` as well.

<sup>Note that train2017==trainval35k==train2014+val2014-minival2014, and val2017==minival2014</sup>


## Usage
### Train:

To train on a single machine:
```
./train.py --config \
    MODE_MASK=True MODE_FPN=True \
    DATA.BASEDIR=/path/to/COCO/DIR \
    BACKBONE.WEIGHTS=/path/to/ImageNet-R50-AlignPadding.npz
```

To run distributed training, set `TRAINER=horovod` and refer to [HorovodTrainer docs](http://tensorpack.readthedocs.io/modules/train.html#tensorpack.train.HorovodTrainer).

Options can be changed by either the command line or the `config.py` file (recommended).
Some reasonable configurations are listed in the table below.

### Inference:

To predict on an image (needs DISPLAY to show the outputs):
```
./predict.py --predict input1.jpg input2.jpg --load /path/to/Trained-Model-Checkpoint --config SAME-AS-TRAINING
```

To evaluate the performance of a model on COCO:
```
./predict.py --evaluate output.json --load /path/to/Trained-Model-Checkpoint \
    --config SAME-AS-TRAINING
```

Several trained models can be downloaded in the table below. Evaluation and
prediction will need to be run with the corresponding configs used in training.

## Results

These models are trained on trainval35k and evaluated on minival2014 using mAP@IoU=0.50:0.95.
All models are fine-tuned from ImageNet pre-trained R50/R101 models in
[tensorpack model zoo](http://models.tensorpack.com/FasterRCNN/), unless otherwise noted.
All models are trained with 8 NVIDIA V100s, unless otherwise noted.

Performance in [Detectron](https://github.com/facebookresearch/Detectron/) can be reproduced.

 | Backbone                       | mAP<br/>(box;mask)                                                      | Detectron mAP <sup>[1](#ft1)</sup><br/> (box;mask) | Time <br/>(on 8 V100s) | Configurations <br/> (click to expand)                                                                                                                                                                                                                                                                                                                                                                        |
 | -                              | -                                                                       | -                                                  | -                      | -                                                                                                                                                                                                                                                                                                                                                                                                             |
 | R50-C4                         | 34.1                                                                    |                                                    | 7.5h                   | <details><summary>super quick</summary>`MODE_MASK=False FRCNN.BATCH_PER_IM=64`<br/>`PREPROC.TRAIN_SHORT_EDGE_SIZE=600 PREPROC.MAX_SIZE=1024`<br/>`TRAIN.LR_SCHEDULE=[140000,180000,200000]` </details>                                                                                                                                                                                                        |
 | R50-C4                         | 35.6                                                                    | 34.8                                               | 23h                    | <details><summary>standard</summary>`MODE_MASK=False` </details>                                                                                                                                                                                                                                                                                                                                              |
 | R50-FPN                        | 37.5                                                                    | 36.7                                               | 11h                    | <details><summary>standard</summary>`MODE_MASK=False MODE_FPN=True` </details>                                                                                                                                                                                                                                                                                                                                |
 | R50-C4                         | 36.2;31.8 [:arrow_down:][R50C41x]                                       | 35.8;31.4                                          | 23.5h                  | <details><summary>standard</summary>this is the default </details>                                                                                                                                                                                                                                                                                                                                            |
 | R50-FPN                        | 38.2;34.8                                                               | 37.7;33.9                                          | 13.5h                  | <details><summary>standard</summary>`MODE_FPN=True` </details>                                                                                                                                                                                                                                                                                                                                                |
 | R50-FPN                        | 38.9;35.4 [:arrow_down:][R50FPN2x]                                      | 38.6;34.5                                          | 25h                    | <details><summary>2x</summary>`MODE_FPN=True`<br/>`TRAIN.LR_SCHEDULE=[240000,320000,360000]` </details>                                                                                                                                                                                                                                                                                                       |
 | R50-FPN-GN                     | 40.4;36.3 [:arrow_down:][R50FPN2xGN]                                    | 40.3;35.7                                          | 31h                    | <details><summary>2x+GN</summary>`MODE_FPN=True`<br/>`FPN.NORM=GN BACKBONE.NORM=GN`<br/>`FPN.FRCNN_HEAD_FUNC=fastrcnn_4conv1fc_gn_head`<br/>`FPN.MRCNN_HEAD_FUNC=maskrcnn_up4conv_gn_head` <br/>`TRAIN.LR_SCHEDULE=[240000,320000,360000]`                                                                                                                                                                    |
 | R50-FPN                        | 41.7;36.2                                                               |                                                    | 17h                    | <details><summary>+Cascade</summary>`MODE_FPN=True FPN.CASCADE=True` </details>                                                                                                                                                                                                                                                                                                                               |
 | R101-C4                        | 40.1;34.6 [:arrow_down:][R101C41x]                                      |                                                    | 28h                    | <details><summary>standard</summary>`BACKBONE.RESNET_NUM_BLOCKS=[3,4,23,3]` </details>                                                                                                                                                                                                                                                                                                                        |
 | R101-FPN                       | 40.7;36.8 [:arrow_down:][R101FPN1x]                                     | 40.0;35.9                                          | 18h                    | <details><summary>standard</summary>`MODE_FPN=True`<br/>`BACKBONE.RESNET_NUM_BLOCKS=[3,4,23,3]` </details>                                                                                                                                                                                                                                                                                                    |
 | R101-FPN                       | 46.6;40.3 [:arrow_down:][R101FPN3xCasAug] <sup>[2](#ft2)</sup>          |                                                    | 69h                    | <details><summary>3x+Cascade+TrainAug</summary>`MODE_FPN=True FPN.CASCADE=True`<br/>`BACKBONE.RESNET_NUM_BLOCKS=[3,4,23,3]`<br/>`TEST.RESULT_SCORE_THRESH=1e-4`<br/>`PREPROC.TRAIN_SHORT_EDGE_SIZE=[640,800]`<br/>`TRAIN.LR_SCHEDULE=[420000,500000,540000]` </details>                                                                                                                                       |
 | R101-FPN-GN<br/>(From Scratch) | 47.7;41.7 [:arrow_down:][R101FPN9xGNCasAugScratch] <sup>[3](#ft3)</sup> | 47.4;40.5                                          | 28h (on 64 V100s)      | <details><summary>9x+GN+Cascade+TrainAug</summary>`MODE_FPN=True FPN.CASCADE=True`<br/>`BACKBONE.RESNET_NUM_BLOCKS=[3,4,23,3]`<br/>`FPN.NORM=GN BACKBONE.NORM=GN`<br/>`FPN.FRCNN_HEAD_FUNC=fastrcnn_4conv1fc_gn_head`<br/>`FPN.MRCNN_HEAD_FUNC=maskrcnn_up4conv_gn_head`<br/>`PREPROC.TRAIN_SHORT_EDGE_SIZE=[640,800]`<br/>`TRAIN.LR_SCHEDULE=[1500000,1580000,1620000]`<br/>`BACKBONE.FREEZE_AT=0`</details> |

 [R50C41x]: http://models.tensorpack.com/FasterRCNN/COCO-MaskRCNN-R50C41x.npz
 [R50FPN2x]: http://models.tensorpack.com/FasterRCNN/COCO-MaskRCNN-R50FPN2x.npz
 [R50FPN2xGN]: http://models.tensorpack.com/FasterRCNN/COCO-MaskRCNN-R50FPN2xGN.npz
 [R101C41x]: http://models.tensorpack.com/FasterRCNN/COCO-MaskRCNN-R101C41x.npz
 [R101FPN1x]: http://models.tensorpack.com/FasterRCNN/COCO-MaskRCNN-R101FPN1x.npz
 [R101FPN3xCasAug]: http://models.tensorpack.com/FasterRCNN/COCO-MaskRCNN-R101FPN3xCasAug.npz
 [R101FPN9xGNCasAugScratch]: http://models.tensorpack.com/FasterRCNN/COCO-MaskRCNN-R101FPN9xGNCasAugScratch.npz

 <a id="ft1">1</a>: Numbers taken from [Detectron Model Zoo](https://github.com/facebookresearch/Detectron/blob/master/MODEL_ZOO.md).
 We compare models that have identical training & inference cost between the two implementations.
 Their numbers can be different due to small implementation details.

 <a id="ft2">2</a>: Our mAP is __10+ point__ better than the official model in [matterport/Mask_RCNN](https://github.com/matterport/Mask_RCNN/releases/tag/v2.0) with the same R101-FPN backbone.

 <a id="ft3">3</a>: This entry does not use ImageNet pre-training. Detectron numbers are taken from Fig. 5 in [Rethinking ImageNet Pre-training](https://arxiv.org/abs/1811.08883).
 Note that our training strategy is slightly different: we enable cascade throughout the entire training.
 As far as I know, this model is the __best open source model__ on COCO dataset.

## Notes

[NOTES.md](NOTES.md) has some notes about implementation details & speed.
