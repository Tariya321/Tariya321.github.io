---
title: "数模转换电路-DAC"
tags:
  - 电路/DAC
publish: yes
---
## 1. function of DAC

a digital word applied to the inputs of the DAC, which is then converted to an analog signal at the sampling frequency (Fs) applied to the DAC clock.
**Convert digital input to analog wave.**
{{< figure src="/attachment/DAC.png" alt="DAC" width="600" >}}

Or generate pulse wave with a comparer further.
{{< figure src="/attachment/DAC-1.png" alt="DAC-1" width="400" >}}


### 1.1. Reference
https://www.ti.com/lit/an/slaa523a/slaa523a.pdf
https://www.ti.com/document-viewer/lit/html/SSZT175

## 2. DAC series

电流舵型（current-steering）DAC以其卓越的性能成为高速设计的首选结构
>However, at cryogenic temperature, the mismatch problem domains circuit designs. 

4 main encode types of DAC
{{< figure src="/attachment/DAC-2.png" alt="DAC-2" width="300" >}}
[Nejquis-Rate D/A Converts (ncku.edu.tw)](http://msic.ee.ncku.edu.tw/course/AdvancedAnalogICDesign/20201210/ch2.pdf)

