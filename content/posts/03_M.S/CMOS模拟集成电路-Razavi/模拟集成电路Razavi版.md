---
title: "模拟集成电路Razavi版"
tags:
  - 模集
date: 2023-07-01
publish: yes
---


---


词汇

| word                  | meanings |
| --------------------- | -------- |
| headroom              | 余量       |
| common source, CS     | 共源       |
| transconductance      | 跨导       |
| open circuit          | 开路       |
| vacuum tube           | 真空管      |
| single-ended          | 单端的      |
| common-mode rejection | 共模抑制比    |
| charge pump           | 电荷泵      |
| kirhoff's law         | 基尔霍夫定律   |

---

# 2 MOSFET
## 1. 连接

处于 psub 的 nmos
{{< figure src="/attachment/%E5%A4%84%E4%BA%8Epsub%E7%9A%84nmos.png" alt="处于psub的nmos" width="400" >}}
处于 nsub 的 pmos
{{< figure src="/attachment/%E5%A4%84%E4%BA%8Ensub%E7%9A%84pmos.png" alt="处于nsub的pmos" width="400" >}}
处于 Psub 的 CMOS
{{< figure src="/attachment/%E5%A4%84%E4%BA%8EPsub%E7%9A%84CMOS.png" alt="处于Psub的CMOS" width="480" >}}

在同一衬底上，所以某种器件需要一个’局部衬底‘——阱（well）
当全局衬底为 Psub 时，nmos 无需 well，而 pmos 需要一个 Nwell 


## 2. 工作特性
在电阻工作区，若还满足 $V_{DS}<<2(V_{GS}-V_{TH})$，则称为**深线性区**
深线性区有类似于三极管的工作特点
![深三极管区的线性工作](/attachment/%E6%B7%B1%E4%B8%89%E6%9E%81%E7%AE%A1%E5%8C%BA%E7%9A%84%E7%BA%BF%E6%80%A7%E5%B7%A5%E4%BD%9C.png)

这是一个MOS电流源，其中只有一个端点是可动的，另一个用于连接电源/地
![MOS电流源](/attachment/MOS%E7%94%B5%E6%B5%81%E6%BA%90.png)
## 3. 跨导
跨导 transconductance
$g_m$定义为漏电流对栅压的变化率
$$
g_m = \frac{\partial{I_D}}{\partial{V_{GS}}} 
$$

放大应用时，常使MOSFET工作于饱和区

## 4. 二阶效应

- 体效应（背栅效应）
以一个NMOS为例，若衬底电位低于0，意味着更多空穴将被吸引至衬底一侧，形成反型层更加困难，即阈值电压$V_{TH}$增加

- 沟长调制效应channel length modulation

$$
L\cdot I_D = L_{eff}\cdot I_D' = V_{DS}
$$
- 亚阈值导电
在亚阈值区域，并非完全关断，沟道仍可以导电

> 一个有用的点是：在亚阈值区域，MOS管可以获得和BJT相比拟的放大能力。但请注意，此时MOS管的速度受限。

## 5. MOS器件电容

> 这部分可以参看 数字集成电路书籍上[反相器链中的CMOS反相器电容组成结构](/attachment/%E5%8F%8D%E7%9B%B8%E5%99%A8%E9%93%BE%E4%B8%AD%E7%9A%84CMOS%E5%8F%8D%E7%9B%B8%E5%99%A8%E7%94%B5%E5%AE%B9%E7%BB%84%E6%88%90%E7%BB%93%E6%9E%84.png)


{{< figure src="/attachment/MOS%E7%94%B5%E5%AE%B9%201.png" alt="MOS电容 1" width="200" >}}


{{< figure src="/attachment/%E5%AE%9E%E9%99%85MOS%E7%9A%84%E7%94%B5%E5%AE%B9%E7%BB%84%E6%88%90.png" alt="实际MOS的电容组成" width="600" >}}

栅漏和栅源电容随Vgs的变化曲线
![栅漏和栅源电容随Vgs的变化曲线](/attachment/%E6%A0%85%E6%BC%8F%E5%92%8C%E6%A0%85%E6%BA%90%E7%94%B5%E5%AE%B9%E9%9A%8FVgs%E7%9A%84%E5%8F%98%E5%8C%96%E6%9B%B2%E7%BA%BF.png)




## 6. 小信号模型
![MOS小信号模型](/attachment/MOS%E5%B0%8F%E4%BF%A1%E5%8F%B7%E6%A8%A1%E5%9E%8B.png)
由b到c的转换，因为b中表示的沟长调制效应的电流源大小与漏极电压成线性关系，所以可以等效为一个电阻，如C中所示

