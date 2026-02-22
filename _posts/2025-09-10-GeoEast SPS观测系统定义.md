---
layout: post
title: 'GeoEast SPS观测系统定义'
subtitle: '定观'
date: 2025-09-10
categories: tech
author: yucol
cover: 'https://images.unsplash.com/photo-1529322365446-6efd62aed02e?w=1600&q=900'
cover_author: 'inma santiago'
cover_author_link: 'https://unsplash.com/@inmasantiago'

tags: GeoEast SPS 观测系统
music-id: 1459852255
---


## 观测系统定义

**观测系统定义**是指将地震数据与辅助数据进行道头合并；辅助数据是指描述地震数据属性的数据，通常是文本格式的数据，比如SPS(Shell Processing Support)，班报电子表格及P190等等数据。

GeoEast处理软件进行观测系统定义，完成对数据道头更新，使其具有相应的属性信息，比如炮点及检波点线号、桩号，站号及XY坐标，CMP线号、CMP号及其XY坐标等信息，同时将炮点、检波点、CMP点等相关信息写入数据库当中。

**完成观测系统定义的基本步骤主要包括以下六项：**
* 数据加载及检查。
* 建立正确的炮点表格，检波点表格以及关系文件信息表格。
* 建立工区网格，一个三维工区需建立一个网格 ；二维工区，每条线需要建立一个独立的网格。
* 面元化处理，确定网格原点位置，网格大小及面元中心点位于炮检中心的位置。
* 数据道头更新，将炮检点信息，CMP信息写入相应的道头，并将这三方面的信息写入数据库中。
* QC 检查观测系统定义是否正确。


## 数据准备

