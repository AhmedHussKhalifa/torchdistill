# Knowledge distillation kit for PyTorch

## Requirements
- Python 3.6
- [pipenv](https://pypi.org/project/pipenv/)
- [myutils](https://github.com/yoshitomo-matsubara/myutils)


## How to setup
```
git clone https://github.com/yoshitomo-matsubara/kdkit.git
cd kdkit/
git submodule init
git submodule update --recursive --remote
pipenv install
```
If you do not wish to use pipenv (a virtual environment), install the packages listed in [Pipfile](Pipfile).

## Examples
### 1. ImageNet (ILSVRC 2012): Image Classification
#### 1.1 Download the datasets
As the terms of use do not allow to distribute the URLs, you will have to create an account [here](http://image-net.org/download) to get the URLs, and replace `${TRAIN_DATASET_URL}` and `${VAL_DATASET_URL}` with them.
```
wget ${TRAIN_DATASET_URL} ./
wget ${VAL_DATASET_URL} ./
```

#### 1.2 Untar and extract files
```
# Go to the root of this repository
mkdir ./resource/dataset/ilsvrc2012/{train,val} -p
mv ILSVRC2012_img_train.tar ./resource/dataset/ilsvrc2012/train/
cd ./resource/dataset/ilsvrc2012/train/
tar -xvf ILSVRC2012_img_train.tar
for f in *.tar; do
  d=`basename $f .tar`
  mkdir $d
  (cd $d && tar xf ../$f)
done
rm -r *.tar

mv ILSVRC2012_img_val.tar ./resource/dataset/ilsvrc2012/val/
wget https://raw.githubusercontent.com/soumith/imagenetloader.torch/master/valprep.sh
mv valpre.sh ./resource/dataset/ilsvrc2012/val/
cd ./resource/dataset/ilsvrc2012/val/
sh valpre.sh
cd ../../../../
```

#### 1.3 Run an experiment
e.g., Teacher: ResNet-152, Student: AlexNet  
a) Use GPUs for multiple distributed training processes
```
pipenv run python -m torch.distributed.launch --nproc_per_node=${NUM_GPUS} --use_env image_classification.py --world_size ${NUM_GPUS} --config config/image_classification/single_stage/kd/alexnet_from_resnet152.yaml --log log/kd/alexnet_from_resnet152.txt
```
b) Use GPU(s) for single training process
```
pipenv run python image_classification.py --config config/image_classification/single_stage/kd/alexnet_from_resnet152.yaml --log log/kd/alexnet_from_resnet152.txt
```  
c) Use CPU
```
pipenv run python image_classification.py --device cpu --config config/image_classification/single_stage/kd/alexnet_from_resnet152.yaml --log log/kd/alexnet_from_resnet152.txt
```  

#### 1.4 Top 1 accuracy of student models
| Teacher \\ Student    | AlexNet   | ResNet-18 |  
| :---                  | ---:      | ---:      |  
| Pretrained (no *KD*)  | 56.52     | 69.76     |  
| ResNet-152            | 57.22     | 70.34     |

### 2. COCO 2017: Object Detection
#### 2.1 Download the datasets
```
wget http://images.cocodataset.org/zips/train2017.zip ./
wget http://images.cocodataset.org/zips/val2017.zip ./
wget http://images.cocodataset.org/annotations/annotations_trainval2017.zip ./
```

#### 2.2 Unzip and extract files
```
# Go to the root of this repository
mkdir ./resource/dataset/coco2017/ -p
mv train2017.zip ./resource/dataset/coco2017/
mv val2017.zip ./resource/dataset/coco2017/
mv annotations_trainval2017.zip ./resource/dataset/coco2017/
cd ./resource/dataset/coco2017/
unzip train2017.zip
unzip val2017.zip
unzip annotations_trainval2017.zip
cd ../../../
```

#### 2.3 Run an experiment
e.g., Teacher: Faster R-CNN with ResNet-50-FPN backbone, Student: Faster R-CNN with ResNet-18-FPN backbone  
a) Use GPUs for multiple distributed training processes
```
pipenv run python -m torch.distributed.launch --nproc_per_node=${NUM_GPUS} --use_env object_detection.py --world_size ${NUM_GPUS} --config config/object_detection/multi_stage/ft/custom_fasterrcnn_resnet18_fpn_from_fasterrcnn_resnet50_fpn.yaml --log log/ft/custom_fasterrcnn_resnet18_fpn_from_fasterrcnn_resnet50_fpn.txt
```
b) Use GPU(s) for single training process
```
pipenv run python object_detection.py --config config/object_detection/multi_stage/ft/custom_fasterrcnn_resnet18_fpn_from_fasterrcnn_resnet50_fpn.yaml --log log/ft/custom_fasterrcnn_resnet18_fpn_from_fasterrcnn_resnet50_fpn.txt
```  
c) Use CPU
```
pipenv run python object_detection.py --device cpu --config config/object_detection/multi_stage/ft/custom_fasterrcnn_resnet18_fpn_from_fasterrcnn_resnet50_fpn.yaml --log log/ft/custom_fasterrcnn_resnet18_fpn_from_fasterrcnn_resnet50_fpn.txt
```  

