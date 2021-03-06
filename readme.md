# Kaggle State Farm 项目总结

## 1  项目分析

### [1.1   项目概述（引用自Kaggle）](https://www.kaggle.com/c/state-farm-distracted-driver-detection)

在据CDC机动车安全事业部统计，在所有车祸当中约有五分之一的车祸是由司机分心（如打电话）造成的，这意味者由于驾驶员分心导致每年有425000人受伤，3000死亡。

![](E:\Programming\Python\5 CV\statefarm\总结\bb.jpg)

为改善这个为题，State Farm希望通过一个2D的仪表板摄像头自动检测驾驶员是否处于分心驾驶的状态，为此State Farm提供一个由仪表板拍摄的驾驶员驾驶状态的数据集，希望能够区分驾驶员是否处于分心驾驶的状态，驾驶员10个状态如下所示，数据集分为训练集和测试集，数量分别为22424以及79726。

•   c0: 安全驾驶  

•   c1: 右手打字  

•   c2: 右手打电话  

•   c3: 左手打字  

•   c4: 左手打电话  

•   c5: 调收音机  

•   c6: 喝饮料   

•   c7: 拿后面的东西  

•   c8: 整理头发和化妆  

•   c9: 和其他乘客说话  

### 1.2  评价指标

Kaggle平台的评价指标为多分类对数损失即交叉熵，公式如下所示：

![](E:\Programming\Python\5 CV\statefarm\总结\aa.jpg)

### 1.3  问题分析

数据集分析

不论任何项目，应该都是对要数据集进行仔细分析，<font color=red>刚开始我拿到数据粗略看一下，就开始建模型分析，这种投机取巧，急于求成得态度和方法非常不可取，最终不仅无法获得令人满意的结果，还费时费力。</font>本项目的数据集有以下特点：  

- 训练集（22424张）远小于测试集（79726张），这种情况很容易过拟合

- 数据集中许多图片是相机拍摄的视频里面截取的不同帧，即同一司机同一状态下存在多张非常相似的图片，这里需要注意的如果验证集划分以图片id划分的话，会验证集里面许多图片其实在训练集已经训练过，验证集不具备参考意义！

- 数据集总共拍摄了26个司机的图片


## 2  数据预处理

在本次项目中图像预处理（本文默认预处理前都已将图片统一成相同尺寸）考虑了以下集中方法：

- a、不做任何预处理，使用Batch Normalization作为第一层，<font color=red>使用自建的小模型进行尝试没有效果后放弃</font>

  ![](E:\Programming\Python\5 CV\statefarm\总结\1.JPG)

  <center>图2.1 预处理方式a</center>

- b、将像素值缩放至-1至+1的范围内，keras自带的预训练模型默认方法，一开始使用的是该种方法，后期参考论坛和论文，发现大多数以减去均值为主，经实际训练对比后发现训练后的泛化性能没有c方法好，放弃使用。

- c、减去训练集图片的均值，经典卷积网络以及Kaggle论坛里面使用比较多的方法，**最终采用此方法**。

- d、模型容易过拟合，可能是由于模型学习到的是关于人的特征，因此考虑使用图像边缘检测做预处理，<font color=red>去除部分冗余特征如颜色</font>，但是时间问题没有进行尝试，以下是几种图像边缘检测预处理后的效果。

  ![](E:\Programming\Python\5 CV\statefarm\总结\2018-09-11_23-06-14.jpg)

  <center>图2.2 预处理方式d</center>

  以下是b、c、d预处理方式在同一个模型上训练（6 epoch）结果的对比<font color=red>由于时间问题对比试验次数和模型数量过少，模型较为简单，不做结论，仅供自己参考</font>

  e、增加方框用于强调某个区域，这个方法是参考网上有人人工对图像的某些区域进行框选增加模型的泛化能力，由于没有这个毅力，自己选择头部和右侧的区域进行框选，出发点是强调两个区域人的手是否出现及相对位置，实际来看效果不佳。

  ![](E:\Programming\Python\5 CV\statefarm\总结\7.JPG)

  <center>表2.1 自建模型2上不同预处理方式对比</center>   

  | 预处理方式                  | 训练集精度/验证集精度 | 训练集loss/验证集loss | 训练时长 | kaggle得分（private） |
  | --------------------------- | --------------------- | --------------------- | -------- | --------------------- |
  | 缩放至+/-1                  | 0.9973/0.6513         | 0.0110/1.3783         | 4803     | 1.39272               |
  | 减去均值                    | 0.9973/0.7313         | 0.0110/1.2528         | 4775     | 1.12028               |
  | sovel+canddy边缘检测+灰度图 | 0.9986/0.6257         | 0.0075/1.4683         | 4873     | 1.03105               |
  | sovel+canddy边缘检测        | 0.9992/0.6597         | 0.0049/1.0919         | 4835     | 0.99339               |
  | 增加图框                    | 0.9962/0.7503         | 0.0161/1.3360         | 8004     | 1.10281               |


