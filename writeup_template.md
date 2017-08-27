**Vehicle Detection Project**

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

---
### Writeup / README

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the second code cell of the IPython notebook.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image0]
![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and find with `orientation = 11` and `pixel_per_cell=8`, `cells_per_block=2` works the best.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using HOG features, spatial features and color histogram. In order to optimize the classifier I use grid search cross validation to tune the hyperparameter and decide to use C=0.01.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to use sliding window with scale=1.2, scale=1.5 and scale=2. The sliding window search the images with one cell per step. To make the vechicle detection save, I use the smallest step and use the two scales to search through the image space.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

To make the pipeline run faster I increase the search steps but to maintain accuracy. Also it is found that there are several scaled windows that are required. To include more searching windows would be very expensive. However, it is found that certain window scale are only suitable for a certian depth range. Therefore by constraining different search window to a certain search depth can save a lot of computation. The sliding window is run twice with three scale (1.2, 1.5, 2). Ultimately I searched on three scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./output_video.mp4)

#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections, I add up the positive detections found in the previous several frames. I use the previous step heatmap times a decay rate and add up with current heatmap to generate a smoothed heatmap. And then thresholded that smoothed heatmap to identify vehicle positions. I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are two frames and their corresponding heatmaps:

![alt text][image7]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames and the resulting bounding boxes are drawn onto the last frame:
![alt text][image8]

---

### Discussion

#### 1.Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In this project, I found using multiple frame will produce better results. In order to detect the vehicle in the far scene, you need to have a small scale. However, this compromise the performance of the system. An imporovement is to have the search scale determined by the scence depth the bounding box is located. The vechicle detection usually fail on the boarder of the images. That is due to there are always some pixels being left out by the sliding window methods. The window size and scale should be optimised to reduce those left over pixels.

