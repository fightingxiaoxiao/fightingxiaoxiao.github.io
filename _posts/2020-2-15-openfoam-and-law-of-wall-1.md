---
date: 2020-02-15T13:18:00.000Z
layout: post
title: 壁面函数及其OpenFOAM实现（一）
subtitle: 壁面函数的基本概念
description: >-
    壁面函数的基本概念
image: ../article_img/2020-2-15-openfoam-and-law-of-wall-1/title.jpg
optimized_image: ../article_img/2020-2-15-openfoam-and-law-of-wall-1/title.jpg
category: CFD
tags:
  - CFD
  - OpenFOAM
author: Xiaoxiao
paginate: true
---
## 一些概念

<p style="text-indent:2em">大量试验表明，对于充分发展的湍流流动，在沿壁面法线的不同高度上，可将流动划分为近壁区和核心区。核心区的流动可认为是完全湍流区，而在近壁区，流体运动受壁面的影响比较明显，其可以分成三个子层：粘性底层、过渡层和对数层(<b>Fig. 1</b>)。</p>
<p style="text-indent:2em">粘性底层满足</p>
<center>$$u^{+}=y^{+}$$</center>
<p style="text-indent:2em">对数层的无量纲形式可写作</p>
<center>$$u^{+}=\frac{1}{\kappa} \ln y^{+}+C^{+}$$</center>
<p style="text-indent:2em">式中的无量纲数可表示为</p>
<center>$$y^{+}=\frac{y u_{\tau}}{\nu}, \quad u_{\tau}=\sqrt{\frac{\tau_{w}}{\rho}} \quad \text , \quad u^{+}=\frac{u}{u_{\tau}}$$</center>

其中，<br>
$$y^{+}$$  是距壁面高度值$$y$$处理后的无量纲值<br>
$$y^{+}$$  是平行于壁面的速度$$u$$处理后的无量纲值<br>
$$\tau_{w}$$  是壁面剪切应力(wall shear stress)<br>
$$\rho$$  是流体密度<br>
$$u_{\tau}$$  是剪切速度(friction velocity/shear velocity)<br>
$$\kappa$$  是卡曼常数(Von Kármán constant)，一般取0.40~0.42<br>
$$C^{+}$$  为常数，一般取5.0<br>

<center><embed src="../article_img/2020-2-15-openfoam-and-law-of-wall-1/law_of_the_wall.svg" style="display:block;width:400px;height:400px" /></center>
<center><b>Fig. 1</b> 壁面率</center>