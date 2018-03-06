---
layout: "post"
title: "The summary of the fifth week"
date: "2018-02-30 10:30"
author: "Fang Zhou"
catalog: true
---
## Summary of Week’s Activities
- Take a photo when the person’s face detected
- Install Dropbox on Raspberry Pi
- Integrate Dropbox API into the code modified in week 4
 
## Problem encountered and solutions
- Camera capture function causes a significant delay.The Dropbox depends on a stable network connection. If the network is unstable, the photo cannot be uploaded. The error information is shown as follows.
![dropbox](/img/site/dropbox.png)
 
## Achievement in this week
- We can upload the photos to the cloud on a stable network connection. There are two steps.
### Get Dropbox uploader
```python
githubclonegithub.com/andreafabrizi/Dropbox-Uploader.git
./dropbox_uploader.sh
```
### Use instruction to upload
```python
from subprocess import call
Upload = "home/pi/Dropbox_Uploader/dropbox_uploader.sh upload path/to/file dropbox_filename"
call ([Upload], shell=True)
```
- The results are shown as follow pictures.
![save](/img/site/save.png)
![cloud](/img/site/cloud.png)
