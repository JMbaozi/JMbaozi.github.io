---
layout: post
title: 'GeoScribeⅡ绿山软件建观'
subtitle: '观测系统'
date: 2025-09-02
categories: tech
author: yucol


tags: 绿山软件 地震处理 GeoScribeⅡ
music-id: 2120442088
---

## 数据预处理

在建观的过程中需要三个数据文件，分别是 **检波点数据、炮点数据、关系数据** 其后缀/名称一般设置为 **.r、.s、.x** 。初始数据一般是excel或csv格式文件，首先对数据进行预处理，主要是对 **检波点数据** 进行检查，对其中的 **炮点号** 后列➖前列的数值进行筛选，因为道间距一般为10m，所以相邻两列数据之间的间隔为10（如果不为10，则说明可能存在问题）：

* 如果值为 **10/-10** 则说明正常
* 值为 **0** 且线号值相同，则说明重复，需要进行删除
* 值为 **-20/20** 且线号值相同，则说明两条数据间缺少一条数据，需要进行补齐，补齐时只需要补 **线号、炮号** , **坐标值 x、y、z** 值在软件中对其进行插值（x、y使用线性插值，z使用空间差值）
* 值为 **-30/30** 且线号值相同，处理方式与上述相同，其他值以此类推。一般情况下，异常的最大差值不会超过 **50** 
* 检查无误后将所有数据 **右对齐** ，坐标值一般保留两位小数（做到统一，三位亦可），另存为 **.prn** 格式文件供后续使用

> 上述差值属于异常的前提是两条数据的线号值相同（说明两者属于同一组数据），如果不相同，则表明两条数据都是正常的。

