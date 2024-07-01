# 无人机定位数据集制作

主要涉及以下内容：
* **通过ODM将拍摄的图像转换成正射地图**
* **通过QGIS将正射地图与谷歌地图对齐并裁剪**
### ODM
OpenDroneMap (ODM) 是一个开源的软件项目，用于处理从无人机获取的图像数据，并生成高质量的地理空间信息产品。它可以被用于多种应用，包括农业、测量、环境监测、考古学等。以下是对 OpenDroneMap 的详细介绍：
* 图像拼接（Orthophoto Generation）：
        ODM 能够将无人机拍摄的重叠图像拼接成高分辨率的正射影像（orthophoto），这是一种经过几何校正的图像，具有统一的尺度和真实的地理位置。

* 三维模型重建（3D Reconstruction）：
        该软件能够将二维图像转换为三维模型，包括点云、网格和纹理模型。这对于地形分析、建筑物测绘等应用非常有用。

* 数字表面模型（DSM）和数字高程模型（DEM）生成：
        ODM 可以生成 DSM 和 DEM，分别表示地表物体和地形的高程数据。

* 地理信息系统（GIS）集成：
        生成的数据可以轻松集成到各种 GIS 软件中，以进行进一步的分析和应用。

* 多光谱和热成像数据处理：
        除了可见光图像，ODM 还支持多光谱和热成像数据的处理，适用于农业和环境监测等领域。

此外， ODM支持跨平台，可以运行在多种操作系统上，包括 Linux、Windows 和 macOS，方便不同用户的使用需求，提供了自动化的图像处理工作流程，用户只需提供原始图像数据，软件将自动完成从图像拼接到三维建模的所有步骤。

