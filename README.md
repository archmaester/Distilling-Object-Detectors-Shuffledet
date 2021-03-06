🔥Updating🔥

## TODO
- Still to come:
  * [ ] Separate sub losses to print. Add visualization of detection output.
  * [ ] Combining the proposed method with model pruning/quantization method.


### Preparation
python2
tensorpack=0.8.6
tensorflow=1.8.0

#### 1 Clone the repository
First of all, clone the code
```
git clone https://github.com/twangnh/Distilling-Object-Detectors-Shuffledet
```

#### 2 Data preparation
Note we split KITTI training set into train/val sets and eval our methods and models on val set, since test set label is not available.
KITTI 2D object detection images are sampled from video, randomly split the training data into train and val set could lead to 
exceptionally high performance due to correlation between video frames, we follow [MSCNN [1], Zhaowei Cai et.al.](https://arxiv.org/abs/1607.07155) which split
KIITI training set to train/val sets while ensuring images frames does not come from close video frames.

* download images at http://www.cvlibs.net/download.php?file=data_object_label_2.zip and extract into `./data/KITTI/training/image_2/`
* download labels at http://www.cvlibs.net/download.php?file=data_object_label_2.zip and extract into `./data/KITTI/training/label_2/`
* The train val split image index files are ready at `./data/KITTI/ImageSets`

#### 3 download imagenet pretrained model and trained 1x teacher model

* download imagenet pretrained **0.5x** modified-shufflenet backbone model at [0.5x GoogleDrive](https://drive.google.com/file/d/1Fk40F-M0QtiDapiFmzRmnJKPPETmwSnv/view?usp=sharing) 
and put it into `./pretrained_model/shuffle_backbone0.5x/`

* download imagenet pretrained **0.25x** modified-shufflenet backbone model at [0.25x GoogleDrive](https://drive.google.com/file/d/1msI_8qpSqb4azhhhDnzl_8N-jiI1tI5B/view?usp=sharing) 
and put it into `./pretrained_model/shuffle_backbone0.25x/`

* download trained **1x** model at [GoogleDrive](https://drive.google.com/file/d/1jTTeyA61ZPDgwrvHeelysTQLUSWbIOul/view?usp=sharing)
and put it into `./kitti-1x-supervisor/`


### Train
we have migrated to **multi-gpu training** with **cross gpu batch normalization**, currently batch size of 32 on 4 GPUs is reported, other settings could be tried.

* train with 0.5x student
```
python train_multi_gpu.py --dataset KITTI --net ShuffleDet_conv1_stride1 --student 0.5 --train_dir xxx --image_set train --pretrained_model_path ./pretrained_model/shuffle_backbone0.5x/model-960000
```

* train with 0.25x student
```
python train_multi_gpu.py --dataset KITTI --net ShuffleDet_conv1_stride1 --student 0.25 --train_dir xxx --image_set train --pretrained_model_path ./pretrained_model/shuffle_backbone0.25x/model-665000
```
* you can turn off imitation by passing `--without_imitation True`, then the training is only with ground truth supervision,
like 
```
python train_multi_gpu.py --dataset KITTI --net ShuffleDet_conv1_stride1 --student 0.5 --train_dir xxx --image_set train --pretrained_model_path ./pretrained_model/shuffle_backbone0.5x/model-960000 --without_imitation True
```

>models will be saved in ```train_dir```

### Evaluation
By default, the evaluation code runs while training progress, test all checkpoint saved, after training has started, e.g., the 0.5x student training, you can run

```
python eval_model.py --dataset KITTI --net ShuffleDet_conv1_stride1 --eval_dir /path_to/eval_dir --image_set val --gpu 0 --checkpoint_path /path_to/train_dir --student 0.5
```
Then, tensorboard records can be loaded as(change port if needed)
```
tensorboard --logdir=/path_to/eval_dir --port 4118
```
and viewed by opening the site

```
http://localhost:4118
```

###
<table class="tg">
  <tr>
    <th class="tg-k19b" rowspan="2">Models</th>
    <th class="tg-k19b" rowspan="2">Flops<br>/G</th>
    <th class="tg-gom2" rowspan="2">Params<br>/M</th>
    <th class="tg-gom2" colspan="3">car</th>
    <th class="tg-gom2" colspan="3">pedestrian</th>
    <th class="tg-gom2" colspan="3">cyclist</th>
    <th class="tg-gom2" rowspan="2">mAP</th>
    <th class="tg-gom2" rowspan="2">ckpt</th>
  </tr>
  <tr>
    <td class="tg-gom2">Easy</td>
    <td class="tg-gom2">Mod</td>
    <td class="tg-gom2">Hard</td>
    <td class="tg-gom2">Easy</td>
    <td class="tg-gom2">Mod</td>
    <td class="tg-gom2">Hard</td>
    <td class="tg-gom2">Easy</td>
    <td class="tg-gom2">Mod</td>
    <td class="tg-gom2">Hard</td>
  </tr>
  <tr>
    <td class="tg-k19b">1x</td>
    <td class="tg-k19b">5.1</td>
    <td class="tg-gom2">1.6</td>
    <td class="tg-gom2">85.7</td>
    <td class="tg-gom2">74.3</td>
    <td class="tg-gom2">65.8</td>
    <td class="tg-gom2">63.2</td>
    <td class="tg-gom2">55.6</td>
    <td class="tg-gom2">50.6</td>
    <td class="tg-gom2">69.7</td>
    <td class="tg-gom2">51.0</td>
    <td class="tg-gom2">49.1</td>
    <td class="tg-tzpo">62.8</td>
    <td class="tg-gom2"><a href="https://drive.google.com/file/d/1jTTeyA61ZPDgwrvHeelysTQLUSWbIOul/view?usp=sharing">GoogleDrive</a></td>
  </tr>
  <tr>
    <td class="tg-k19b">0.5x</td>
    <td class="tg-k19b">1.5</td>
    <td class="tg-gom2">0.53</td>
    <td class="tg-gom2">81.6</td>
    <td class="tg-gom2">71.7</td>
    <td class="tg-gom2">61.2</td>
    <td class="tg-gom2">59.4</td>
    <td class="tg-gom2">52.3</td>
    <td class="tg-gom2">45.5</td>
    <td class="tg-gom2">59.7</td>
    <td class="tg-gom2">43.5</td>
    <td class="tg-gom2">42.0</td>
    <td class="tg-tzpo">57.4</td>
    <td class="tg-gom2"><a href="https://drive.google.com/file/d/1SYWeyRLbeDIQZzETKy0oqONe5hvepWqK/view?usp=sharing">GoogleDrive</a></td>
  </tr>
  <tr>
    <td class="tg-k19b" rowspan="2">0.5x-I</td>
    <td class="tg-k19b" rowspan="2">1.5</td>
    <td class="tg-gom2" rowspan="2">0.53</td>
    <td class="tg-gom2">84.9</td>
    <td class="tg-gom2">72.9</td>
    <td class="tg-gom2">64.1</td>
    <td class="tg-gom2">60.7</td>
    <td class="tg-gom2">53.3</td>
    <td class="tg-gom2">47.2</td>
    <td class="tg-gom2">69.0</td>
    <td class="tg-gom2">46.2</td>
    <td class="tg-gom2">44.9</td>
    <td class="tg-tzpo">60.4</td>
    <td class="tg-gom2"><a href="https://drive.google.com/file/d/1TyU7b957pRkD5PGHgoPy6803XAK0oTK3/view?usp=sharing">GoogleDrive</a></td>
  </tr>
  <tr>
    <td class="tg-oesp">+3.3</td>
    <td class="tg-oesp">+1.2</td>
    <td class="tg-oesp">+2.9</td>
    <td class="tg-oesp">+1.3</td>
    <td class="tg-oesp">+1.0</td>
    <td class="tg-oesp">+1.7</td>
    <td class="tg-oesp">+9.3</td>
    <td class="tg-oesp">+2.7</td>
    <td class="tg-oesp">+2.9</td>
    <td class="tg-oesp">+3.0</td>
    <td class="tg-186s"></td>
  </tr>
  <tr>
    <td class="tg-k19b">0.25x</td>
    <td class="tg-k19b">0.67</td>
    <td class="tg-gom2">0.21</td>
    <td class="tg-gom2">67.2</td>
    <td class="tg-gom2">56.6</td>
    <td class="tg-gom2">47.5</td>
    <td class="tg-gom2">54.7</td>
    <td class="tg-gom2">48.4</td>
    <td class="tg-gom2">42.1</td>
    <td class="tg-gom2">49.1</td>
    <td class="tg-gom2">33.3</td>
    <td class="tg-gom2">32.9</td>
    <td class="tg-gom2">48.0</td>
    <td class="tg-gom2"><a href="https://drive.google.com/file/d/1NLTUI7nJx6-6BjncxUx0nJ9vX5FqcS_C/view?usp=sharing">GoogleDrive</a></td>
  </tr>
  <tr>
    <td class="tg-k19b" rowspan="2">0.25x-I</td>
    <td class="tg-k19b" rowspan="2">0.67</td>
    <td class="tg-gom2" rowspan="2">0.21</td>
    <td class="tg-gom2">76.6</td>
    <td class="tg-gom2">62.3</td>
    <td class="tg-gom2">54.6</td>
    <td class="tg-gom2">56.8</td>
    <td class="tg-gom2">48.2</td>
    <td class="tg-gom2">42.6</td>
    <td class="tg-gom2">56.6</td>
    <td class="tg-gom2">37.3</td>
    <td class="tg-gom2">36.5</td>
    <td class="tg-gom2">52.4</td>
    <td class="tg-gom2"><a href="https://drive.google.com/file/d/18J350FX3MSTG5pgvXDXPfuvx0xxbDl1V/view?usp=sharing">GoogleDrive</a></td>
  </tr>
  <tr>
    <td class="tg-186s">+9.4</td>
    <td class="tg-186s">+5.7</td>
    <td class="tg-186s">+7.1</td>
    <td class="tg-186s">+2.1</td>
    <td class="tg-186s">-0.2</td>
    <td class="tg-186s">+0.5</td>
    <td class="tg-186s">+7.5</td>
    <td class="tg-186s">+4.0</td>
    <td class="tg-186s">+3.6</td>
    <td class="tg-186s">+4.4</td>
    <td class="tg-186s"></td>
  </tr>
</table>

>models with highest mAP are reported for both baseline and distilled model

>**Note** the numbers are different from the paper as they are independent running of the algorithm and we have migrated from single GPU training to multi-gpu training with larger batch size.

### Test with trained model

* For example, to test the 0.5x distilled model download the trained model at the corresponding [GoogleDrive](https://drive.google.com/file/d/1TyU7b957pRkD5PGHgoPy6803XAK0oTK3/view?usp=sharing) link, then run

```
python eval_model.py --dataset KITTI --net ShuffleDet_conv1_stride1 --eval_dir xxx --image_set val --gpu 0 --checkpoint_path /path_to/model0.5x60.4/model.ckpt-33000 --run_once True --student 0.5
```
* To test the 1x supervisor model, run

```
python eval_model.py --dataset KITTI --net ShuffleDet_conv1_stride1_supervisor --eval_dir xxx --image_set val --gpu 0 --checkpoint_path ./kitti-1x-supervisor/model.ckpt-725000 --run_once True
```

### Parameter counts
Note for model size, tensorflow saved checkpoint contains gradients/other information, so the size is larger than it should be, we have not yet freeze the model, to check model size, for exampel, the baseline 0.25x model without imitation, run

```
python param_count.py --model_path /home/wangtao/prj/shuffledet-multi-gpu-ckpt/model0.25x_nosup_48.0/model.ckpt-40000
```

### Flops counts
Still to come...

### Trouble shooting
* if you got permission denied when eval the model, please try 
```
chmod +x ./dataset_tool/kitti-eval/cpp/evaluate_object
```
if not work, just compile the evaluate_object excutable from source, i.e., run **make** under `./dataset_tool/kitti-eval`

## Citation
```
@inproceedings{wang2019distilling,
  title={Distilling Object Detectors With Fine-Grained Feature Imitation},
  author={Wang, Tao and Yuan, Li and Zhang, Xiaopeng and Feng, Jiashi},
  booktitle={Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition},
  pages={4933--4942},
  year={2019}
}
```
## Reference
[1] Zhaowei Cai, Quanfu Fan, Rogerio S Feris, and Nuno Vasconcelos. A unified multi-scale deep convolutional neural
network for fast object detection. ECCV 2016

## License
The code and the models are MIT licensed, as found in the LICENSE file.