### 验证集的划分

关于验证集的划分有两种观点：

一种是随机划分，这种情况验证集由于与训练集高度重合，不具备参考意义

二种是根据司机id划分，这种划分方式是的验证集能够比较真实的反映模型的效果，但是对模型性能的提高这一点有待确认

## 3  模型及训练

### 3.1 系统硬件

训练平台1：个人笔记本（前期使用），Win10(使用虚拟机Linux无法调用GPU)

**配置：**

​    CPU处理器：I7-7700HQ，主频2.8GHz,4核

​    内存：8G

​    显卡（GPU）: GeForce GTX 1050, 4GB, CUDA核数量 640

本次项目尝试了多个模型，主要分为自建的小模型，以及经典网络模型的训练

训练平台2： Linux服务器  Ubuntu 16 

配置：内存32G, 显卡（GPU）: GeForce GTX Titan X , 12GB, CUDA核数量 3584

### **3.2 模型**训练

模型的搭建和训练主要分为两个部分，一个是经典网络模型的微调，另外一个是自建网络模型的搭建和训练。针对经典网络模型的微调，主要问题如下：

1、VGG、ResNet、Inception等经典网络模型针对的任务场景与本项目区别较大，因此直接替换最后的全连接层，只训练最后一层的结果不佳，精度在0.11上下浮动。

2、存在比较严重的过拟合问题

![](E:\Programming\Python\5 CV\statefarm\总结\2.jpg)

<center>图3.1  VGG预训练模型</center>

自建网络的搭建，主要参考VGG网络模型，增加了BatchNormalization层，尝试了使用1X1的卷积和跳跃连接，以下分别是自建模型1和2的网络结构。

![](E:\Programming\Python\5 CV\statefarm\总结\3.jpg)

<center>图3.2 自建模型1</center>

![](E:\Programming\Python\5 CV\statefarm\总结\4.jpg)

<center>图3.3  自建模型2</center>

![](E:\Programming\Python\5 CV\statefarm\总结\4.jpg)

<center>图3.3  VGG with bn</center>

​       上图中的模型是在VGG的每个Block后面加一个Batch Normalization层，实际交叉验证训练效果不太好（没做严格的对比试验，见后面）。

 在学习率的选择中遇到一些问题，总结如下：

- 预训练模型网络与自建搭建的模型（不加载权重）在学习率方面的选择有比较大的区别，预训练模型的学习率通常应该选择比较小的值，VGG预训练模型中学习率在0.0001时能够较快的收敛，而使用RMSprop默认学习率0.001则会一直在较高的loss附近反复，难以收敛。

- 使用自建模型（即结构与VGG一致，权重随机初始化）的学习率使用RMSprop默认学习率能够较快收敛。

- 对于模型如果无经验确定合适的学习率，可以通过快速开始训练过程，观察loss在训练过程中是否下降，如无明显下降，可以考虑中断训练过程，使用较小的学习率

- 模型训练过程中发现训练集和验证集的损失和精度在最后几个epoch上下震荡，无法进一步收敛，推测为学习率的问题，使用keras自带的学习率调整回调函数ReduceLR后，模型在最后几个epoch能够较好的进一步收敛。

![](E:\Programming\Python\5 CV\statefarm\总结\6.jpg)

​        对于Batch size理论上来说过小会导致训练过程剧烈震荡，过大则会导致内存开销加大，合适的大小能够兼顾收敛速度、训练时间和对硬件的要求，以下是个人自己做的一番尝试，其中batch size为50由于未调整学习率导致未能收敛到合适位置，最终选取的是32作为训练的batch size。