![完整的MOS小信号模型](/attachment/%E5%AE%8C%E6%95%B4%E7%9A%84MOS%E5%B0%8F%E4%BF%A1%E5%8F%B7%E6%A8%A1%E5%9E%8B.png)

>只要可能，人们倾向于NMOS而非PMOS

## 7. 电路基础

戴维南等效

诺顿等效



# 3 单级放大器

八边形法则-analog circuit
{{< figure src="/attachment/%E6%A8%A1%E6%8B%9F%E7%94%B5%E8%B7%AF%E8%AE%BE%E8%AE%A1%E7%9A%84%E5%85%AB%E8%BE%B9%E5%BD%A2%E6%B3%95%E5%88%99.png" alt="模拟电路设计的八边形法则" width="300" >}}

放大器的分类（从接法分类）
{{< figure src="/attachment/%E6%94%BE%E5%A4%A7%E5%99%A8%E5%88%86%E7%B1%BB.png" alt="放大器分类" width="600" >}}

## 1. 共X级接法的判别
>因为放大器有输入和输出两个端口，会占用MOSFET其中两个极，剩下那个极接地或电源，作为参考电极。  

① 共源级放大电路是源级作为参考电极，栅极和漏极作为输入输出；  
  
② 共栅极放大电路是栅极作为参考电源，源级和漏极作为输入输出；  
  
③ 共漏极放大电路（也称源级跟随器）是漏极作为参考电极，栅极和源级作为输入输出。

## 2. 共源级接法

![MOS管放大器-共源极接法](/attachment/MOS%E7%AE%A1%E6%94%BE%E5%A4%A7%E5%99%A8-%E5%85%B1%E6%BA%90%E6%9E%81%E6%8E%A5%E6%B3%95.png)

实际上，由于**集成电路工艺的精度仅在工艺中心**，所以常以导通的MOS管（栅压总为导通状态）作为负载电阻，如下图所示

### 2.1. 二极管作负载的共源级接法
common-source stage
{{< figure src="/attachment/%E4%BA%8C%E6%9E%81%E7%AE%A1%E4%BD%9C%E8%B4%9F%E8%BD%BD%E7%9A%84%E5%85%B1%E6%BA%90%E7%BA%A7%E6%8E%A5%E6%B3%95.png" alt="二极管作负载的共源级接法" width="400" >}}
图中的M2晶体管 栅极与漏极相连，既满足
$$
\begin{gather}
V_G = V_D
\end{gather}
$$
所以，不论为何种沟道的器件，总是饱和的

### 2.2. 电流源作为负载

>为什么提出使用电流源作为负载？
>因为电流增益表达为$A_v = g_m\cdot R_o$，所以要增大增益，必须通过增大跨导$g_m$或者增加输出阻抗。但是一旦MOS管的状态确定，跨导确定，所以只能通过增加输出阻抗来满足增益要求。如果通过增加负载电阻的方式，又会造成支流压降的下降，这意味着被压制的电压摆幅。

>更换理想电流源，理想电流源具有无穷大的负载！

- 电流源常用工作在饱和区的MOS管代替

{{< figure src="/attachment/%E7%94%B5%E6%B5%81%E6%BA%90%E5%81%9A%E8%B4%9F%E8%BD%BD%E7%9A%84%E5%85%B1%E6%BA%90%E7%BA%A7%E6%8E%A5%E6%B3%95.png" alt="电流源做负载的共源级接法" width="600" >}}
其中$V_b$是指，bias偏压

### 2.3. 引入源级负反馈

{{< figure src="/attachment/%E5%B8%A6%E6%BA%90%E7%BA%A7%E8%B4%9F%E5%8F%8D%E9%A6%88%E7%9A%84%E5%85%B1%E6%BA%90%E7%BA%A7%E6%8E%A5%E6%B3%95.png" alt="带源级负反馈的共源级接法" width="600" >}}

源级添加的电阻$R_s$即为源级负反馈电阻，起作用为：

while $V_{in}$ is increasing(we assume N-MOSFET works before saturation region), current $I_D$ will increase as well. By kirhoff's law, we know voltage on $R_s$ will increase

这等效于将部分的栅压增量作用于负载电阻上，从而改善了栅压过驱动的问题

## 3. 源跟随器（共漏极接法）
> source follower

{{< figure src="/attachment/%E5%85%B1%E6%BC%8F%E6%9E%81%E6%94%BE%E5%A4%A7%E5%99%A8%E5%8F%8A%E4%BD%9C%E4%B8%BA%E7%BC%93%E5%86%B2%E5%99%A8%E7%9A%84%E4%BE%8B%E5%AD%90.png" alt="共漏极放大器及作为缓冲器的例子" width="600" >}}

