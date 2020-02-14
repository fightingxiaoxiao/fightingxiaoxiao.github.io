---
date: 2020-02-15T13:18:00.000Z
layout: post
title: 壁面函数及其OpenFOAM实现（一）
subtitle: 壁面函数的基本概念
description: >-
    壁面函数的基本概念
image: ../article_img/2020-2-14-openfoam-study-1/title.jpg
optimized_image: ../article_img/2020-2-14-openfoam-study-1/title.jpg
category: CFD
tags:
  - CFD
  - OpenFOAM
author: Xiaoxiao
paginate: true
---
<p sytle="text-indent:2em">大量试验表明，对于充分发展的湍流流动，在沿壁面法线的不同高度上，可将流动划分为近壁区和核心区。核心区的流动可认为是完全湍流区，而在近壁区，流体运动受壁面的影响比较明显，其可以分成三个子层：粘性底层、过渡层和对数层。</p>
<p>粘性底层满足</p>
$$u^{+}=y^{+}$$
<p>对数层的无量纲形式可写作</p>
$$u^{+}=\frac{1}{\kappa} \ln y^{+}+C^{+}$$
<p>其中的无量纲数可表示为</p>
$$y^{+}=\frac{y u_{\tau}}{\nu}, \quad u_{\tau}=\sqrt{\frac{\tau_{w}}{\rho}} \quad \text { 与 } \quad u^{+}=\frac{u}{u_{\tau}}$$
<embed src="../article_img/2020-2-15-openfoam-and-law-of-wall-1/law_of_the_wall.svg" type="image/svg+xml" />