通过绿山软件对炮点、检波点和关系文件进行整合，从而建观生成**SPS**文件，具体流程见[GeoScribeⅡ绿山软件建观](https://yucol.top/2025/09/02/GeoScribe%E2%85%A1%E7%BB%BF%E5%B1%B1%E8%BD%AF%E4%BB%B6%E5%BB%BA%E8%A7%82.html)。


## 定观

点击选中需要定观的**Swath**或者**Line**（未选中的话，后面的流程无法识别会报错），**Processing**-->**Geometry**-->**SPS**打开SPS定义界面，选择File-->Open，选择已经创建好的 **.sps** 文件，*.sps*、*.rps*、*.xps*文件会自动导入-->**OK**。

> * Auto Input：当Auto Input选项为选中状态时（缺省状态），文件名处于关联状态，输入其中一个文件名时，另两个文件名同步改变。例如输入炮点文件时，检波点文件和关系文件名也会同步改为相同的文件名（仅后缀不同）。若该选项未选中Auto Input时，则需要分别填入炮点、检波点、关系文件名。
> * 在导入数据的时候，关系文件的**Field Record Number**默认列范围是8-11，共四位数。但有时候数据的文件号可能是其他位数的数（例如五位）。这种情况下需要手动修改：**File Format: Custom...**-->**Ralation File**-->**Field Record Number**，将其修改为正确的列范围值。


![SPS_1.png](https://free.picui.cn/free/2025/09/11/68c2188fcc6c2.png)
![SPS_2.png](https://free.picui.cn/free/2025/09/11/68c2188faf70a.png)

数据导入成功后，可以选择**Check**-->**Batch**来检查炮点和检波点之间的关系是否正确，如果准确无误会提示*Check completed, noerrors found.*。选择**Grid**-->**Gridding**-->**检查Group1参数无误后**-->**Get Group2**-->**Apply**-->**Save to DB**，根据Group1的数据来计算出Group2的数值，应用并保存到数据库中。
![SPS_3.png](https://free.picui.cn/free/2025/09/11/68c2188fbb26d.png)

之后选择**Run**-->**Bin**-->**OK**，其数值会自动计算。运行完后结果如下。
![SPS_4.png](https://free.picui.cn/free/2025/09/11/68c2188fe3fa1.png)
![SPS_5.png](https://free.picui.cn/free/2025/09/11/68c2188fa2b95.png)

选择**File**-->**Project**-->**Seismic Data**，可以选择需要更新或删除的地震数据中已有的观测系统信息。其中**Clear Geometry**为删除数据库内已有的观测系统信息（含炮点、检波点、CMP点及CCP点的信息）：若选在工区上，则删除该工区内所有信息；若选在测线上，则删除该条测线内信息。该项功能一般用于观测系统重置的情况。

然后选择**Run**-->**Update**-->**OK**进行数据更新，其数值同样会自动计算。（更新前需要选中被更新的**Project**）
![SPS_6.png](https://free.picui.cn/free/2025/09/11/68c218c110423.png)



## 其他

* **Gridding**

  * 在创建工区网格**Gridding**时，三维情况下，工区网格只能定义一个网格，且将网格信息保存在数据库，网格定义时需要将工区边界束线的SPS文件同时加载，或者加载工区全部SPS文件进行三维网格定义，完成网格定义后，可以在主控工具栏base map上进行查看。*二维情况则需要每条测线定义网格*。

  * **Gridding**窗口中，**Inline Azimuth**测线方位角（角度），Inline线与正东方向的夹角，该参数由程序自动计算得到。**Coordinate System**坐标系统，左手系或右手系（请参考手册）。**Interval**：间隔，单位为m，Line线距和CMP点距，该参数由程序自动计算，需要检查与观测系统设计是否相符。
  * 检查Group1的参数无误后，点击**Get Group2**根据第一组参数计算出第二组参数。通过Apply 绘制出网格。也可以通过**Load from DB** 从数据库获得网格信息（数据库里已经有网格信息情况下使用）。**Apply**之后，一般情况下，网格大小合适，不需要进行调整，特殊情况，可以通过**Adjust**进行网格调整，调整网格原点位置，使炮检中点尽量分布在网格中心位置。然后，**Save to DB**将已绘制的网格的参数存入数据库。**Merge with DB**参数是合并数据库已有网格和当前网格，合并后的网格置为当前网格，数据库网格不变（仅当三维工区时显示）。
  * 如果给定的CMP间距在网格定义时不合适，造成同一个网格里面有两组炮检中心点，此时该网格需要重新定义，需要重新回到网格定义界面。
  * 有的时候如果炮检中心点不在网格中心位置，此时可以适当调整网格位置，调整方法分为自动调整和手动调整两种方式。如图手动调整0.5面元，网格调整正确后点击Apply, 再点击save to DB,将网格信息存入到数据库中。
![SPS_7.png](https://free.picui.cn/free/2025/09/11/68c218c1106a8.png)

* **Run**
  * Bin…计算共中心点面元
  * Update…更新地震数据卷头、头块，输出炮点、检波点、CMP点信息到数据库。
  * 点击Run 选择bin,自动从库里得到四点坐标，选择OK 得到CMP面元图。
  * 网格的四角顶点标记为**0、1、2、3**：**0->1为Inline**；方向**0->2 Crossline**方向。
![SPS_8.png](https://free.picui.cn/free/2025/09/11/68c218c088a83.png)

* **Update**
  * Target 更新内容
  * Seismic Data 开关按钮，选中表明将要对地震数据头块进行更新。
  * Database 开关按钮，选中表明将要把炮点、检波点、CMP等信息写入数据库中。
  * Match Method 更新地震数据时，SPS文件和道头的炮检点的匹配方法.
  * FFID+Channel 按野外文件号和通道号匹配
  * Line+Point 按线号和点号匹配（仅供导航及海上数据使用）
  * Reset Trace Type(H64) 更新地震数据时，是否重置地震道类型（H64）
  * Not in SPS->Invalid 若地震道炮检点不在SPS文件中，则置为无效道
  * In SPS:Invalid->Valid 若无效地震道的炮检点在SPS文件中，则置为有效道。
  * Renumber Source 是否重置炮号。若选中，则按照炮线号、桩号顺序对地震数据中的炮号（H89）重新顺序编号。
  * Start Source 重置炮号时采用的起始炮号（二维测线时使用）勾选RenumberSource 选项时才有效
