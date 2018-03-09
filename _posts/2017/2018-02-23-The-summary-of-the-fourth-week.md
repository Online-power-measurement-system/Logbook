---
layout: "post"
title: "The summary of the fourth week"
date: "2018-02-23 10:30"
author: "Fang Zhou"
catalog: true
---
## Summary of Week’s Activities
- Try a new coding method -- multi-process
- Modify the circuit to ensure the circuit works well. The circuit is shown as follows.
![main](/img/site/main.jpg)
 
## Problem encountered and solutions
- Multi-process failed finally because the value could not be transferred 
from one sub-program to another. We did not find the way to solve this 
problem. Therefore, we combined them into one program of two running windows 
to avoid this kind of problem. 
- When modifying the circuit, the LED could not light at the beginning because 
the pin used to control the LED was not connected to the ground, which leaded 
to the pin recognizing the HIGH and LOW voltage incorrectly. So, we connected 
it to the ground and another resistor which was used to protect the circuit.
 
## Achievement in this week
- The original goals of the project were achieved successfully. The code of 
the Raspberry Pi and the circuit worked well. When the camera catches a person’s face, 
it will show a green square around his face on the screen and the LED which represents 
the automatic door will light for 5 seconds at the same time. The demo video is shown below.
<iframe 
     width="560" 
     height="315" 
     src="https://www.youtube.com/embed/yfqtsYiklZI" 
     frameborder="0" 
     allow="autoplay; 
     encrypted-media" 
     allowfullscreen>
</iframe>
