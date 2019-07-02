---
layout: post
title: "First incursions into Computer Vision"
date:   2019-07-15 01:05 -0800
categories: opencv computervision deadlifts
comments: true
author: Eugenio Pace
---

I took a few days off and decided to dive into a completely unexplored domain. With all the hype in ML and AI, I figured I would look into that first. 

However, for me, theory with no application is not very attractive. I suspected I would get bored, distracted, and achieved little. So I thought deeper into building something useful. I was in the middle of a (somewhat challenging) workout when I had an idea: would it be possible to build an app that could track my deadlifts and tell me if I was doing something wrong? Sort of a "virtual coach" for deadlifts.

I imagined having a camera on my side that would track the movements in real-time, alerting me when I lose good form. 

The exciting thing about it is that I didn't know where to start. Image processing, video is outside my league. Great!

Some extensive googling took me to an entirely new domain. I love increasing awareness of my ignorance.

> "You should keep learning as long as you are ignorant, – even to the end of your life if there is anything in the proverb. And the proverb suits the present case as well as any: "As long as you live, keep learning how to live." For all that, there is also something which I can teach in that school. You ask, do you, what I can teach? That even an old man should keep learning." - Seneca


### OpenCV

Wow. Just wow. This library is amazing. I have barely scratched the surface on what it can do, but even the simplest tutorials can allow you to do amazing things with just a few lines of code.

[OpenCV is an open source library](https://opencv.org) developed by Intel more than 20 years ago! It's a massive library. I followed [these instructions]() to install the source, compile it, and run it. It took me _hours_ with many trial and errors. However, eventually it all worked out, and I had all tests working.

OpenCV is a C++  library, and I'm sure it would be a great case study. However, that will have to wait.

> I've always found reading someone else's codebase an excellent learning tool. 

### First attempt

Without knowing _anything_ I figured it would be easier to detect well-defined spots on a picture (say a green dot). After all, following a deadlift movement is about tracking the motion of the hinge in your hips: between the back and your hamstrings. Also, the barbell should move as vertically as possible throughout the movement.

![](/media/1a.jpg)

So I set my first goal to be able to detect the shoulder, the hip, and the bar as seen from the side as seen in the picture above.

To make things simpler, I just pasted a green dot on the picture (as if I was wearing a sticker) and saw how far I could go.

### C++ or Python

I like and enjoy writing C++ code. I use it extensively for all my hardware projects. However, I wanted fast iterations and make it easy to run many trial and errors, so I opted to install Python, and bindings for OpenCV. These are wonderful, and allows me to throw something on the script quickly, have a terminal window ready and try very quickly. So, Python it is!

> I know very little Python. My son Agustin taught me the little I know. So, double challenge! New domain, new language.

### First result

I googled my way around this first assignment. I ran my algorithm on the picture and ... WOW:

![](/media/cv-dl.jpg) 

I hardly believed it.

The first picture is the original frame with the added "stickers." The middle one is the result of applying a filter to eliminate anything, not green and finding the location of each. The final picture is the original with new red circles on the detected green dots. SUCCESS!

### The code

```py
# import the necessary packages
import numpy as np
import argparse
import cv2

# Draws detected circles
def draw_keypoints(vis, keypoints, color = (0, 255, 255)):
    for kp in keypoints:
            print(kp.size)
            print(kp.pt)
            x, y = kp.pt
            cv2.circle(vis, (int(x), int(y)), int(kp.size), color, -1)

ap = argparse.ArgumentParser()
ap.add_argument("-i", "--image", required = True, help = "Path to the image")
args = vars(ap.parse_args())

# load the image and copy to edit
image = cv2.imread(args["image"])
original = image.copy()

# Detect green dots
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

sensitivity = 15;
lower_green = np.array([60 - sensitivity, 100, 50])
upper_green = np.array([60 + sensitivity, 255, 255])

mask = cv2.inRange(hsv, lower_green, upper_green)
res = cv2.bitwise_and(image,image, mask=mask)

#Run SimpleBlobvDetector on filtered image
params = cv2.SimpleBlobDetector_Params()
params.filterByColor = True;
params.blobColor = 255;

detector = cv2.SimpleBlobDetector_create(params)
keypoints = detector.detect(res)

draw_keypoints(image, keypoints, (0,0,255))

# Show all three
cv2.imshow("Original", original)
cv2.imshow("Green Dots", res)
cv2.imshow("KeyPoints", image)
cv2.waitKey(0)
```

I am *sure* this is far from optimal. I still don't fully understand how it works, to be frank. I kind of know, but this early success has encouraged me to continue this journey. Perhaps more impressive, all this can be achieved with just 45-ish lines of code (and thousands in the library).

> There're of course many versions of Python and many versions of OpenCV. Each one has its and incompatibilities. I had to google and stackoverflow many of those to find my way around. The above code is *OpenCV 4.0.0* and *Pyhton 3.7.3*

### What's next

I'm excited about the opportunities, so I decided to take a course on OpenCV and minimize my _trail and error_ method. I found [this course on Udemy](https://www.udemy.com/python-for-computer-vision-with-opencv-and-deep-learning) with great reviews, so...back to school, I guess.

> The number of resources available at incredibly low cost, to learn *anything* amazes me. Makes me grateful to live in this time. 

