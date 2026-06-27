+++
date = '2026-06-21T17:36:15+08:00'
draft = true
title = 'OpenCV'
+++

# OpenCV

## 1.Preview

#### 1.Overview

***OpenCV is the world's biggest computer vision library***

#### 2.Install

[GitHub - opencv/opencv: Open Source Computer Vision Library · GitHub](https://github.com/opencv/opencv)

<img src="https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260621184850397.png" alt="image-20260621184848143" style="zoom: 67%;" />

<img src="https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260621185009861.png" alt="image-20260621185006263" style="zoom:80%;" />

#### 3.Configure environment variables

将YourPath\\opencv\build\bin与YourPath\opencv\build\x64\vc16\bin配置到Path环境变量当中

#### 4.Configure header files and library files in VS

![image-20260621185725167](https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260621185727454.png)

* 在VC++目录中配置包含目录和库目录,在包含目录中配置YourPath\opencv\build\include和在库目录中配置YourPath\opencv\build\x64\vc16\lib

![image-20260621190052297](https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260621190054146.png)

* 在链接器中的输入选项中的附加依赖项中配置opencv_world500d.lib,改值需要根据你的下载的opencv的版本为主，在YourPath\opencv\build\x64\vc16\lib中进行查看

#### 5.Example

```c++
1#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>
#include<string>
int main(int argc, const char* argv[]) {
	std::string path{ "C:/Users/weixiao/Pictures/Saved Pictures/yeqi.png" };
	cv::Mat img{ cv::imread(path) };
	cv::imshow("Image", img);
	cv::waitKey(0);
}
```

## 2.Chapter1



