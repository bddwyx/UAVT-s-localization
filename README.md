# UAVT-s-localization

## 构建方法
* 安装Boost, Eigen3
* 编译OpenCV
* 编译AprilTag
* 安装相应的相机驱动
* （方法1）在根目录执行如下操作
``` shell
mkdir build
cd build
cmake ..
make -j8
```
* （方法二）直接利用CLion载入CmakeLists.txt，即可分别构建不同的可执行文件

## 使用方法
在根目录下的Bin目录中存放着编译后的可执行文件

### Acquisition
相机图像与视频采集，并按照系统时间自动存放归类。代码中选择相机并执行，即可看到相机的实时预览效果。在图像模式与视频模式下，操作模式略有不同。
#### 图像模式
按下s会保存当前采集图片，并按照日期与时间自动命名，按照日期归档到根目录下的data文件夹。关闭预览并不能停止程序，只有按下q键时程序会停止。
#### 视频模式
程序首先采集一张图像，并根据这张图像的尺寸与颜色信息开始后续的录制。按下q键后，程序终止，并将视频按照起始时刻时间命名，归档至相应的文件夹。

### Calibration（已暂停使用）
读取指定路径下的全部图片，提取标定板的角点并进行展示。在遍历目录下的图片后，根据提取的角点信息对相机进行标定，并将标定参数写入intrinsics.txt与dis_coeff.txt两个文件中。

### Localization
利用代码中指定的相机，（可选）读取文件中的内参并进行畸变校正，并识别图片中的AprilTag的位置和编号。

## 开发指南
### 设计思路
#### 模块化
所有可能的图像来源和对应的算法均按照其特性进行了封装，存储于Inc文件夹中的hpp文件。在需要特定来源的图像数据时，仅需要从对象中读取并获得图像；在需要算法处理时，构造算法类时指定输入来源，算法类即可自动输出图像。

当存在层级依赖关系时（例如多个MindVision相机共用一套SDK），代码中采用的单例模式能够避免多次重复初始化造成的错误，且能够利用合适时间的对象构造和生命周期结束时自动调用的析构函数统一接口，实现不同图像来源能够方便互换与扩展数量的效果。

对于相机类，统一只提供GetFrame接口，返回指针，指向存放图像的cv::Mat。
