---
date: 2020-06-12T14:35:00.000Z
layout: post
title: 基于C++/Python的CFD数据接口开发（一）Fluent Mesh
subtitle: Fluent
description: >-
    Fluent Mesh格式的解读与接口开发
image: >-
  ../article_img/2020-6-12-cfd-interface-1/title.jpg
optimized_image: >-
  ../article_img/2020-6-12-cfd-interface-1/title.jpg
category: CFD
tags:
  - CFD
  - Fluent
  - C++
  - Python
author: xiaoxiao
paginate: true
top: true
---
## 前言
&emsp;&emsp;**Fluent**是运用最为广泛的通用商业CFD软件，也是我CFD学习生涯接触的第一个CFD软件。长期以来，对Fluent的数据处理基本局限于**Tecplot**和**CFD-Post**，在后处理过程中，大量的精力浪费在软件反复的开启和导入导出、GUI操作和纷繁的Colorful Fluid Dynamics中。作为科研工具，我更希望获得对计算数据的充分掌握和提炼，并且回避大量手动操作（人活着就是为了自动化！）。

&emsp;&emsp;最早的想法是采用**Fluent**的journal脚本进行I/O操作，并采用Python进行文本提取。但是存在两个问题：
* journal文件(.jnl)包括TUI/GUI两类。通常，我们在**Fluent**的GUI界面直接录制的journal是**GUI**语言，也就是通过用户界面直接录制的操作宏。但是需要指出，**Fluent**在17版本以后全面贯彻落实了母公司ANSYS“科技源于换壳”的崭新理念（大误），基本上隔一个版本就换一套GUI，因此**GUI**型的journal文件跨版本往往不能互通。而**TUI**学习成本比较高，而且这个语言本质上还是**GUI**操作那样一层一层标签往下找，使用体验还是类似软件界面的点击而非CFD模型的建立，反观**Abaqus**和**LS-DYNA**一类的有限元软件采用的keywords脚本反而比较友好。而且按照以前阅读UDF手册和理论手册的经验，遇到一些冷门的功能估计手册还是讲不清，因此直接放弃。
* 每次开启**Fluent**都要浪费大量的时间启动证书和读取case和data，并占用大量的内存，甚至由于证书的莫名BUG存在启动失败的可能，可靠性并不高。对于没有**Fluent**的机器，还需要进行安装。
总之，这种方式不太适合团队形式的CFD计算及处理（除非你热衷于帮助每个课题组成员排雷）。

&emsp;&emsp;本文主要总结**Fluent User's Guide(Solution Mode)**附录B1中.msh文件的格式规范，并尝试对ASCII格式的.msh文件进行读取。由于主要工作集中在I/O操作，Python相较C++性能差异不大, 因此先采用Python。但是考虑到后续对数据灵活处理的需求，不排除采用C++重写或C++/Python混合编程形式的可能。

## Fluent Mesh格式解读
&emsp;&emsp;Fluent Mesh的书写格式主要应用于 *.msh* 和 *.cas* 两类文件中。*.msh* 是**Fluent**导入时指定的网格文件格式，*.cas* 则是同时包含网格信息和求解器信息的Case文件。在19.3版本之前，这两类格式均采用ASCII格式，能通过文本工具直接阅读。但在版本更新至2019R1（即19.3）后，**Fluent**为了追求I/O性能开始采用binary格式（即二进制）的文件——经由**Fluent Meshing**生成的 *.msh* 和计算模块保存的 *.cas* 直接打开均是乱码。**针对binary格式的读写我会放在下一篇文章中进行探讨。**
值得一提的是，**ICEM CFD**导出的 *.msh* 文件仍然是ASCII格式，可以导入至**Fluent**，但binary格式的 *.msh* 和 *.cas* 无法导回至**ICEM CFD**进行再处理（这软件真的有人在维护吗？）。

### 注释(Comment)
&emsp;&emsp;括号内编号为0时，后跟注释文本。
```
 (0 "comment text") 
```

### 文件头(Header)
&emsp;&emsp;括号内编号为1时，后跟文件头，用于指明文件来源和处理方式。
```
 (1 "fluent20.1.0 build-id: 0") 
```
PS: 官方文档似乎把1写错成了0...

### 维度(Dimensions)
&emsp;&emsp;括号内编号为2时，后跟维度 `ND`，取2或3。
```
 (2 ND) 
```

### 节点(Nodes)
&emsp;&emsp;括号内编号为10时，表示节点集。
```
(10 (zone-id first-index last-index type ND)(
   x1 y1 z1
   x2 y2 z2
   .
   .
   .
   )) 
```
**其中：**

