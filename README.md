# yolov5汉化版
## 简介
本仓库Fork自Ultralytics公司出品的yolov5，原仓库地址为：[ultralytics/yolov5](https://github.com/ultralytics/yolov5) ，所有版权均属于原仓库作者所有，请参见原仓库[License](https://github.com/ultralytics/yolov5/blob/master/LICENSE)。本人汉化自用，也方便各位国人使用。

#### 1. 模型效果
yolov5按大小分为四个模型yolov5s、yolov5m、yolov5l、yolov5x，这四个模型的表现见下图：

<img src="https://user-images.githubusercontent.com/26833433/90187293-6773ba00-dd6e-11ea-8f90-cd94afc0427f.png" width="1000">  

上图为基于5000张COCO val2017图像进行推理时，每张图像的平均端到端时间，batch size = 32, GPU：Tesla V100，这个时间包括图像预处理，FP16推理，后处理和NMS（非极大值抑制）。 EfficientDet的数据是从 [google/automl](https://github.com/google/automl) 仓库得到的（batch size = 8）。

#### 2. yolov5版本：

- 2021年1月5日：[v4.0 release](https://github.com/ultralytics/yolov5/releases/tag/v4.0)
- 2020年10月29日：[v3.1 release](https://github.com/ultralytics/yolov5/releases/tag/v3.1)
- 2020年8月13日: [v3.0 release](https://github.com/wudashuo/yolov5/releases/tag/v3.0)
- 2020年7月23日: [v2.0 release](https://github.com/wudashuo/yolov5/releases/tag/v2.0)
- 2020年6月26日: [v1.0 release](https://github.com/wudashuo/yolov5/releases/tag/v1.0)

  
**注意**：v2.0, v3.0, v3.1, v4.0权重通用，但不兼容v1.0，不建议使用v1.0，建议使用最新版本代码。


## 依赖
yolov5官方说Python版本需要≥3.8，但是我自用3.7也可以，但仍然推荐≥3.8。其他依赖都写在了[requirements.txt](https://github.com/wudashuo/yolov5/blob/master/requirements.txt) 里面。一键安装的话，打开命令行，cd到yolov5的文件夹里，输入：
```bash
$ pip install -r requirements.txt
```
pip安装慢的，请配置镜像源，下面是清华的镜像源。
```bash
$ pip install pip -U
$ pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
想配其他镜像源直接把网址替换即可，以下是国内常用的镜像源：
```yaml
豆瓣 https://pypi.doubanio.com/simple/
网易 https://mirrors.163.com/pypi/simple/
阿里云 https://mirrors.aliyun.com/pypi/simple/
腾讯云 https://mirrors.cloud.tencent.com/pypi/simple
清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
```
**注意**：
1. Windows版的PyTorch1.7已确定在检测图片视频时有问题，训练有无问题尚不清楚，请尽量在Linux或者MacOS使用本工程。如实在想用Windows，请降级PyTorch至1.6版本。
2. Windows版使用pip安装pycocotools需要有C++编译环境，也可以不安装pycocotools。
3. Windows上如果遇到`import cv2`错误，有可能是numpy版本有问题，降级到1.19.3可能会解决。

## 训练
#### 1. 快速训练/复现训练
下载 [COCO数据集](https://github.com/wudashuo/yolov5/blob/master/data/scripts/get_coco.sh)，然后执行下面命令。根据你的显卡情况，使用最大的 `--batch-size` ，(下列命令中的batch size是16G显存的显卡推荐值).
```bash
$ python train.py --data coco.yaml --cfg yolov5s.yaml --weights '' --batch-size 64
                                         yolov5m.yaml                           40
                                         yolov5l.yaml                       	24
                                         yolov5x.yaml                       	16
```
四个模型yolov5s/m/l/x使用COCO数据集在单个V100显卡上的训练时间为2/4/6/8天。
<img src="https://user-images.githubusercontent.com/26833433/90222759-949d8800-ddc1-11ea-9fa1-1c97eed2b963.png" width="900">
#### 2. 自定义训练
##### 2.1 准备标签
yolo格式的标签为txt格式的文件，文件名跟对应的图片名一样，除了后缀改为了.txt。
具体格式如下：
- 每个目标一行，整个图片没有目标的话不需要有txt文件
- 每行的格式为`class_num x_center y_center width height`
- 其中`class_num`取值为`0`至`total_class - 1`，框的四个值`x_center` `y_center` `width` `height`是相对于图片分辨率大小正则化的`0-1`之间的数，左上角为`(0,0)`，右下角为`(1,1)`
<img src="https://user-images.githubusercontent.com/26833433/91506361-c7965000-e886-11ea-8291-c72b98c25eec.jpg" width="900">
最终的标签文件应该是这样的：
<img src="https://user-images.githubusercontent.com/26833433/78174482-307bb800-740e-11ea-8b09-840693671042.png" width="900">

##### 2.2 数据规范
不同于DarkNet版yolo，图片和标签要分开存放。yolov5的代码会根据图片找标签，具体形式的把图片路径`/images/*.jpg`替换为`/labels/*.txt`，所以要新建两个文件夹，一个名为`images`存放图片，一个名为`labels`存放标签txt文件，如分训练集、验证集和测试集的话，还要再新建各自的文件夹，如图：
<img src="https://user-images.githubusercontent.com/26833433/83666389-bab4d980-a581-11ea-898b-b25471d37b83.jpg" width="900">

##### 2.3 准备yaml文件
自定义训练需要修改.yaml文件，一个是模型文件(可选)，一个是数据文件。
- 模型文件(可选):可以根据你选择训练的模型，直接修改`./models`里的`yolov5s.yaml` / `yolov5m.yaml` / `yolov5l.yaml` / `yolov5x.yaml`文件，只需要将`nc: 80`中的80修改为你数据集的类别数。其他为模型结构不需要改。
**注意** :当需要随机初始化时才会使用本文件，官方推荐使用预训练权重初始化。
- 数据文件:根据`./data`文件夹里的coco数据文件，制作自己的数据文件，在数据文件中定义训练集、验证集、测试集路径；定义总类别数；定义类别名称
    ```yaml
    # train and val data as 1) directory: path/images/, 2) file: path/images.txt, or 3) list: [path1/images/, path2/images/]
    train: ../coco128/images/train2017/
    val: ../coco128/images/val2017/
    test:../coco128/images/test2017/

    # number of classes
    nc: 80

    # class names
    names: ['person', 'bicycle', 'car', 'motorcycle', 'airplane', 'bus', 'train', 'truck', 'boat', 'traffic light',
            'fire hydrant', 'stop sign', 'parking meter', 'bench', 'bird', 'cat', 'dog', 'horse', 'sheep', 'cow',
            'elephant', 'bear', 'zebra', 'giraffe', 'backpack', 'umbrella', 'handbag', 'tie', 'suitcase', 'frisbee',
            'skis', 'snowboard', 'sports ball', 'kite', 'baseball bat', 'baseball glove', 'skateboard', 'surfboard',
            'tennis racket', 'bottle', 'wine glass', 'cup', 'fork', 'knife', 'spoon', 'bowl', 'banana', 'apple',
            'sandwich', 'orange', 'broccoli', 'carrot', 'hot dog', 'pizza', 'donut', 'cake', 'chair', 'couch',
            'potted plant', 'bed', 'dining table', 'toilet', 'tv', 'laptop', 'mouse', 'remote', 'keyboard', 
            'cell phone', 'microwave', 'oven', 'toaster', 'sink', 'refrigerator', 'book', 'clock', 'vase', 'scissors', 
            'teddy bear', 'hair drier', 'toothbrush']
    ```
##### 2.4 进行训练
训练直接运行`train.py`即可，后面根据需要加上指令参数，`--weights`指定权重，`--data`指定数据文件，`--batch-size`指定batch大小，`--epochs`指定epoch。一个简单的训练语句：
```bash
# 使用yolov5s模型训练coco128数据集5个epochs，batch size设为16
$ python train.py --batch 16 --epochs 5 --data ./data/coco128.yaml --weights ./weights/yolov5s.pt
```
#### 3. 训练指令说明
有参：
- `--weights` (⭐)指定权重，如果不加此参数会默认使用COCO预训的`yolov5s.pt`，`--weights ''`则会随机初始化权重
- `--cfg` 指定模型文件
- `--data` (⭐)指定数据文件
- `--hyp`指定超参数文件
- `--epochs` (⭐)指定epoch数，默认300
- `--batch-size` (⭐)指定batch大小，默认`16`，官方推荐越大越好，用你GPU能承受最大的`batch size`，可简写为`--batch`
- `--img-size` 指定训练图片大小，默认`640`，可简写为`--img`
- `--name` 指定结果文件名，默认`result.txt`        
- `--device` 指定训练设备，如`--device 0,1,2,3`
- `--local_rank` 分布式训练参数，不要自己修改！
- `--log-imgs` W&B的图片数量，默认16，最大100
- `--workers` 指定dataloader的workers数量，默认`8`
- `--project` 训练结果存放目录，默认./runs/train/
- `--name` 训练结果存放名，默认exp

无参： 
- `--rect`矩形训练
- `--resume` 继续训练，默认从最后一次训练继续
- `--nosave` 训练中途不存储模型，只存最后一个checkpoint
- `--notest` 训练中途不在验证集上测试，训练完毕再测试
- `--noautoanchor` 关闭自动锚点检测
- `--evolve`超参数演变
- `--bucket`使用gsutil bucket
- `--cache-images` 使用缓存图片训练
- `--image-weights` 训练中对图片加权重
- `--multi-scale` 训练图片大小+/-50%变换
- `--single-cls` 单类训练
- `--adam` 使用torch.optim.Adam()优化器
- `--sync-bn` 使用SyncBatchNorm，只在分布式训练可用
- `--log-artifacts` 输出artifacts,即模型效果
- `--exist-ok` 如训练结果存放路径重名，不覆盖已存在的文件夹
- `--quad` 使用四分dataloader



## 检测
推理支持多种模式，图片、视频、文件夹、rtsp视频流和流媒体都支持。
#### 1. 快速检测命令
直接执行`detect.py`，指定一下要推理的目录即可，如果没有指定权重，会自动下载默认COCO预训练权重模型。手动下载：[Google Drive](https://drive.google.com/open?id=1Drs_Aiu7xx6S-ix95f9kNsA6ueKRpN2J)、[国内网盘待上传](待上传)。 
推理结果默认会保存到 `./runs/detect`中。  
注意：每次推理会清空output文件夹，注意留存推理结果。
```bash
# 快速推理，--source 指定检测源，以下任意一种类型都支持：
$ python detect.py --source 0  # 本机默认摄像头
                            file.jpg  # 图片 
                            file.mp4  # 视频
                            path/  # 文件夹下所有媒体
                            path/*.jpg  # 文件夹下某类型媒体
                            rtsp://170.93.143.139/rtplive/470011e600ef003a004ee33696235daa  # rtsp视频流
                            http://112.50.243.8/PLTV/88888888/224/3221225900/1.m3u8  # http视频流
```
#### 2. 自定义检测
使用权重`./weights/yolov5s.pt`去推理`./data/images`文件夹下的所有媒体，并且推理置信度阈值设为0.5:

```bash
$ python detect.py --source ./data/images/ --weights ./weights/yolov5s.pt --conf 0.5
```

#### 3. 检测指令说明

自己根据需要加各种指令。

有参：
- `--source(⭐)`  指定检测来源
- `--weights` 指定权重，不指定的话会使用yolov5s.pt预训练权重
- `--img-size` 指定推理图片分辨率，默认640，也可使用`--img`
- `--conf-thres` 指定置信度阈值，默认0.4，也可使用`--conf`
- `--iou-thres` 指定NMS(非极大值抑制)的IOU阈值，默认0.5
- `--device` 指定设备，如`--device 0` `--device 0,1,2,3` `--device cpu`
- `--classes` 只检测特定的类，如`--classes 0 2 4 6 8`
- `--project` 指定结果存放路径，默认./runs/detect/
- `--name` 指定结果存放名,默认exp

无参：
- `--view-img` 图片形式显示结果
- `--save-txt` 输出标签结果(yolo格式)
- `--save-conf` 在输出标签结果txt中同样写入每个目标的置信度
- `--agnostic-nms` 使用agnostic NMS(前背景)
- `--augment` 增强识别，速度会慢不少。[详情](https://github.com/ultralytics/yolov5/issues/303)
- `--update` 更新所有模型  
- `--exist-ok` 若重名不覆盖


## 测试
#### 1.测试命令
首先明确，推理是直接检测图片，而测试是需要图片有相应的真实标签的，相当于检测图片后再把推理标签和真实标签做mAP计算。  
使用`./weights/yolov5x.pt`权重检测`./data/coco.yaml`里定义的测试集，图片分辨率设为672。
```bash
$ python test.py --weights ./weights/yolov5x.pt --data ./data/coco.yaml --img 672
```
#### 2.各指令说明
有参：
- `--weights(⭐)` 测试所用权重，默认yolov5sCOCO预训练权重模型
- `--data(⭐)` 测试所用的.yaml文件，默认使用`./data/coco128.yaml`
- `--batch-size` 测试用的batch大小，默认32，这个大小对结果无影响
- `--img-size` 测试集分辨率大小，默认640，测试建议使用更高分辨率
- `--conf-thres`目标置信度阈值，默认0.001
- `--iou-thres`NMS的IOU阈值，默认0.65
- `--task` 指定任务模式，train, val, 或者test,测试的话用`--task test`
- `--device` 指定设备，如`--device 0` `--device 0,1,2,3` `--device cpu`
- `--project` 指定结果存放路径，默认./runs/test/
- `--name` 指定结果存放名,默认exp

无参：
- `--single-cls` 视为只有一类
- `--augment` 增强识别
- `--verbose` 输出各个类别的mAP
- `--save-txt` 输出标签结果(yolo格式)
- `--save-hybrid` 输出标签+预测混合结果到txt
- `--save-conf` 保存置信度至输出文件中
- `--save-json` 保存结果为json
- `--exist-ok` 若重名不覆盖


## 联系方式
如有疑问可在[本repo的Issues](https://github.com/wudashuo/yolov5/issues)里提，我看到会及时回复。

如有代码bug请去[yolov5官方Issues](https://github.com/ultralytics/yolov5/issues)下提。

个人联系方式：<wudashuo@gmail.com>

## LICENSE
遵循yolov5官方[LICENSE](https://github.com/ultralytics/yolov5/blob/master/LICENSE)