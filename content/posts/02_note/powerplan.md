---
title: "powerplan"
tags:
  - powerplan
  - EDA
  - 数字后端
publish: yes
---
需要额外打power ring的场景
- different power domain
- analog module（such as PLL
下面图文show一个powerplan的case

打power ring
```tcl
# power ring
selectObject Group PLL
addRing -type core_rings -nets {vdd_lp_s vss vdd} -layer {top METAL7 bottom
METAL7 left METAL8 right METAL8} -offset 1 -width 8 -spacing 1.0 -
exclude_selected 1
deselectAll

selectInst DTMF_INST/PLLCLK_INST
addRing -type block_rings -nets {Avss Avdd} -around selected -layer {top
METAL7 bottom METAL7 left METAL8 right METAL8} -width 5 -spacing 1 -offset 1
deselectAll

selectObject Group TDSPCore
addRing -type block_rings -nets {vdd_lp_s vss vdd} -around power_domain -
layer {top METAL7 bottom METAL7 left METAL8 right METAL8} -width 5 -spacing
1 -offset 1
deselectAll
```
{{< figure src="/attachment/Pasted%20image%2020240421183652.png" alt="Pasted image 20240421183652" width="500" >}}

打switch cell pins的power stripe
domain stripes are added around the stripes over switch cell pins for
each different layer
```tcl
# power stripe
selectObject Group TDSPCore
setAddStripeMode -skip_via_on_pin {}
addStripe -over_pins 1 -nets vdd_lp_s -over_power_domain 1 -layer METAL4 -
width 8 -master HEAD16DM -pin_layer METAL2
addStripe -over_power_domain 1 -nets vss -direction vertical -layer METAL4 -
width 8 -set_to_set_distance 70 -start_from left -start_offset 43.46 -
spacing 1
addStripe -over_power_domain 1 -nets {vdd_lp_s vss} -direction horizontal -
layer METAL7 -width 8 -set_to_set_distance 70 -start_from bottom -
start_offset 34.46 -spacing 1
addStripe -nets {vdd_lp_s vss} -over_power_domain 1 -layer METAL8 -width 8 -
set_to_set_distance 70 -start_from left -start_offset 34.46 -spacing 1
```
{{< figure src="/attachment/Pasted%20image%2020240421183601.png" alt="Pasted image 20240421183601" width="500" >}}

加core area的power stripe
stripes are added in the core area for each different layer
```tcl
addStripe -nets {vdd vss} -layer METAL4 -width 8 -set_to_set_distance 70 -
xleft_offset 37.9 -spacing 1
addStripe -nets {vdd vss} -direction horizontal -layer METAL7 -width 8 -
set_to_set_distance 70 -ybottom_offset 14 -spacing 1
addStripe -nets {vdd vss} -layer METAL8 -width 8 -set_to_set_distance 70 -
xleft_offset 37.9 -spacing 1
```
{{< figure src="/attachment/Pasted%20image%2020240421183427.png" alt="Pasted image 20240421183427" width="500" >}}

打power rail
```tcl
sroute -connect corePin
```
{{< figure src="/attachment/Pasted%20image%2020240421183322.png" alt="Pasted image 20240421183322" width="500" >}}

连接pad和ring
```
sroute -connect padPin
```
