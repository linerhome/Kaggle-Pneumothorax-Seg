code for [kaggle siim-acr-pneumothorax-segmentation](https://www.kaggle.com/c/siim-acr-pneumothorax-segmentation)

## Requirements
* Pytorch 1.1.0 
* Torchvision 0.3.0
* Python3.7
* Install [backboned-unet](https://github.com/mkisantal/backboned-unet) first
```
git clone https://github.com/mkisantal/backboned-unet.git
cd backboned-unet
pip install .
```
* Install image augumentation library [albumentations](https://github.com/albu/albumentations)
```
conda install -c conda-forge imgaug
conda install albumentations -c albumentations
```
* Install [TensorBoard for Pytorch](https://pytorch.org/docs/stable/tensorboard.html)
```
pip install tb-nightly
pip install future
```
* Install [segmentation_models.pytorch](https://github.com/qubvel/segmentation_models.pytorch)
```
pip install git+https://github.com/qubvel/segmentation_models.pytorch
```

When you run `tensorboard --logdir=checkpoints/unet_resnet34`,  it happens to this error: `ValueError: Duplicate plugins for name projector`, please see [this](https://github.com/pytorch/pytorch/issues/22676)
```
I downloaded a test script from https://raw.githubusercontent.com/tensorflow/tensorboard/master/tensorboard/tools/diagnose_tensorboard.py
I run it and it told me that I have two tensorboards with a different version. Also, it told me how to fix it.
I followed its instructions and I can make my tensorboard work.

I think this error means that you have two tensorboards installed so the plugin will be duplicated. Another method would be helpful that is to reinstall the python environment using conda.
```

## TODO
- [x] unet_resnet34(matters a lot)
- [x] two stage set: two stage batch size(768,1024 big solution matters a lot) and two stages epoch
- [x] epoch freezes the encoder layer in the first stage
- [x] epoch gradients accumulate in the second stage
- [x] data augmentation
- [x] CLAHE for every picture(matters a little)
- [x] lr decay - cos annealing(matters a lot)
- [x] cross validation
- [x] Stratified K-fold
- [x] Average each result of cross validation(matters a lot) 
- [x] stage2 init lr and optimizer
- [x] weight decay(When equal to 5e-4, the negative effect, val loss decreases and dice oscillates, the highest is 0.77)
- [ ] leak, TTA
- [x] adapt to torchvison0.2.0, tensorboard

## Dataset
Creat dataset soft links in the following directories.
```
ln -s ../../../input/train_images/ train_images
ln -s ../../../input/train_mask/ train_mask
ln -s ../../../input/test_images/ test_images
ln -s ../../../input/sample_mask sample_mask
ln -s ../../../input/sample_images/ sample_images
ln -s ../../../input/train-rle.csv train-rle.csv
```

## How to run
ues one gpu for K-fold:
```
CUDA_VISIBLE_DEVICES=0 python main.py
```

use all gpu for K-fold:
```
python main.py
```

run all folds for Stratified K-fold
```
python train_sfold.py
```

The differences between train_sfold.py and main.py are
- the former uses Stratified K fold and Fixed lr, while the latter uses K-fold and random lr

The difference between solver_freeze.py and solver.py are:
- the former considers freezing the encoding part in the first stage, while the latter does not.

Please note that
- solver_freeze.py in both main.py and train_sfold.py are only work for pretrained unet_resnet34. 
- solver.py in both main.py and train_sfold.py are work for all model
  
## Tensorboard
### different event files
Tensorboard displays different event files:
```
tensorboard --logdir=name1:/path/to/logs/1,name2:/path/to/logs/2
```

for example, when the files in the checkpoints/unet_resnet34 folder are as follows
```
├── run1
│   └── events.out.tfevents.1564306811.zdkit.25995.0
├── run2
│   └── events.out.tfevents.1564324685.zdkit.25995.1
```

you can run:
```
cd checkpoints/unet_resnet34
tensorboard --logdir=name1:run1,name2:run2
```

### one event file
Tensorboard displays one event file:
```
tensorboard --logdir=/path/to/logs
```

for example, when the files in the checkpoints/unet_resnet34 folder are as follows
```
├── run1
│   └── events.out.tfevents.1564306811.zdkit.25995.0
```

you can run:
```
cd checkpoints/unet_resnet34
tensorboard --logdir=run1
```

## Results
### Old Submission.csv
|backbone|batch_size|image_size|pretrained|data proprecess|mask resize|less than sum|T|lr|thresh|sum|score|
|--|--|--|--|--|--|--|--|--|--|--|--|
|U-Net|32|224|w/o|w/o|w/o|w/o|w/o|random|||0.7019|
|ResNet34|32|224|w/|w/o|w/o|w/o|w/o|random|||0.7172|
|ResNet34|32|224|w/o|w/o|w/o|w/o|w/o|random|||0.7295|
|ResNet34|20|512|w/|w/o|w/o|w/o|w/o|random|||0.7508|
|ResNet34|20|512|w/|w/|w/o|w/o|w/o|random|||0.7603|
|ResNet34|20|512|w/|w/|w|w/o|w|random|||0.7974|
|ResNet34|20|512|w/|w/|w|1024*2|w/o|random|||0.7834|
|ResNet34|20|512|w/|w/|w|2048*2|w|random||115|0.8112|
|ResNet34 freeze|20|512|w/|w/|w|2048*2|w|random||107|0.8118|
|ResNet34 freeze|20|512|w/|w/|w/|2048*2|w|CosineAnnealingLR|0.45|164|0.8259|
|ResNet34 freeze|20|512|w/|w/ CLAHE|w/|2048*2|w|CosineAnnealingLR|0.47|208|0.8401|
|ResNet34 freeze|20|512|w/|w/ CLAHE|w/|2048*2|w|CosineAnnealingLR|0.40|225|0.8412|
|ResNet34 freeze|20|512|w/|w/ CLAHE|w/|2048*2|w|CosineAnnealingLR|0.36|-|0.8446|
|ResNet34 freeze/No accumulation|20/8|512/1024|w/|w/ CLAHE|512|2048*2|w|CosineAnnealingLR|0.48|210|0.8419|
|ResNet34 freeze/No accumulation|20/8|512/1024|w/|w/ CLAHE|1024|1024*2|w|CosineAnnealingLR|0.48|118|0.7969|
|ResNet34 freeze/No accumulation|20/8|512/1024|w/|w/ CLAHE|1024|1024*2|w|CosineAnnealingLR|0.30|172|0.7958|
|ResNet34 freeze/No accumulation|8|1024|w/|w/ CLAHE|1024|2048*2|w|CosineAnnealingLR|0.35|209|0.8399|
|ResNet34/No accumulation|20|768|w/|w/ CLAHE|1024|2048|w|CosineAnnealingLR|0.46|249(ensemble)|0.8455|

### New Submission.csv
|backbone|batch_size|image_size|pretrained|data proprecess|mask resize|less than sum|T|lr|thresh|ensemble|sum|score|
|--|--|--|--|--|--|--|--|--|--|--|--|--|
|ResNet34/No accumulation|20|768|w/|w/ CLAHE|1024|2048|w|CosineAnnealingLR|0.46|average|171|0.8588|
|ResNet34/No accumulation|20|1024|w/|w/ CLAHE|1024|2048|w|CosineAnnealingLR|0.306|average|207|0.8648|
|ResNet34(New)/No accumulation|20|768|w/|w/ CLAHE|1024|2048|w|CosineAnnealingLR|0.5499|None|172|0.8503|

## Experiment record
|backbone|batch_size|image_size|pretrained|data proprecess|lr|weight_decay|score|
|--|--|--|--|--|--|--|--|
|ResNet34/768|10|768|w/|w/ CLAHE|1e-7,1e-5|0.0|0.79|
|ResNet34/768|10|768|w/|w/ CLAHE|0.0002|0.0|0.8264294|
