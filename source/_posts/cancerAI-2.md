---
layout: posts
title: 多模态数字病理图像（H&E 与多重荧光）配准进度报告2
date: 2025-07-12 21:10:55
tags:
---
本次的核心目标是，在两张成像原理、颜色、形态截然不同的图像（H&E与荧光）之间，找到一种可靠的自动对齐方法。

**实现方法一：基于稳定解剖结构的特征匹配策略**

1. **特征提取（核心创新点）：**

   - **H&E图像：** 利用胆管腔及背景区域在H&E图像中呈现为**高亮度白色**的特性，通过高阈值二值化，精确提取其轮廓。
   - **荧光图像：** 利用同一结构在荧光图像中呈现为**无信号黑色**的特性，通过低阈值反向二值化，提取其轮廓。

2. **特征优化：**

   - 使用形态学操作（核大小[kernel_size = 5]去除噪声，并过滤掉面积过小（[min_contour_area = 100]的伪影，保证了特征的纯净度。

3. **执行配准：**

   - 将上述提取并优化后的“特征掩码”作为输入，利用OpenCV的ECC（增强相关系数）算法，该算法对亮度线性变化不敏感，非常适合本场景。
   - 我系统性地测试了**平移、欧几里得（包含了平移、旋转和等比缩放）、仿射、透视**四种变换模型，并最终根据**相关系数（CC）**指标，选取了最优的变换模型。

   ![alt text](cancerAI-2-1.png)

   - **第一行：特征提取**
     - **左侧两张 (H&E)**: 显示了从原始H&E图中提取出的白色胆管区域（掩码）。
     - **右侧两张 (荧光)**: 显示了从原始荧光图中提取出的黑色胆管区域（掩码）。
   - **第二行：特征增强与叠加**
     - **左二 (增强特征)**: 展示了经过距离变换和边缘增强后的胆管特征图。这些图像的灰度变化更丰富，纹理更清晰，非常适合ECC算法进行配准。
     - **右二 (特征叠加)**: 将提取的胆管掩码（红色/绿色）叠加在原始图像上，直观地展示了程序识别的“锚点”位置是否准确。
   - **第三行：配准结果**
     - **配准后H&E胆管特征**: 这是将H&E的特征图根据计算出的矩阵进行变换后的样子。
     - **胆管特征差异图**: 将配准后的H&E特征图与荧光特征图相减得到的差异。图像越黑，说明差异越小，对齐效果越好。
     - **胆管特征棋盘格对比**: 将两张特征图拼接成棋盘格。如果对齐得好，您应该能看到胆管的结构在方格之间是连续的，没有明显的错位。
     - **胆管特征叠加效果**: 将两张特征图半透明地叠加在一起。如果对齐完美，它们的轮廓会高度重合，形成一个清晰的融合图像。

#### 核心成果与对后续工作的启发

**核心成果：**

我成功实现了局部图像的**全自动、高精度配准**。最终在`HOMOGRAPHY`（透视）模型下，获得了高达**0.9669**的相关系数（CC），质量评估为“优秀”。我得到了一份精确的**3x3变换矩阵**，它以数学形式描述了将H&E图像精准对齐到荧光图像上所需的全部几何变换（旋转、平移、缩放、透视）。

![alt text](cancerAI-2-2.png)
- `HOMOGRAPHY`是本次测试中**最复杂、最强大**的2D变换模型。它不仅包含平移、旋转、缩放、剪切（这些`AFFINE`模型也能做到），还能够模拟**透视畸变**——就像我们从一个倾斜的角度看一个矩形，它会变成梯形一样。
- **为什么是它？** 在所有四个模型中，`HOMOGRAPHY`取得了最高的**相关系数（0.966946）**，尽管只比次优的`AFFINE`模型（0.964276）高出一点点。这说明，在图像中可能存在极其微小的、非线性的形变（例如组织在切片或放置时产生的轻微褶皱或拉伸），而`HOMOGRAPHY`模型凭借其更高的自由度，捕捉到了这种细微的畸变，从而获得了最佳的匹配分数。

------

**实现方式二：基于互信息的图像配准方案**
我们不直接比较两张图的“颜色”（像素值），因为它们本来就不同。而是采用一种更聪明的策略——**互信息 (Mutual Information)**。
它巧妙地绕开了不同模态图像像素值无法直接比较的难题，转而从统计学的角度寻找最佳对齐，使得该方法具有广泛的适用性和鲁棒性。
#### 工作流程
1. **选择工具:**使用强大的医学图像处理库 `SimpleITK` 来执行这个任务。
2. **配置配准参数:** 详细设置配准过程中的每一个环节，包括：

- 用什么标准衡量对齐的好坏？**(Metric - 度量标准)**

- 如何“猜测”移动过程中非整数像素点的值？**(Interpolator - 插值器)**

- 用什么方法去寻找最佳的对齐位置？**(Optimizer - 优化器)**

- 允许对图像进行哪些类型的变换（移动、旋转、缩放）？**(Transform - 变换类型)**

3. **执行与应用:** 启动配准过程，找到最佳变换参数，并将这个变换应用到需要对齐的图像上，生成最终对齐好的图像。

#### 工具库介绍
1. `SimpleITK`
- 作用: 这是实现图像配准的核心工具。它是一个专门为处理和分析多维生物医学图像（2D, 3D, 4D）而设计的开源库。