`zone-id`&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;域ID，`zone-id = 0` 表明这是对计算域中的全部节点的一个属性声明，一般见于文件开头<br>
`first-index`&emsp;&emsp;&emsp;&ensp;起始节点编号（一般为1），十六进制<br>
`last-index`&emsp;&emsp;&emsp;&ensp;&ensp;结尾节点编号，十六进制<br>
`type`&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;默认为1<br>
`ND`&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;维度，取2或3

&emsp;&emsp;这是一个很典型的面向C语言的文本，因为原生的C没有类似C++中vector容器的动态数组，读取不确定量的数据前需要明确数据量来申请动态内存。文档也指出结尾节点编号必须大于或等于声明的节点数，否则会引发数组下标越界导致动态内存耗尽。

&emsp;&emsp;示例见下
```
  (10 (1 1 2d5 1 2)(
   1.500000e-01 2.500000e-02
   1.625000e-01 1.250000e-02
   .
   .
   .
   1.750000e-01 0.000000e+00
   2.000000e-01 2.500000e-02
   1.875000e-01 1.250000e-02
   )) 
```

### 周期边界(Periodic Shadow Faces)
&emsp;&emsp;括号内编号为18时，表示周期边界。
```
(18 (first-index last-index periodic-zone shadow-zone)(
   f00 f01
   f10 f11
   f20 f21
   .
   .
   .
   )) 
```
**其中：**

`first-index`&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;起始边界编号（取1），十六进制。<br>
`last-index`&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;结尾编号，，十六进制，比实际的周期边界数大就可以了。<br>
`periodic-zone` `shadow-zone`&emsp;&ensp;周期边界对的两个 `zone-id`

&emsp;&emsp;示例见下
```
  (18 (1 2b a c) (
   12 1f
   13 21
   ad 1c2
   .
   .
   .
   )) 
```

### 单元(Cells)
&emsp;&emsp;单元的前置编号为12，书写格式见下。
```
 (12 (zone-id first-index last-index type element-type)) 
```
**其中：**

`zone-id`&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;单元所在的域ID，`zone-id = 0` 表明这是对计算域中的全部单元的一个属性声明，一般见于文件开头<br>
`first-index`&emsp;&emsp;&emsp;起始单元编号（一般为1），十六进制<br>
`last-index`&emsp;&emsp;&emsp;&ensp;结尾单元编号，十六进制<br>
`type`&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;`type = 1`表示单元为激活状态，`type = 32`表示单元为未激活状态<br>
`element-type` &emsp;&emsp;单元类型，其取值可见下表

<table>
  <thead>
    <tr>
      <th>element-type</th>
      <th>description</th>
      <th>nodes/cell</th>
      <th>faces/cell</th>
    </tr>
  </thead>
  <tfoot>
  </tfoot>
  <tbody>

    <tr align="middle">
    <td>0</td>
    <td align="left">mixed(混合)</td>
    <td>-</td>
    <td>-</td>
    </tr>

    <tr align="middle">
    <td>1</td>
    <td align="left">triangular(三角形)</td>
    <td>3</td>
    <td>3</td>
    </tr>

    <tr align="middle">
    <td>2</td>
    <td align="left">tetrahedral(四面体)</td>
    <td>4</td>
    <td>4</td>
    </tr>

    <tr align="middle">
    <td>3</td>
    <td align="left">quadrilateral(四边形)</td>
    <td>4</td>
    <td>4</td>
    </tr>

    <tr align="middle">
    <td>4</td>
    <td align="left">hexahedral(六面体)</td>
    <td>8</td>
    <td>6</td>
    </tr>

    <tr align="middle">
    <td>5</td>
    <td align="left">pyramid(四棱锥)</td>
    <td>5</td>
    <td>5</td>
    </tr>

    <tr align="middle">
    <td>6</td>
    <td align="left">wedge(三棱柱)</td>
    <td>6</td>
    <td>5</td>
    </tr>

    <tr align="middle">
    <td>7</td>
    <td align="left">polyhedral(多面体)</td>
    <td>NN</td>
    <td>NF</td>
    </tr>
  </tbody>
</table>

&emsp;&emsp;对于混合形式(mixed)的单元集合，需要在后跟第二个括号以声明每个单元的类型，见下（一个三角形和四边形混合的网格体系）。
```
  (12 (9 1 3d 0 0)(
   1 1 1 3 3 1 1 3 1
   .
   .
   .
   ))
```

### 面(Faces)
&emsp;&emsp;面的前置编号为13，格式见下。
```
 (13 (zone-id first-index last-index bc-type face-type)) 
```
**其中：**