高输出阻抗意味着输入阻抗必须足够大，否则压降大部分落在前级输出电阻上

所以需要一个 源跟随器，在提高后级输入阻抗的同时，驱动负载

## 4. 共栅极

{{< figure src="/attachment/%E6%94%BE%E5%A4%A7%E5%99%A8%E7%9A%84%E5%85%B1%E6%A0%85%E6%9E%81%E6%8E%A5%E6%B3%95.png" alt="放大器的共栅极接法" width="600" >}}

输入信号控制源端，栅极保持一定偏置状态

## 5. 共源共栅级
>cascode

{{< figure src="/attachment/cascode.png" alt="cascode" width="300" >}}

- 常称M1为共源器件，M2为共栅器件

- 具有高输出阻抗

- 也可用于构成恒定电流源

- 具有屏蔽特性，避免输入的变化作用到输出

# 4 差动放大器
> Differential Amplifiers

## 1. 单端与差动的工作方式对比
{{< figure src="/attachment/%E5%8D%95%E7%AB%AF%E5%92%8C%E5%B7%AE%E5%88%86%E8%BF%9E%E6%8E%A5%E5%AF%B9%E6%AF%94.png" alt="单端和差分连接对比" width="600" >}}

- 共模电平$V_{CM}$：信号为0时的的电压，可以理解为偏置电压
- 更强的抗干扰能力：差动输出
- 更大的电压摆幅

- 差动既可以运用到输出，还可以运用到输入，都可以降低噪声

总体评价
>The numerous advantages of differential operation by far outweigh the possible increase in the area.


## 2. 基本差动对

{{< figure src="/attachment/%E7%AE%80%E5%8D%95%E5%B7%AE%E5%8A%A8%E7%94%B5%E8%B7%AF.png" alt="简单差动电路" width="300" >}}
但是当偏置电压$V_{CM}$变化时，偏置电流的同步变化将导致跨导和输出共模电平的变化

方法是，在源级添加一个电流源，从而使导通使$I_{D1}+I_{D2}$不依赖于偏置电压

{{< figure src="/attachment/%E5%9F%BA%E6%9C%AC%E5%B7%AE%E5%8A%A8%E7%94%B5%E8%B7%AF.png" alt="基本差动电路" width="300" >}}

# 5 频率响应

## 1. Miller效应

{{< figure src="/attachment/Miller%E6%95%88%E5%BA%94%E5%9C%A8%E6%B5%AE%E5%8A%A8%E9%98%BB%E6%8A%97%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8.png" alt="Miller效应在浮动阻抗中的应用" width="400" >}}

如果左边电路可以等效为右边电路，则他们必然满足下面的关系

{{< figure src="/attachment/miller%E7%AD%89%E6%95%88%E7%9A%84%E9%98%BB%E6%8A%97%E5%85%B3%E7%B3%BB%E6%8E%A8%E5%AF%BC.png" alt="miller等效的阻抗关系推导" width="600" >}}

{{< figure src="/attachment/%E5%88%A9%E7%94%A8Miller%E7%AD%89%E6%95%88%E8%BD%AC%E5%8C%96%E7%9A%84%E5%80%8D%E5%A2%9E%E8%BE%93%E5%85%A5%E7%94%B5%E5%AE%B9.png" alt="利用Miller等效转化的倍增输入电容" width="600" >}}

必须说明，上面是基于“电路可以等效”这一前提进行的推导，反过来是不可以的

然而在下图的这些情况中，miller效应被证明是有用的（即可以进行反向推导）

{{< figure src="/attachment/%E5%8F%AF%E4%BB%A5%E8%BF%90%E7%94%A8Miller%E5%AE%9A%E7%90%86%E7%9A%84%E9%80%9A%E5%B8%B8%E6%83%85%E5%86%B5.png" alt="可以运用Miller定理的通常情况" width="200" >}}


















# 6 反馈

## 1. 反馈结构

电压-电压反馈
![电压-电压反馈](/attachment/%E7%94%B5%E5%8E%8B-%E7%94%B5%E5%8E%8B%E5%8F%8D%E9%A6%88.png)

检测输出电压，返回电压的反馈信号

>或者称为“串联-并联“反馈，第一个词表示发反馈与输入的连接，第二个表示反馈与输出的连接




# 7 锁相环
>phase lock loop, PLL



## 1. 电荷泵
charge pump, CP

>A charge pump consists of two switched current sources that pump charge into or out of the loop filter according to two logical inputs.

![使用电荷泵的PFD](/attachment/%E4%BD%BF%E7%94%A8%E7%94%B5%E8%8D%B7%E6%B3%B5%E7%9A%84PFD.png)