- `SimpleITK` 就像一个装备齐全的**“专业地图对齐工作室”**。它里面不仅有尺子、量角器（用于**测量和变换**），还有一套完整的工作流程和自动化设备（**配准框架**），可以高效、精确地完成地图对齐任务。代码中的 `ImageRegistrationMethod` 就是这个工作室里的核心**自动化设备**。

2. `scikit-image` 

- 作用: 一个通用的图像处理库。在配准前后，你可能需要用它进行一些**预处理或后处理**，比如图像裁剪、降噪、格式转换等。

- 如果 SimpleITK 是专业工作室，那 scikit-image 就是一个非常方便的**“多功能工具箱”**。在把地图送进工作室之前，你可能需要用这个工具箱里的剪刀（裁剪）、清洁布（降噪）来整理一下地图。

3. `napari` 

- 作用: 一个为科学家设计的、交互式的多维数据可视化工具。在配准任务中，它的作用至关重要：可以非常直观地检查配准前和配准后的效果。你可以将配准前的两张图以不同颜色叠加显示，看到它们是错位的；配准后，再将固定图像和变形后的移动图像叠加显示，检查对齐的精度。

- `napari` 就像一个高精度的“数字灯箱”。你可以把两张地图（图像）作为不同的图层放上去，调节每个图层的透明度，放大缩小，来仔细比对河流和城市峡谷是否真的对齐了。没有它，你只能通过数字来判断效果，非常不直观。

4. `CycleGAN`

- 作用: 这是一种深度学习模型，属于生成对抗网络（GAN）的一种。它可以学习将一种风格的图像转换成另一种风格，例如，将H&E染色的图像“翻译”成看起来像是DAPI染色的图像。

-- 如果H&E和DAPI图像的差异巨大，以至于互信息这种统计方法也难以奏效时，可以先用CycleGAN将H&E图像伪染成“伪DAPI”图像。这样，配准任务就从“多模态”（H&E vs DAPI）变成了“单模态”（伪DAPI vs DAPI），难度会大大降低。

- 假设你的“地形图”是黑白素描风格，而“水文图”是彩色油画风格，差异太大导致对齐困难。CycleGAN 就像一位“艺术风格模仿大师”，可以学习素描和油画的风格，然后帮你把黑白的“地形图”重新绘制成一张彩色油画风格的“地形图”。这样你手里就有了两张风格相似的油画，对齐起来就容易多了。

#### 互信息
它不关心像素值的绝对大小，而是关心像素值分布的统计依赖性。
> 想象一下，一个只会说中文的人（代表H&E图像）和一个只会说英语的人（代表DAPI图像）被关在一个房间里。他们无法直接沟通（像素值无法直接比较）。
> 但是，如果每次房间里出现“苹果”（H&E图像中出现某个特定范围的像素值）时，那个说英语的人总会说出"Apple"（DAPI图像中对应位置也出现某个特定范围的像素值）。即使你听不懂"Apple"是什么意思，但你观察久了就会发现，“苹果”这个词的出现，可以很好地预测对方会说"Apple"这个词。

互信息就衡量的是这种“预测能力”有多强。如果一个信号（H&E的像素值）的出现，能极大地减少另一个信号（DAPI的像素值）的不确定性，那么它们之间的互信息就很高。

```python
import SimpleITK as sitk

# 导入图像文件：

# fixed_image: H&E苏木精通道 (作为参考)

# moving_image: DAPI通道 (需要被移动对齐的图像)



# 1. 实例化配准方法

registration_method = sitk.ImageRegistrationMethod()



# 2. 设置度量标准 (Metric) - Mattes Mutual Information

registration_method.SetMetricAsMattesMutualInformation(numberOfHistogramBins=50)//将图像的连续像素值0~255分成50个桶。通过统计两个图像中来自不同的“桶”的像素对出现的频率来计算互信息。

registration_method.SetMetricSamplingStrategy(registration_method.RANDOM)//随机采样部分像素来计算，可节省计算资源

registration_method.SetMetricSamplingPercentage(0.01)



# 3. 设置插值器 (Interpolator)

registration_method.SetInterpolator(sitk.sitkLinear)
//当我们对图像进行旋转缩放时，它的原始像素点的位置会发生变化。通过线性插值可以合理地猜测出这个新的位置应该时什么像素值。


# 4. 设置优化器 (Optimizer)

registration_methodSetOptimizerAsRegularStepGradientDescent(learningRate=1.0, minStep=0.001, numberOfIterations=200)



# 5. 设置变换类型 (Transform) - 从简单的刚性或仿射变换开始

# initial_transform = sitk.CenteredTransformInitializer(fixed_image, 

#                                                       moving_image, 

#                                                       sitk.Euler2DTransform(), 

#                                                       sitk.CenteredTransformInitializerFilter.GEOMETRY)

# registration_method.SetInitialTransform(initial_transform)



final_transform = sitk.Similarity2DTransform() # 仿射变换的一种

registration_method.SetInitialTransform(final_transform)



# 6. 执行配准

final_transform = registration_method.Execute(fixed_image, moving_image)



print(f"Final Transform: {final_transform}")

print(f"Optimizer stop condition: {registration_method.GetOptimizerStopConditionDescription()}")



# 7. 应用变换

resampler = sitk.ResampleImageFilter()

resampler.SetReferenceImage(fixed_image)

resampler.SetInterpolator(sitk.sitkLinear)

resampler.SetTransform(final_transform)



warped_moving_image = resampler.Execute(moving_image)
```