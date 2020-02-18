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
&emsp;&emsp;blockMesh是OpenFOAM中最基本的网格生成器之一，依赖于字典文件blockMeshDict。该程序以块(block)的思想来构造六面体(hexahedron)网格，类似于ICEM的结构化网格功能。blockMesh的主函数代码位于 **/applications/utilities/mesh/generation/blockMesh/blockMesh.C** 。<br>
用法(Usage):<br>
&emsp;&emsp;&emsp; **blockMesh** [OPTION]<br>
选项(OPTION):<br>
&emsp;&emsp;&emsp;  **-case** <directory><br>
&emsp;&emsp;&emsp; 指定案例所在目录，若缺省，则为当前路径。<br>
&emsp;&emsp;&emsp;  **-dict** <filename><br>
&emsp;&emsp;&emsp; 从指定路径读取blockMeshDict。<br>
&emsp;&emsp;&emsp;  **-region** <name><br>
&emsp;&emsp;&emsp; 只生成指定区域的网格。<br>
&emsp;&emsp;&emsp;  **-blockTopology**<br>
&emsp;&emsp;&emsp; 将块的边缘和中心输出为.obj文件，以便进行可视化。<br>
&emsp;&emsp;&emsp;  **-noFunctionObjects**<br>
&emsp;&emsp;&emsp; 跳过functionObjects的执行<br> 

### 字典文件的书写

