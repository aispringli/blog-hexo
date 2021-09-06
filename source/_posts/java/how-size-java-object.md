---
title: 如何计算Java对象大小
comments: true
mp3: /music/blog.mp3
date: 2021-08-31 13:55:19
updated: 2021-08-31 13:55:19
tags:
categories:
keywords:
---

### Java Object 组成
先看一下一个Object对象的组成部分及大小

<style>
content table th, .content table td {
    border: 1px solid #ddd;
    padding: 0;
}
</style>

<table border="2">
  <thead>
    <tr>
      <th></th>
      <th>32位</th>
      <th>64位压缩指针</th>
      <th>64位</th>
    </tr>
  </thead>
  <tody>
    <tr>
      <td>对象头(Header)</td>
      <td>8b</td>
      <td>12b</td>
      <td>16b</td>
    </tr>
    <tr>
      <td>实例数据(Data)</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>对象填充(Padding)</td>
      <td>4b倍</td>
      <td>8b倍</td>
      <td>8b倍</td>
    </tr>
  </tbody>
</table>

数组需要额外的内存存储数组的长度，在32位上占用4个字节，在64位上占用8个字节，这也是为什么数组的长度小于2的32次方。
据此可以推断出基本类型大小如下所示

<table border="2" >
  <thead>
    <tr>
        <th>类型</th>
        <th colspan = "3">对象头</th>
        <th colspan = "2">数据</th>
        <th colspan = "3">填充前</th>
        <th colspan = "3">填充</th>
        <th colspan = "3">总大小</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>类型</td>
      <td>32</td>
      <td title="64压缩指针">64-</td>
      <td>64</td>
      <td>32</td>
      <td>64</td>
      <td>32</td>
      <td title="64压缩指针">64-</td>
      <td>64</td>
      <td>32</td>
      <td title="64压缩指针">64-</td>
      <td>64</td>
      <td>32</td>
      <td title="64压缩指针">64-</td>
      <td>64</td>
    </tr>
    <tr>
      <td>Byte</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>1</td>
      <td>1</td>
      <td>9</td>
      <td>13</td>
      <td>17</td>
      <td>3</td>
      <td>3</td>
      <td>7</td>
      <td>12</td>
      <td>16</td>
      <td>24</td>
    </tr>
    <tr>
      <td>Short</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>2</td>
      <td>2</td>
      <td>10</td>
      <td>14</td>
      <td>18</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
      <td>12</td>
      <td>16</td>
      <td>24</td>
    </tr>
    <tr>
      <td>Integer</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>4</td>
      <td>4</td>
      <td>12</td>
      <td>16</td>
      <td>20</td>
      <td>-</td>
      <td>-</td>
      <td>4</td>
      <td>12</td>
      <td>16</td>
      <td>24</td>
    </tr>
    <tr>
      <td>Long</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>8</td>
      <td>8</td>
      <td>16</td>
      <td>20</td>
      <td>24</td>
      <td>-</td>
      <td>4</td>
      <td>-</td>
      <td>16</td>
      <td>24</td>
      <td>24</td>
    </tr>
    <tr>
      <td>Float</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>4</td>
      <td>4</td>
      <td>12</td>
      <td>16</td>
      <td>20</td>
      <td>-</td>
      <td>-</td>
      <td>4</td>
      <td>12</td>
      <td>16</td>
      <td>24</td>
    </tr>
    <tr>
      <td>Double</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>8</td>
      <td>8</td>
      <td>16</td>
      <td>20</td>
      <td>24</td>
      <td>-</td>
      <td>4</td>
      <td>-</td>
      <td>16</td>
      <td>24</td>
      <td>24</td>
    </tr>
    <tr>
      <td>Char</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>2</td>
      <td>2</td>
      <td>10</td>
      <td>14</td>
      <td>18</td>
      <td>2</td>
      <td>2</td>
      <td>6</td>
      <td>12</td>
      <td>16</td>
      <td>24</td>
    </tr>
    <tr>
      <td>Boolean</td>
      <td>8</td>
      <td>12</td>
      <td>16</td>
      <td>4</td>
      <td>4</td>
      <td>12</td>
      <td>16</td>
      <td>20</td>
      <td>-</td>
      <td>-</td>
      <td>4</td>
      <td>12</td>
      <td>16</td>
      <td>24</td>
    </tr>
    <tr>
      <td>数组</td>
      <td>12</td>
      <td>16</td>
      <td>24</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

从这里也可以看出为什么Boolean用4b而不是1b,开启指针压缩后可以节省大概30%的内存。

### 对象头详解
对象头内分为运行时元数据（Mark Word）、类型指针(Klass Point)。
元数据在32位上占用4b，在64位上占用8b；指针在32位上占4b，在64位上占8b，开启指针压缩后为4b;

32位对象头内部信息
{% asset_img Object_Header_32.png %}

64位对象头内部信息
{% asset_img Object_Header_64.jpg %}

