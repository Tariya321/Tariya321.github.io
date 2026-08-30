---
title: "floorplan-macroPlace"
tags:
  - floorplan
publish: yes
---
## 1. vocab
detour
绕道，绕行

halo
依附于macro的隔离带

## 2. how

将macro摆放于chip边测，避免detour和IR drop
{{< figure src="/attachment/Pasted%20image%2020240322160713.png" alt="Pasted image 20240322160713" width="300" >}}

根据电路间连接关系决定摆放位置（short is better）
>可视化电路连接

保留macro间足够的布线空间
>分析hotspots
{{< figure src="/attachment/Pasted%20image%2020240322162721.png" alt="Pasted image 20240322162721" width="300" >}}

减少macro间的空白区域（要么不留，要留就留够）
{{< figure src="/attachment/Pasted%20image%2020240322162845.png" alt="Pasted image 20240322162845" width="300" >}}