`zone-id`&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;面所在的域ID，`zone-id = 0` 表明对计算域中全部面的一个属性声明，一般见于文件开头<br>
`first-index`&emsp;&emsp;&emsp;起始面编号（一般为1），十六进制<br>
`last-index`&emsp;&emsp;&emsp;&ensp;结尾面编号，十六进制<br>
`bc-type`&emsp;&emsp;&emsp;&ensp;&emsp;&ensp;面所在域对应的边界条件编号，见下表<br>
`face-type`&emsp;&emsp;&emsp;&emsp;面类型，见下表

<table>
  <thead align="left">
    <tr>
      <th>bc-type</th>
      <th>description</th>
    </tr>
  </thead>
  <tfoot>
  </tfoot>
  <tbody>
    <tr align="middle">
    <td>2</td>
    <td align="left">interior</td>
    </tr>

    <tr align="middle">
    <td>3</td>
    <td align="left">wall</td>
    </tr>

    <tr align="middle">
    <td>4</td>
    <td align="left">pressure-inlet, inlet-vent, intake-fan</td>
    </tr>

    <tr align="middle">
    <td>5</td>
    <td align="left">pressure-outlet, exhaust-fan, outlet-vent</td>
    </tr>

    <tr align="middle">
    <td>7</td>
    <td align="left">symmetry</td>
    </tr>

    <tr align="middle">
    <td>8</td>
    <td align="left">periodic-shadow</td>
    </tr>

    <tr align="middle">
    <td>9</td>
    <td align="left">pressure-far-field</td>
    </tr>

    <tr align="middle">
    <td>10</td>
    <td align="left">velocity-inlet</td>
    </tr>

    <tr align="middle">
    <td>12</td>
    <td align="left">periodic</td>
    </tr>

    <tr align="middle">
    <td>14</td>
    <td align="left">fan, porous-jump, radiator</td>
    </tr>

    <tr align="middle">
    <td>20</td>
    <td align="left">mass-flow-inlet, mass-flow-outlet</td>
    </tr>

    <tr align="middle">
    <td>24</td>
    <td align="left">interface</td>
    </tr>

    <tr align="middle">
    <td>31</td>
    <td align="left">parent (hanging node)</td>
    </tr>

    <tr align="middle">
    <td>36</td>
    <td align="left">outflow</td>
    </tr>

    <tr align="middle">
    <td>37</td>
    <td align="left">axis</td>
    </tr>

    </tbody>
  </table>

&emsp;&emsp;文档指出，由非共形网格的相交产生的面存储在单独的面区域中，在该区域中bc-type会提升1000（例如，1003是wall区域）。不过我估摸着应该用不太上...
<table>
  <thead align="left">
    <tr>
      <th>face-type</th>
      <th>description</th>
      <th>nodes/face</th>
    </tr>
  </thead>
  <tfoot>
  </tfoot>
  <tbody>
    <tr align="middle">
    <td>0</td>
    <td align="left">mixed</td>
    <td>-</td>
    </tr>

    <tr align="middle">
    <td>2</td>
    <td align="left">linear</td>
    <td>2</td>
    </tr>

    <tr align="middle">
    <td>3</td>
    <td align="left">triangular</td>
    <td>3</td>
    </tr>
    
    <tr align="middle">
    <td>4</td>
    <td align="left">quadrilateral</td>
    <td>4</td>
    </tr>

    <tr align="middle">
    <td>5</td>
    <td align="left">polygonal</td>
    <td>NN</td>
    </tr>
  </tbody>
  </table>

&emsp;&emsp;`zone-id` 非0时（即该行数据并非声明），需要定义面的数据，其格式见下。
```
n0 n1 n2 c0 c1
```
其中，`n*` 是面所存储的节点， `c*` 是面相邻的单元。**针对3D模型**，单元的先后排序按照右手准则：如果按照节点的顺序卷曲右手的手指，则拇指将指向c0单元。**针对2D模型**，定义向量 $$ \vec{r}=\vec{n_1}-\vec{n_0} $$，定义向量 $$ \vec{k} $$ 为从n0出发，指向观察者一侧的外法线，则$$\vec{r} \times \vec{k} $$指向c1单元。

&emsp;&emsp;如果面的类型为混合 (`face-type = 0`) 或多边形 (`face-type = 5`)，则需要在每行开头声明节点数量（见下）。
```
x n0 n1 ... nf c0 c1 
```

&emsp;&emsp;在**Fluent Mesh**文件的格式定义中，所有有效节点、单元及面的编号均为正数。因此，对于边界上的面，其只与一个单元相邻，则将对应的c0或c1写作0。对于只存在面网格的3D模型，则c0和c1均为0。