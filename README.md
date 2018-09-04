# Vehicle Detection
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


The goal of this project is to write a software pipeline to detect vehicles in a video.

Here are links to the labeled data for [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) examples to train your classifier.  These example images come from a combination of the [GTI vehicle image database](http://www.gti.ssr.upm.es/data/Vehicle_database.html), the [KITTI vision benchmark suite](http://www.cvlibs.net/datasets/kitti/), and examples extracted from the project video itself. Also You are welcome and encouraged to take advantage of the recently released [Udacity labeled dataset](https://github.com/udacity/self-driving-car/tree/master/annotations) to augment your training data. 

Steps of this project
---
* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.


[//]: # (Image References)
[image0]: ./examples/car.png
[image1]: ./examples/notcar.png
[image2]: ./examples/HOG_example.png
[image3]: ./examples/sliding_window1.png
[image4]: ./examples/sliding_window2.png
[image5]: ./examples/sliding_window3.png
[image6]: ./examples/sliding_window4.png
[image7]: ./examples/bboxes_and_heat.png
[image8]: ./examples/final_output.png
[video1]: ./output_video_final.mp4

Histogram of Oriented Gradients (HOG)
---
The code for this step is contained in the second code cell of the IPython notebook.

Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image0]
![alt text][image1]

I has explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]

I tried various combinations of parameters and find with `orientation = 11` and `pixel_per_cell=8`, `cells_per_block=2` works the best.

SVM classifier
---
I trained a linear SVM using HOG features, spatial features and color histogram. In order to optimize the classifier I use grid search cross validation to tune the hyperparameter and decide to use `C=0.01`.

Sliding Window Search
---
Use sliding window with scale=1.2, scale=1.5 and scale=2. The sliding window search the images with one cell per step. To make the vechicle detection save, I use the smallest step and use two scales to search through the image space.

Inference
---
It is found that there are several scaled windows that are required. To include more searching windows would be very expensive. However, it is found that certain window scale are only suitable for a certian depth range. Therefore by constraining different search window to a certain search depth can save a lot of computation. The sliding window is run twice with three scale `(1.2, 1.5, 2)`. Ultimately I searched on three scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
---

Video Implementation
---
Here's a [link to my video result](./output_video_final.mp4)

I recorded the positions of positive detections in each frame of the video.  From the positive detections, I add up the positive detections found in the previous several frames. Then use the previous step heatmap times a decay rate and add up with current heatmap to generate a smoothed heatmap. After that threshold on the smooth heatmap to identify vehicle positions. 

I use `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  Then assume each blob corresponds to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

Here are two frames and their corresponding heatmaps:
![alt text][image7]

Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames and the resulting bounding boxes are drawn onto the last frame:
![alt text][image8]


Discussion
---
I found using multiple frame will produce better results. In order to detect the vehicle in the far scene, you need to have a small scale. However, this compromise the performance of the system. An imporovement is to have the search scale determined by the scence depth the bounding box is located. The vechicle detection usually fail on the boarder of the images. That is due to there are always some pixels being left out by the sliding window methods. The window size and scale should be optimised to reduce those left over pixels.