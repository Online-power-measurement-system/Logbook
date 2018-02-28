---
layout: "post"
title: "The summary of the third week"
date: "2018-02-23 10:30"
author: "Fang Zhou"
catalog: true
---
## Summary of Week’s Activities

- Solve the problem of real-time face recognition.
- Write the code for Raspberry Pi to drive LED lights.
- Run the program which can recognize the face and control LED lights.

## Problem encountered and solutions

- In the beginning, the system cannot shoot the video and recognize the human’s face at the same time. Finally, we find some information from the Internet and modify the code to solve this problem.
- We try to combine the code of controlling LED and recognizing face. However, there is a problem that when the camera has recognized a face, the LED is light, but the display of camera is stuck. This problem now we can’t solve it.


## Achievement in this week

- The code for recognizing human face is shown as follows. 
```Python
### Imports ###
from picamera.array import PiRGBArray
from picamera import PiCamera
import cv2
import RPi.GPIO as GPIO
# Setup the GPIO mode as pin-coded
GPIO.setmode(GPIO.BCM)
#Set GPIO 18 as output mode
TRANS_OUT=18
GPIO.setup(TRANS_OUT, GPIO.OUT)
# Setup the camera
camera = PiCamera()
camera.resolution = ( 320, 240 )
camera.framerate = 60
rawCapture= PiRGBArray( camera, size=( 320, 240 ) )
# Load a cascade file for detecting faces
face_cascade = cv2.CascadeClassifier( '/home/pi/opencv-3.0.0/data/lbpcascades/lbpcascade_frontalface.xml' ) 
# Capture frames from the camera
for frame in camera.capture_continuous( rawCapture, format="bgr", use_video_port=True ):
    image = frame.array    # Use the cascade file we loaded to detect faces
    gray = cv2.cvtColor( image, cv2.COLOR_BGR2GRAY )
    faces = face_cascade.detectMultiScale( gray )   
    print "Found " + str( len( faces ) ) + " face(s)"
    # Draw a rectangle around every face and move the motor towards the face
    for ( x, y, w, h ) in faces:
        cv2.rectangle( image, ( x, y ), ( x + w, y + h ), ( 100, 255, 100 ), 2 )
        cv2.putText( image, "Face No." + str( len( faces ) ), ( x, y ), cv2.FONT_HERSHEY_SIMPLEX, 0.5, ( 0, 0, 255 ), 2 )
    temp=len(faces)
    if temp!=0:
	GPIO.output(TRANS_OUT,True)
	test=True
    else: 
	GPIO.output(TRANS_OUT,False)
	test=False
    # Show the frame
    cv2.imshow( "Frame", image )
    cv2.waitKey( 1 )    # Clear the stream in preparation for the next frame
    rawCapture.truncate( 0 )
GPIO.cleanup()
```
- The code for LED is shown as follows.
```python
import RPi.GPIO as GPIO
import time
GPIO.setmode(GPIO.BCM)
TRANS_IN=2	#transfer 
OUT=24
GPIO.setup(TRANS_IN,GPIO.IN)	#receive the data from PIN TRANS_OUT
GPIO.setup(OUT,GPIO.OUT)	#control the LED
data=0
while True:
	data=GPIO.input(TRANS_IN)
	print(data)
	if data==1 :
    	    GPIO.output(OUT, True)
    	    time.sleep(5)
    	    GPIO.output(OUT, False)
GPIO.output(OUT, False)
RPi.GPIO.cleanup()
```
