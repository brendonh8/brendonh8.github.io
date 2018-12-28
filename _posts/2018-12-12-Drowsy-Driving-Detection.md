---
title: "Drowsy Driving Detection"
date: 2018-12-12
tags: [Python, AWS, OpenCV]
htmlwidgets: TRUE
header:
    overlay_image: "/assets/images/drowsydriving/crash.jpg"
excerpt: "Detecting drowsy driving from live video using Neural Networks"
---
## Table of Contents

- [Overview](#heading-1)

- [Face Detection](#heading-2)

## <a name="heading-1"></a>Overview

I decided to focus on a problem that has the potential to affect all of us for this project. I am going to try and solve the issue of falling asleep at the wheel by using a live video feed to detect drowsiness.

Drowsy driving has been said to be just as dangerous as drunk driving. It accounts for a total of 100000 crashes and 6000 deaths making up 7% of all crashes every year. It is definitely a prevalent problem and one that can be subsided. In order to accomplish this in real-time, the computations need to be fast. When drowsiness is detected, an alarm would sound to wake the driver up, so the computations will also need to be accurate or else there will be a lot of false alarms.

## <a name="heading-2"></a>Face Detection

Computations can be run on images by using the values of each pixel. Images are just a large matrix with sets of three numbers. Each number is the value of how strong red, blue, or green is in that pixel. The first step to determining drowsiness is to take These pixel colors and detect a face from patterns in the pixels. 

I detected faces in the image using something called the Histogram of Oriented Gradients (HOG). The histogram of oriented gradients is going to take the color values of each pixel and the pixels surrounding it ang give it a magnitude and direction. If there is a large difference in color between the pixel and surrounding pixel, it will be given a large magnitude. The angle of direction is going to area with the greatest difference in color. This is done in blocks on the image to determine features. 

<figure>
	<img scr='/assets/images/drowsydriving/hog-cell-gradients.png'>
	<figcaption>Center : The RGB patch and gradients represented using arrows. Right : The gradients in the same patch represented as numbers</figcaption>
</figure>

Once the magnitudes and directions are made for each pixel in the the patches, a histogram is created binning the pixels by angle. This would look something like below:

![image-center](/assets/images/drowsydriving/hog-histogram-1.png){: .align-center}

Then the histogram for a single patch would end up looking like this.

<figure>
	<img scr='/assets/images/drowsydriving/histogram-cell.png'>
	<figcaption>Histogram of Gradients for a single cell.</figcaption>
</figure>

Then these histograms are input into the SVM algorithm which determines if a facial feature is found or not. This is what allows the computer to recognize the facial features of an image. The face detection package dlib uses HOG to determine if a face is found similar to the image below:

![image-center](/assets/images/drowsydriving/face_hog.tiff){: .align-center}