ODM项目github地址：[https://github.com/OpenDroneMap/ODM](https://github.com/OpenDroneMap/ODM)
ODM项目文档：[https://docs.opendronemap.org/](https://docs.opendronemap.org/)


#### 快速开始

使用docker运行ODM, 首先拉取镜像
~~`sudo docker pull opendronemap/opendronemap` , 此镜像已弃用~~

拉取镜像
```bash
sudo docker pull opendronemap/odm
```
!!! note   拉取这个镜像需要代理服务器



```bash
xujg@xujg-ps:~$ sudo docker images
REPOSITORY                  TAG       IMAGE ID       CREATED        SIZE
opendronemap/odm            latest    e3d462eb2c32   26 hours ago   1.61GB
hello-world                 latest    feb5d9fea6a5   2 years ago    13.3kB
opendronemap/opendronemap   latest    8c89143494c7   5 years ago    3.04GB

```
示例数据集在： [ https://github.com/OpenDroneMap/odm_data ]( https://github.com/OpenDroneMap/odm_data )

例如pull aukerman数据集
在仓库根目录下运行以下命令：
```bash
sudo docker run -it --rm \
    -v "$(pwd)/images:/code/images" \
    -v "$(pwd)/odm_orthophoto:/code/odm_orthophoto" \
    -v "$(pwd)/odm_texturing:/code/odm_texturing" \
    opendronemap/opendronemap
```
#### 输入数据
无人机拍摄的重叠图像。这些图像可以是普通的**可见光图像**、**多光谱图像**或**热成像图像**。图像应具有足够的重叠度，通常建议 60-80% 的前向重叠（沿飞行方向）和侧向重叠（垂直于飞行方向）。
#### 输出数据
正射影像 (Orthophoto)：

    高分辨率、地理校正后的正射影像，通常以 GeoTIFF 格式存储。

三维模型 (3D Model)：

    包括点云（Point Cloud）、网格模型（Mesh）和纹理模型（Textured Model），通常以 .obj 或 .ply 格式存储。

数字表面模型 (DSM) 和 数字高程模型 (DEM)：

    表示地表和地形的高程数据，通常以 GeoTIFF 格式存储。

!!! danger    需要指出的是，如果输入图像数据没有EXIF信息， 则需要加地理信息， 否则表面纹理映射，数字高程模型，正射图像都没办法得到


==默认情况下，在项目根目录中加入`geo.txt`， 此时ODM会自动检测到它，如果它有其他名称，则需要指定==


命令示例：
```bash
sudo docker run -it --rm     \
-v "/home/xujg/code/data_odm_50:/datasets"      \
opendronemap/odm     /datasets

```

输出：
```text
该过程完成后，结果将按如下方式组织：

|-- images/
    |-- img-1234.jpg
    |-- ...
|-- opensfm/
    |-- see mapillary/opensfm repository for more info
|-- odm_meshing/
    |-- odm_mesh.ply                    # A 3D mesh
|-- odm_texturing/
    |-- odm_textured_model.obj          # Textured mesh
    |-- odm_textured_model_geo.obj      # Georeferenced textured mesh
|-- odm_georeferencing/
    |-- odm_georeferenced_model.laz     # LAZ format point cloud
|-- odm_orthophoto/
    |-- odm_orthophoto.tif              # Orthophoto GeoTiff
```


### 通过QGIS将正射地图与谷歌地图对齐并裁剪
要将生成的正射影像与谷歌地图对齐并裁剪成小块，可以按照以下步骤进行：

#### 将正射影像与谷歌地图对齐
使用QGIS等GIS软件，将正射影像与谷歌地图对齐。以下是详细步骤：

##### 1.安装QGIS
下载链接[https://www.qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu](https://www.qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu)
使用教程: [https://www.osgeo.cn/qgisdoc/docs/user_manual/index.html](https://www.osgeo.cn/qgisdoc/docs/user_manual/index.html)
###### 导入正射影像
打开QGIS
```text
点击“图层”->“添加图层”->“添加栅格图层”。
选择您的正射影像文件（如 odm_orthophoto.tif）并点击“打开”。
```
###### 添加谷歌地图底图
```text
在QGIS中，点击“插件”->“管理和安装插件”。
搜索并安装“QuickMapServices”插件。
安装完成后，点击“Web”->“QuickMapServices”->“OSM”->“Google”->“Google Satellite”。
```
>如果OSM中没有Google， 可以点击OSM中的setting->More services进行下载

###### 可以切换成中文，并设置代理服务器
##### 对齐正射影像

###### 重投影Geo-TIFF文件
```text
右键该文件-> 打开属性 -> 信息 -> 查看坐标参考系CRS 
栅格-> 投影 -> 变形（重投影：
在弹出的窗口中，设置以下参数：
    Input Layer（输入图层）：选择你的栅格图层。
    Source CRS（源CRS）：选择当前栅格数据的CRS。
    Target CRS（目标CRS）：选择EPSG:3857。
    Resampling method（重采样方法）：选择适合你的数据的重采样方法（例如，Bilinear 或 Cubic）。
    Output file（输出文件）：选择保存重投影栅格的路径和文件名
```

1. 裁剪成小块
在QGIS中裁剪
打开配准后的正射影像。
使用“裁剪（Clip）”工具将影像裁剪成所需的小块。
导出裁剪后的影像。
批量裁剪
如果需要批量裁剪，可以使用Python脚本或GDAL工具进行自动化裁剪。

使用Python脚本裁剪
安装GDAL库

```python
from osgeo import gdal
import os

def crop_image(input_path, output_dir, tile_width, tile_height):
    dataset = gdal.Open(input_path)
    band = dataset.GetRasterBand(1)
    xsize = band.XSize
    ysize = band.YSize
    geotransform = dataset.GetGeoTransform()
    projection = dataset.GetProjection()

    tile_num = 0
    for i in range(0, ysize, tile_height):
        for j in range(0, xsize, tile_width):
            tile_num += 1
            output_path = os.path.join(output_dir, f'tile_{tile_num}.tif')
            gdal.Translate(output_path, dataset, srcWin=[j, i, tile_width, tile_height],
                           format='GTiff', outputSRS=projection, outputBounds=geotransform)

input_path = 'path_to_your_orthophoto.tif'
output_dir = 'path_to_output_tiles'
tile_width = 256  # 设置每块的宽度
tile_height = 256  # 设置每块的高度

crop_image(input_path, output_dir, tile_width, tile_height)
```

3. 检查裁剪结果
使用QGIS或其他GIS软件检查裁剪后的图块，确保它们正确对齐和裁剪。

总结
使用QGIS对齐正射影像与谷歌地图。
使用QGIS或Python脚本裁剪正射影像成小块。
检查裁剪结果，确保准确性。