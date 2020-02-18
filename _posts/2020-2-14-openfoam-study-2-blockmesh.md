---
date: 2020-02-18T16:46:05.000Z
layout: post
title: OpenFOAM初探(二)
subtitle: 基本环境配置
description: >-
    blockMesh详解
image: ../article_img/2020-2-14-openfoam-study-1/title.jpg
optimized_image: ../article_img/2020-2-14-openfoam-study-1/title.jpg
category: CFD
tags:
  - CFD
  - OpenFOAM
author: Xiaoxiao
paginate: true
---

### blockMesh命令
&emsp;&emsp;blockMesh是OpenFOAM中最基本的网格生成器之一，依赖于字典文件blockMeshDict。该程序以块(block)的思想来构造六面体(hexahedron)网格，类似于ICEM的结构化网格功能。blockMesh的主函数代码位于 /applications/utilities/mesh/generation/blockMesh/blockMesh.C 。

用法(Usage):<br>
&emsp;&emsp;&emsp; **blockMesh** [OPTION]

选项(Options):<br>
&emsp;&emsp;&emsp;  **-case** &lt;dir&gt; <br>
&emsp;&emsp;&emsp; 指定案例所在目录，若缺省，则为当前路径。

&emsp;&emsp;&emsp;  **-dict** &lt;filename&gt; <br>
&emsp;&emsp;&emsp; 从指定路径读取blockMeshDict。

&emsp;&emsp;&emsp;  **-region** &lt;name&gt; <br>
&emsp;&emsp;&emsp; 只生成指定区域的网格。

&emsp;&emsp;&emsp;  **-blockTopology** <br>
&emsp;&emsp;&emsp; 将块的边缘和中心输出为.obj文件，以便进行可视化。

&emsp;&emsp;&emsp;  **-noFunctionObjects** <br>
&emsp;&emsp;&emsp; 跳过functionObjects的执行

### 字典文件的书写
&emsp;&emsp;`blockMesh` 指令依赖于字典文件blockMeshDict，该字典文件可位于
* system/blockMeshDict
* system/&lt;region&gt;/blockMeshDict
* constant/polyMesh/blockMeshDict
* constant/&lt;region&gt;/polyMesh/blockMeshDict

字典文件的内容包括：
#### 文件头(File Header)
&emsp;&emsp;文件头用于申明字典类型。
```
FoamFile
{
    version        2.0;
    format         ascii;
    class          dictionary;
    location      "constant/polyMesh";
    object         blockMeshDict;
}
```

#### 尺度缩放
&emsp;&emsp;关键词`convertToMeters` 用于定义几何尺度，一般均转换为国际单位-米。
```
convertToMeters 1.0;

```
#### 顶点(vertices)
&emsp;&emsp;顶点关键词用于指定block顶点坐标，编号从0开始。

```
vertices
(
    (0 0 0)
    (1 0 0)
);
```