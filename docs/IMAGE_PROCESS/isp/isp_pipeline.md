

# ISP Pipeline

## 1. RAW域处理

### 1.1 BLC（黑电平校正 Black Level Correction）

* **产生原因**

  * 传感器器件会产生暗电流，导致输出不是绝对0，影响真实电平值。
  * 模拟信号转换成数字信号时存在精度问题，无法区分很小的电平值。
* **作用**

  * 去除图像黑电平偏移，使RAW数据线性化，方便后续处理。
* **实现方法**

  * 固定值扣除法：对每个通道减去一个固定值（均值或中值）。
  * 可选对Gr、Gb通道归一化处理。

### 1.2 LSC（镜头阴影校正 Lens Shading Correction）

* **产生原因**

  * 镜头光学折射不均匀，造成四角偏暗等阴影效应。
* **实现方法**

  * 网格法：将图像划分为固定大小的网格，计算每个网格的增益，利用插值计算每个像素的增益并补偿。

### 1.3 DPC（动态坏点校正 Defective Pixel Correction）

* **产生原因**

  * 制造工艺缺陷导致某些像素异常，如全黑环境下的白点或高亮环境下的黑点。
* **实现方法**

  * 通过算法检测坏点，基于周围正常像素插值滤波修正坏点。

### 1.4 AWB（自动白平衡 Automatic White Balance）

* **产生原因**

  * 传感器对不同光源无颜色恒常性，导致图像色偏。
* **常用算法**

  * **灰度世界法**

    * 假设图像RGB三通道均值相等，通过计算R、G、B均值得到增益值，调整各通道。
  * **完美反射法（白点法）**

    * 选取图像中亮度最高的像素作为白点，计算各通道增益进行校正。
  * **统计法、基于机器学习的AWB**（可选）

### 1.5 BNR（Bayer降噪 Bayer Noise Reduction）

* **产生原因**

  * 传感器受温度、暗光、传输等影响产生噪声，且ISP处理可能放大噪声。
* **作用**

  * 在RAW域去噪比后期YUV域降噪效果更好，因为线性变化小，易处理。
* **常用算法**

  * 空域法：中值滤波、均值滤波、高斯滤波、双边滤波。
  * 频域法：傅里叶变换、离散余弦变换（DCT）、小波变换。
  * 图像自相似性法：非局部均值（NLM）、BM3D。

### 1.6 HDR（高动态范围处理）

* **作用**

  * 合成多张不同曝光的RAW图像，扩展图像动态范围。
* **实现方法**

  * 多曝光合成，局部对比度增强等。

### 1.7 Demosaic（去马赛克）

* **产生原因**

  * 将Bayer RAW图像转成完整的RGB图像。
* **常用算法**

  * 线性插值法：用缺失颜色的邻近像素线性插值。
  * 色比法：假设邻近像素的色比（R/G，B/G）恒定，先插值G通道，再推算缺失通道。
  * 色差法：基于色彩差异进行插值，更准确。
  * 高级方法：基于边缘检测、频域滤波、深度学习的去马赛克算法。

---

## 2. RGB域处理

### 2.1 CCM（色彩校正矩阵 Color Correction Matrix）

* **产生原因**

  * 人眼对不同色彩的响应不同，需对传感器捕获的色彩进行校正。
* **实现方法**

  * 使用3x3矩阵对RGB三通道进行线性变换，调整色彩。
  * 通过拍摄标准色卡进行矩阵标定。

### 2.2 Gamma校正

* **产生原因**

  * 人眼对亮度感知非线性，而传感器线性响应。
* **实现方法**

  * 使用查找表（LUT）或线性插值实现非线性映射，使图像符合人眼视觉特性。

### 2.3 CSC（色彩空间转换 Color Space Conversion）

* **产生原因**

  * 视频编码、显示设备等通常使用YUV色彩空间。
* **实现方法**

  * 将RGB转换成YUV（或YCbCr）色彩空间，方便后续压缩与处理。

---

## 3. YUV域处理

### 3.1 YNR（亮度降噪 Luma Noise Reduction）

* **产生原因**

  * 图像亮度通道的噪声影响细节和视觉效果。
* **常用算法**

  * 结合空域滤波和频域滤波，采用自适应滤波、引导滤波等。

### 3.2 UVNR（色度降噪 Chroma Noise Reduction）

* **产生原因**

  * 虽经过白平衡，RGB三通道仍存在随机色噪，特别在色度分量。
* **实现方法**

  * 利用色度通道特性进行噪声抑制，通常比亮度降噪强度低，避免色彩失真。

### 3.3 EE（边缘增强 Edge Enhancement）

* **产生原因**

  * 光学系统的低通滤波作用导致图像细节锐度下降。
* **实现方法**

  * 使用Unsharp Mask（USM）提取高频分量，加强边缘明暗反差，提升视觉锐度。

### 3.4 Warping（去畸变 Distortion Correction）

* **产生原因**

  * 镜头畸变（径向桶形、枕形畸变及切向畸变）导致图像几何失真。
* **标定方法**

  1. 收集标定图像（棋盘格等）
  2. 获取角点三维坐标与像素坐标
  3. 求内外参数及畸变系数（使用最小二乘法、极大似然法优化）
  4. 生成畸变矫正映射表
* **算法**

  * 利用畸变模型计算校正坐标，采用双三次插值获得目标像素值。

---

# 备注及补充

* ISP Pipeline具体实现会因厂商不同有所差异。
* 部分环节可能有多个算法版本，选择需考虑性能与效果平衡。
* 现代ISP还会包含自动曝光（AE）、自动对焦（AF）、色调映射（Tone Mapping）、锐化、伽马校正后的色彩增强等。
* 深度学习算法开始被引入ISP后端以提升降噪和去马赛克等效果。

