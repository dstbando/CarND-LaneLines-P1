# **Finding Lane Lines on the Road**
---

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### 1. Finding lanes pipeline

My pipeline consisted of five steps as below:
1. Convert captured image into gray scale
2. Apply Gaussian filter
3. Detect edge features
4. Apply Hough transform to find line segments
5. Filter & integrate obtained line segments as left and right lane marks


First, I converted the images to grayscale, then applied Gaussian filter. As the third step, I detected edges via Canny edge detector. After masking ROI based on geometric constraints (in our case, because the lane marks are always on the road, we can focus trapezoid area of the bottom of the captured image), I extracted line segments using Hough transform. Usually, Hough transform can extract a lot of line segments; thus, we need to integrate them to find left and right lane marks. After removing outliers via simple thresholding, I clustered the line segments according to the gradients of the line segments. Because the origin of the image is the top-left corner, negative gradients mean that they belong to the left lane mark, and positive gradients are right lane mark. Then, finally, I got the gradients and the intercepts via weighted averaging of the gradients and the intercepts of the line segments. In my pipeline, I emphasized the longer and lower line segments just because they are more confident. (The longer line segments got more votes in Hough transform. The lower lane marks are confident lanes because they are detected in higher-resolution and more focused area in the captured image)   

Also, because my pipeline has several parameters, I used [ipywidgets](https://github.com/jupyter-widgets/ipywidgets) to implement the interactive parameter controller for finding efficient parameters on Jupyter notebook.


To make my pipeline more robust, especially for the "optional challenge," I employed two additional routines: pre-process and tracking. As a preprocess, I resized the captured image to the one resolution for standardizing the parameters of lane mark detection. Additionally, I emphasized yellow color via simple thresholding in Hue space. After generating the color mask for yellow color extraction, I used cv2.addWeighted() to emphasize masked region. For tracking, I implemented simple Kalman filter. I employed random walk as the dynamics model and calculated the variance of observation noise, i.e., confidence for the observation, by using the weight parameter for the line segments which are used to obtain the gradients for left and right lane marks. If we can get enough line segments (for example, with solid lane mark) to calculate the integrated gradients and intercepts, the variance of the observation noise should be small, i.e., with strong confident for the observation. Conversely, if we can get only a few segments with dashed lines or Botts' dots, we can naturally reduce the confidence of observation with increasing the variance of noises. As a result, I can get smooth detection results of the lane marks. For the detail results, please see my Jupyter notebook.

### 2. Identify potential shortcomings with your current pipeline
The potential shortcomings are listed below:
* **Curved lane marks:** So far, I'm using line fitting to detect lane marks. This is obviously not enough to detect curved lanes. Up and down of the road also are represented as "curved" lanes on the captured images.
* **Concrete road surface:** Concrete road surface usually has low contrast especially in the case of yellow lane marks. Also, it tends to be contaminated with tire marks or some other kinds of stains. This will lead false detection of edges on the captured images as well as line segment detection.
* **Severe lighting condition:** Especially in California, we have strong sun light in every morning and evening, and unfortunately, those time are commute time. This kind of severe lighting condition contaminates the edge detection results as well as line segments detection.
* **Ramps, merging spots, lane changes:** Those break my assumption for setting ROI on the captured images. Fixed ROI is not suitable to apply to the situation of lane width changing at merging spots such as freeway ramps. Also, during the driver changes his/her lane, the lane marks should be located at unusual positions.


### 3. Suggest possible improvements to your pipeline

The most of the potential shortcomings I mentioned above contaminate the edge detection results. Thus, a possible improvement would be to make edge detection procedure more robust, e.g., by employing adaptive setting of threshold parameter. Employing more robust ROI setting such as utilizing past frames to set ROI would also be useful. Additionally, using the map information with localization of the vehicle seems to improve the results. With those improvements, employing polynomial representation of the lanes (or employing bezier or spline curves) is suite for representations of curved lane marks. Furthermore, introducing the more rich approach for tracking is also useful. Because Kalman filter, the tracking method now I'm employing, assumes linear dynamics of states of lane marks and Gaussian distributions for process and observation noises. Those assumptions are not realistic, especially in the severe situations. Particle filtering is one candidate of good tracking approach which allows us to employ non-linear dynamics with non-Gaussian noises.

Another potential improvement could be to employ deep learning approach to detect lanes directly from the captured images. But it requires massive images with annotations to train networks.  
