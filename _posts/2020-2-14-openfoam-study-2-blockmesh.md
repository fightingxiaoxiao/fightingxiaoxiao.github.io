---
date: 2020-02-18T16:46:05.000Z
layout: post
title: OpenFOAM初探（二）
subtitle: blockMesh详解
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

## blockMesh命令
&emsp;&emsp;`blockMesh` 是OpenFOAM中最基本的网格生成器之一，依赖于字典文件blockMeshDict。该程序以块(block)的思想来构造六面体(hexahedron)网格，类似于ICEM的结构化网格功能。blockMesh的主函数代码位于 /applications/utilities/mesh/generation/blockMesh/blockMesh.C 。关于该功能的详细介绍可以查阅[Mesh generation with blockMesh](https://cfd.direct/openfoam/user-guide/v7-blockmesh/#x26-1850005.3)

用法(Usage):

```
blockMesh [OPTION]
```
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


## 字典文件的书写
&emsp;&emsp;`blockMesh` 指令依赖于字典文件blockMeshDict，该字典文件可位于
* system/blockMeshDict
* system/&lt;region&gt;/blockMeshDict
* constant/polyMesh/blockMeshDict
* constant/&lt;region&gt;/polyMesh/blockMeshDict<br>

字典文件的内容包括：
### 文件头(File Header)
&emsp;&emsp;文件头用于申明字典类型。
```
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      blockMeshDict;
}
```
### 尺度缩放
&emsp;&emsp;关键词 `convertToMeters` 用于定义几何缩放尺度，一般均转换为国际单位-米。
```
convertToMeters 1.0;
```
### 顶点定义
&emsp;&emsp;顶点关键词用于指定block顶点坐标，顶点编号从0开始。

```
vertices
(
    (0. 0. 0.)
    (1. 0. 0.)
);
```

### 块定义
&emsp;&emsp;block是用八个顶点定义的六面体。第一个括号内书写八个顶点编号；第二个括号内书写x,y,z三个方向上的网格节点总数；simpleGrading后的括号可简单地定义节点分布律，表示x,y,z三个方向上的梯度率(*Grading Ratio*, 第一个网格尺寸/最后一个网格尺寸)(**Fig. 1**)。
```
blocks
(
    hex (0 1 2 3 4 5 6 7) (10 10 10) simpleGrading (2 2 1)
);
```

&emsp;&emsp;`blockMesh` 同样也可以定义比较复杂的节点分布律。以下展示的脚本中，对block的y方向定义了3个子block,对每个子block施加不同的节点分布律。每个子括号的第一个数字代表子block在该方向上的几何尺寸权重，第二个数字代表子block在该方向上的节点数量权重，第三个数字代表梯度率。
```
blocks
(
    hex (0 1 2 3 4 5 6 7) (10 10 10) 
    simpleGrading ( 1
                    ((2 2 0.5)(3 6 0.2)(1 2 0.6))
                    1
                  )
);
```

### 曲边
&emsp;&emsp;此部分用于定义block中的曲线边缘，如几何中没有曲线可以将括号中内容缺省。[Mesh generation with blockMesh](https://cfd.direct/openfoam/user-guide/v7-blockmesh/#x26-1850005.3)中有比较详细的介绍，此处不表。
```
edges
(
    arc 1 5 (1.0 0.0 0.5)
);
```

### 边界定义
&emsp;&emsp;边界的定义是CFD分析的重要步骤。其结构包括：
* 说明边界名称 —— inlet, outlet, any_name_u_like... 
* 边界类型 —— patch表示后续需要赋予明确数值的边界，比如出入口；wall当然就是壁面；symmetry就是对称边界；其余定义尚在探索。
* 几何面 —— 表示该边界包含的block面。
对于未定义的所有外露面，`blockMesh` 在执行过程中会有警告，并将这些面合并成一个集合。
```
boundary               // keyword
    (
        inlet              // patch name
        {
            type patch;    // patch type for patch 0
            faces
            (
                (0 4 7 3)  // block face in this patch
            );
        }                  // end of 0th patch definition
        outlet             // patch name
        {
            type patch;    // patch type for patch 1
            faces
            (
                (1 2 6 5)
            );
        }
        walls
        {
            type wall;
            faces
            (
                (0 1 5 4)
                (0 3 2 1)
                (3 7 6 2)
                (4 5 6 7)
            );
        }
    );
```

### 合并表面对
&emsp;&emsp;用于把两个不完全共节点的面熔接，反正对我而言没啥用，缺省。具体操作依然可见[Mesh generation with blockMesh](https://cfd.direct/openfoam/user-guide/v7-blockmesh/#x26-1850005.3)(没错我就是这么懒:-))。
```
mergePatchPairs
(
);
```


## 宏语法和动态编译（参数化）
&emsp;&emsp;对于超过1个blcok的字典文件，顶点数超过了10个，后期修改简直要了懒人们的命。万幸的是，OpenFOAM的字典文件提供宏语法，比如在开头申明 `x 1.;` ，那在后续定义数字类参数时就可以用 `$x` 来表示。而对于需要进行数学计算的部分，可以采用 `#calc` 植入简单的计算语句：
```
Y        5;
Y_left   #calc "-0.5*$Y";
```
&emsp;&emsp;对于更复杂的需求，当然也可以直接植入C++代码片段。当然放这写显得杀鸡焉用牛刀，先给个[链接](https://cfd.direct/openfoam/user-guide/v7-basic-file-format/#x17-1230004.2)。


## 总结
&emsp;&emsp;`blockMesh` 从本质上讲就是一个脚本化的结构化网格生成器，对于一些简单模型，免去了在GUI上点来点去的麻烦，参数化的支持也能一定程度上改善几何参数化分析时的前处理效率。同时， `blockMesh` 在很多算例中都用于生成一个大流域，以进行下一步 `sanppyHexMesh` 的精细化非结构网格处理。