![GeoScribe2_3.png](https://youke1.picui.cn/s1/2025/09/02/68b6b9e22b1f0.png)

## 软件初始

打开 **GeoScribe2** 软件后，在 **Startup** 中的设置如下，其中最大通道数可按照实际情况设置（高于手头数据的数目，后续在软件中也可对此进行修改），如果有现成的DB，则直接选择 **Open Existing Database** 

![GeoScribe2_1.png](https://youke1.picui.cn/s1/2025/09/02/68b6b9e0f21e5.png)

软件启动后，会有两个设置的弹窗，其中 **Station Spreadsheet** 需要用到 **检波点** 文件， **Source Spreadsheet** 需要用到 **炮点、关系数据** 文件。

![GeoScribe2_2.png](https://youke1.picui.cn/s1/2025/09/02/68b6b9e211695.png)

## 建观步骤

### 1.设置检波点数据

首先设置 **Station Spreadsheet** ，点击 **IMPORT** --> **Import Window..** --> **OPEN DATA FILE** 选择 **检波点数据** ，其中需要设置的参数有：

* Line Number：线号
* Station：检波点号
* X Coordinate：X坐标
* Y Coordinate：Y坐标
* Z Coordinate：Z坐标

![GeoScribe2_4.png](https://youke1.picui.cn/s1/2025/09/02/68b6b9e2081f9.png)

依次选中参数所对应的数据，然后使用 **DEFINE DATA TYPE** 定义数据类型，全部定义完成后，点击 **SET FIRST DATA LINE** 和 **SET LAST DATA LINE** ，最后点击 **CONFIGURATION COMPLETE.EXIT** 完成数据导入。

在配置完成后的弹窗中点击 **Combination Options** --> 勾选 **Combine Line and Station Numbers** --> **Line Multiplier** 设置为炮点数值位数的 **10** 倍（例如炮点数值最大值为4位数，则值设置为5位数，即10000）。

> **CONFIGURATION FILE** 可以对当前的参数选取设置进行保存，当再次导入格式相同的文件时可以直接套用设置（一定是格式相同）。

### 2.设置关系文件数据

在 **Source Spreadsheet** 中进行导入，其中两个导入按钮，分别应用于步骤2和3，此步骤2的设置为 **IMPORT** --> **Import Window..** --> **OPEN DATA FILE** 选择 **关系数据** ，其中需要设置的参数有：

* Record Num：文件号
* Source：炮号
* Line Number：线号（炮点）

> 设置文件号时可以查看原始数据（segd）的文件名样式，数值范围可能是4位或其它位数（本数据为5位），具体情况具体分析。

![GeoScribe2_5.png](https://youke1.picui.cn/s1/2025/09/02/68b6b9e1e1315.png)

依次选中参数所对应的数据，然后使用 **DEFINE DATA TYPE** 定义数据类型，全部定义完成后，点击 **SET FIRST DATA LINE** 和 **SET LAST DATA LINE** ，最后点击 **CONFIGURATION COMPLETE.EXIT** 完成数据导入。

在配置完成后的弹窗中选择 **Match on Record Number** ,点击 **Combination Options** --> 勾选 **Combine Line and Source Numbers** --> **Line Multiplier** 设置为炮点数值位数的 **10** 倍（例如炮点数值最大值为4位数，则值设置为5位数，即10000）。

### 3.设置关系文件数据Patterns

> 导入数据的时候先选中需要导入的数据，再IMPORT文件，例如本步骤需要导入Patterns，则先选中 **Patterns** 列，再进行下面的步骤。

步骤三中依旧导入关系文件数据，但这次使用另外一个导入按钮，具体设置为 **IMPORT PATTERNS** --> **Import Window** --> **Import Window..** --> **OPEN DATA FILE** 选择 **关系数据** ，其中需要设置的参数有：

* From Chan：炮点首道
* To Chan：炮点尾道
* From Sta：检波点首道
* To Sta：检波点尾道
* Source：炮点号
* Record Number：文件号
* Station Line：检波点线号
* Source Line：炮点线号

依次选中参数所对应的数据，然后使用 **DEFINE DATA TYPE** 定义数据类型，全部定义完成后，点击 **SET FIRST DATA LINE** 和 **SET LAST DATA LINE** ，最后点击 **CONFIGURATION COMPLETE.EXIT** 完成数据导入。

在配置完成后的弹窗中选择 **Match on Record Number** ,点击 **Combination Options** --> 勾选 **Combine Line and Station Numbers** 和 **Combine Line and Source Numbers** --> **Line Multiplier** 设置为炮点数值位数的 **10** 倍（例如炮点数值最大值为4位数，则值设置为5位数，即10000）。

数据导入后，对其进行筛选，首先对Pattern进行排序：选中 **Pattern** 列 --> **Edit** --> **Sort...** ，然后选中所有 **Pattern** 为空值的数据，全部删除 --> **Edit** --> **Delete Rows** ，然后再选中 **Record Num** 列对其排序，将数据设置回规范状态。

![GeoScribe2_6.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba492b4ad.png)

![GeoScribe2_7.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba491f657.png)

### 4.设置炮点数据

选中 **X/Y/Z** 坐标列，导入炮点文件。 **IMPORT** --> **Import Window..** --> **OPEN DATA FILE** 选择 **炮点数据** ，其中需要设置的参数有：

* Source：炮号
* Line Number：线号
* X Coordinate：X坐标
* Y Coordinate：Y坐标
* Z Coordinate：Z坐标

依次选中参数所对应的数据，然后使用 **DEFINE DATA TYPE** 定义数据类型，全部定义完成后，点击 **SET FIRST DATA LINE** 和 **SET LAST DATA LINE** ，最后点击 **CONFIGURATION COMPLETE.EXIT** 完成数据导入。

在配置完成后的弹窗中选择 **Match on Source Number** ,点击 **Combination Options** --> 勾选 **Combine Line and Source Numbers** --> **Line Multiplier** 设置为炮点数值位数的 **10** 倍（例如炮点数值最大值为4位数，则值设置为5位数，即10000）。

数据导入后，对其进行筛选，首先对Pattern进行排序：选中 **Pattern** 列 --> **Edit** --> **Sort...** ，然后选中所有 **Pattern** 为空值的数据，全部删除 --> **Edit** --> **Delete Rows** ，然后对 **X Coordinate** 排序，查看是否存在空值（如果存在就对其中进行插值），最后选中 **Record Num** 列对其排序，将数据设置回规范状态。

![GeoScribe2_8.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba49884f0.png)

## 保存

至此，整个建观流程已经完成，可以点击 **Plot** --> **Map View** 查看检波点和炮点图，检查是否导入成功。检查无误后，点击 **File** --> **Save Database** 保存文件。

**Interfaces** --> **SPS**，设置好炮点、检波点和关系文件的保存位置和名称后，导出为SPS文件。

![GeoScribe2_9.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba49bdb16.png)

![GeoScribe2_10.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba496fa89.png)

## 其他

因为地震数据采集的人员及方式各有不同，返回的检波点、炮点、关系数据文件格式各不相同，所以并没有统一的格式形式，这对进行数据参数配置的时候造成了一定的麻烦。不过，绝大多数文件的排列顺序是一致的，所以按照参数的顺序及三个文件的互相验证基本可以确定参数的具体位置。

在定义参数的时候，因为预处理中对数据作了右对齐处理，所以选取数值时右侧不要有空白，而左侧可以尽量的冗余（以保证包括不同位数的数值）。例如定义某 **From Sta** 数据：

![GeoScribe2_11.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba7773c88.png)

下面是 **检波点数据、炮点数据、关系数据** 的一般实例：

![GeoScribe2_12.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba7869a5c.png)
![GeoScribe2_13.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba7847639.png)
![GeoScribe2_14.png](https://youke1.picui.cn/s1/2025/09/02/68b6ba786e700.png)

* **Utilities**-->**Datum Statics**可以查看和设定*Datum Elevation*和*Correctional Velocity*值
* **Utilities**-->**Merge Database**合并多个数据库