## References
- [:mag:](image_classification.py) [pytorch/vision/references/classification/](https://github.com/pytorch/vision/blob/master/references/classification/)
- [:mag:](object_detection.py) [pytorch/vision/references/detection/](https://github.com/pytorch/vision/tree/master/references/detection/)
- [:mag:](config/image_classification/single_stage/kd) Hinton, Geoffrey, Oriol Vinyals and Jeff Dean. ["Distilling the Knowledge in a Neural Network"](https://fb56552f-a-62cb3a1a-s-sites.googlegroups.com/site/deeplearningworkshopnips2014/65.pdf?attachauth=ANoY7co8sQACDsEYLkP11zqEAxPgYHLwkdkDP9NHfEB6pzQOUPmfWf3cVrL3WE7PNyed-lrRsF7CY6Tcme5OEQ92CTSN4f8nDfJcgt71fPtAvcTvH5BpzF-2xPvLkPAvU9Ub8XvbySAPOsMKMWmGsXG2FS1_X1LJsUfuwKdQKYVVTtRfG5LHovLHIwv6kXd3mOkDKEH7YdoyYQqjSv6ku2KDjOpVQBt0lKGVPXeRdwUcD0mxDqCe4u8%3D&attredirects=1) (Deep Learning and Representation Learning Workshop: NIPS 2014)
- [:mag:](config/image_classification/multi_stage/fitnet) Adriana Romero, Nicolas Ballas, Samira Ebrahimi Kahou, Antoine Chassang, Carlo Gatta and Yoshua Bengio. ["FitNets: Hints for Thin Deep Nets"](https://arxiv.org/abs/1412.6550) (ICLR 2015)
- [:mag:](config/image_classification/multi_stage/fsp) Junho Yim, Donggyu Joo, Jihoon Bae and Junmo Kim. ["A Gift From Knowledge Distillation: Fast Optimization, Network Minimization and Transfer Learning"](http://openaccess.thecvf.com/content_cvpr_2017/html/Yim_A_Gift_From_CVPR_2017_paper.html) (CVPR 2017)
- [:mag:](config/image_classification/single_stage/at) Sergey Zagoruyko and Nikos Komodakis. ["Paying More Attention to Attention: Improving the Performance of Convolutional Neural Networks via Attention Transfer"](https://openreview.net/forum?id=Sks9_ajex) (ICLR 2017)
- [:mag:](config/image_classification/single_stage/pkt) Nikolaos Passalis, and Anastasios Tefas. ["Learning Deep Representations with Probabilistic Knowledge Transfer"](http://openaccess.thecvf.com/content_ECCV_2018/html/Nikolaos_Passalis_Learning_Deep_Representations_ECCV_2018_paper.html) (ECCV 2018)
- [:mag:](config/image_classification/multi_stage/ft) Jangho Kim, Seonguk Park and Nojun Kwak. ["Paraphrasing Complex Network: Network Compression via Factor Transfer"](http://papers.neurips.cc/paper/7541-paraphrasing-complex-network-network-compression-via-factor-transfer) (NeurIPS 2018)
- [:mag:](config/image_classification/multi_stage/dab) Byeongho Heo, Minsik Lee, Sangdoo Yun and Jin Young Choi. ["Knowledge Transfer via Distillation of Activation Boundaries Formed by Hidden Neurons"](https://aaai.org/ojs/index.php/AAAI/article/view/4264) (AAAI 2019)
- [:mag:](config/image_classification/single_stage/rkd) Wonpyo Park, Dongju Kim, Yan Lu and Minsu Cho. ["Relational Knowledge Distillation"](http://openaccess.thecvf.com/content_CVPR_2019/html/Park_Relational_Knowledge_Distillation_CVPR_2019_paper.html) (CVPR 2019)
- [:mag:](config/image_classification/single_stage/vid) Sungsoo Ahn, Shell Xu Hu, Andreas Damianou, Neil D. Lawrence and Zhenwen Dai. ["Variational Information Distillation for Knowledge Transfer"](http://openaccess.thecvf.com/content_CVPR_2019/html/Ahn_Variational_Information_Distillation_for_Knowledge_Transfer_CVPR_2019_paper.html) (CVPR 2019)
- [:mag:](config/image_classification/single_stage/cckd) Baoyun Peng, Xiao Jin, Jiaheng Liu, Dongsheng Li, Yichao Wu, Yu Liu, Shunfeng Zhou and Zhaoning Zhang. ["Correlation Congruence for Knowledge Distillation"](http://openaccess.thecvf.com/content_ICCV_2019/html/Peng_Correlation_Congruence_for_Knowledge_Distillation_ICCV_2019_paper.html) (ICCV 2019)
- [:mag:](config/image_classification/single_stage/spkd) Frederick Tung and Greg Mori. ["Similarity-Preserving Knowledge Distillation"](http://openaccess.thecvf.com/content_ICCV_2019/html/Tung_Similarity-Preserving_Knowledge_Distillation_ICCV_2019_paper.html) (ICCV 2019)
- [:mag:](config/image_classification/single_stage/crd) Yonglong Tian, Dilip Krishnan and Phillip Isola. ["Contrastive Representation Distillation"](https://openreview.net/forum?id=SkgpBJrtvS) (ICLR 2020)
- [:mag:](config/object_detection/single_stage/ghnd) Yoshitomo Matsubara and Marco Levorato. ["Neural Compression and Filtering for Edge-assisted Real-time Object Detection in Challenged Networks"](https://arxiv.org/abs/2007.15818) (ICPR 2020)
- [:mag:](config/image_classification/single_stage/tfkd)  Li Yuan, Francis E.H.Tay, Guilin Li, Tao Wang and Jiashi Feng. ["Revisiting Knowledge Distillation via Label Smoothing Regularization"](https://openaccess.thecvf.com/content_CVPR_2020/papers/Yuan_Revisiting_Knowledge_Distillation_via_Label_Smoothing_Regularization_CVPR_2020_paper.pdf) (CVPR 2020)