| 序号 | optimizer | Learning rate | epochs | Time(s)/epoch | Batch size | Loss:   train/validate | Accuracy:   train/validate |
| ---- | --------- | ------------- | ------ | ------------- | ---------- | ---------------------- | -------------------------- |
| 1    | Adam      | 0.001         | 15     | 319           | 16         | 0.229/0.2678           | 0.9621/0.9615              |
| 2    | Adam      | 0.001         | 15     | 277           | 32         | 0.1875/0.1315          | 0.9672/0.9875              |
| 3    | Adam      | 0.001         | 15     | 274           | 50         | 0.0828/0.8843          | 0.9778/0.8036              |
| 4    | Adam      | 0.001         | 15     |               | 64         | 内存溢出，报错         |                            |
| 5    | RMSprop   | 0.001         | 15     | 263           | 32         | 0.0645/0.0944          | 0.9877/0.9869              |

 

针对模型的过拟合问题，主要尝试以下方法：

- 1、经典卷积网络上使用正则化，效果不明显。

- 2、使用数据增强（选择和偏移），对验证集精度有一定的提升，未在交叉验证中使用该方法。

- 3、S折交叉验证，对结果具有较好的提升效果，同一个模型使用交叉验证后得分有明显提高


### 3.3  训练结果

部分模型未在Jupyter  notebook中进行

| 序号 | 模型名称            | 模型大小 | 参数(万) | 验证集精度 | S折交叉验证 | S值  | 根据司机划分 | LB score（Private） |
| ---- | ------------------- | -------- | -------- | ---------- | ----------- | ---- | ------------ | ------------------- |
| 1    | Inception           | 216M     | 2182     | 0.9211     | 无          | 1    | 否           | 1.09030             |
| 2    | 自建模型-跳跃链接   | 18.7M    | 159.4    | 0.9949     | 无          | 1    | 否           | 1.03579             |
| 3    | 自建模型2           | 7.8M     | 88.8     | 0.9890     | 无          | 1    | 否           | 0.84384             |
| 4    | 自建模型2           | 7.8M*5   | 88.8     | 0.992      | 有          | 5    | 否           | 0.45749             |
| 5    | 自建模型2           | 7.8M*13  | 88.8     | 0.8612     | 有          | 13   | 是           | 0.41360             |
| 6    | VGG预训练           | 1.1G     | 13430    | 0.9931     | 无          | 1    | 否           | 0.78763             |
| 7    | VGG预训练           | 1.1G     | 13430    | 训练中     | 有          | 10   | 是           | 0.52304             |
| 8    | VGG with bn（试验） | 1.1G     | 13430    | 0.9939     | 有          | 5    | 是           | 0.89343             |
| 9    | InceptionV3         | 187M     |          | 0.8225     | 无          | 1    | 是           | 0.49177             |
| 10   | InceptionResNetV2   | 437M     |          | 0.8025     | 无          | 1    | 是           | 0.5360              |
| 11   | 3、6、9和10集成     | ——       | ——       | ——         | ——          | ——   | ——           | 0.38372             |







​    

### 3.4  总结

通过该项目的实践，以下几点是需要反思提高的地方：

- 不要急于搭建模型，而应该详细了解项目的要求、评价指标、特点，以及查找相关信息，盲目开始搭建模型和训练会造成时间和资源的浪费

- 针对项目，应该制定切实可行的计划如决定采用的数据预处理方式、模型、可视化方法，没有计划的项目实践导致做了很多工作，却无法做出总结和改进。

- 每一项尝试和改进都应该及时进行记录和改进。

- 模型过拟合是应为模型学习特征可能不是与结果相关联的，应该有一种方式能够去除不需要的信息，增加模型泛化能力。

**需要作出的改进**

  在Kaggle上了解其他人的思路，以及了解类似项目的实践经验后，考虑从以下方面进行改进：

- 尝试使用K近邻算法进行对结果进行提高（论坛经验）；
- 考虑对图像添加噪声（亮度、对比度，裁剪，标记）进行数据增强；
- 采用3个以上不同模型进行集成，增大S折交叉验证的折数