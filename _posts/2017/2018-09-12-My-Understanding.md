---
layout: "post"
title: "My Understanding"
date: "2018-09-12 10:30"
author: "Fang Zhou"
catalog: true
---
## The process of calculation
- Read input data from probes, then save them in Data1[i] and Data2[i]
- Customize a function to prevent invalid input
- Obtain the value of voltage with the position of data(127 is the initial value which representS 0 V)
- Compare each input value to get the peak value(max & min)
- Every complete cycle contains five controls(0, 0.7, 0, 0.7, 0). In other word, this software use the value of control to judge the period.
- Use Lissa
- output result

## Question
- So many codes are looks useless.
- I cannot understand the meaning of the function of m_Chart2d.SetXYValue.
- Two times smooth and peak-peak value.
