# SFND 2D Feature Tracking

<img src="images/keypoints.png" width="820" height="248" />

The idea of the camera course is to build a collision detection system - that's the overall goal for the Final Project. As a preparation for this, you will now build the feature tracking part and test various detector / descriptor combinations to see which ones perform best. This mid-term project consists of four parts:

* First, you will focus on loading images, setting up data structures and putting everything into a ring buffer to optimize memory load. 
* Then, you will integrate several keypoint detectors such as HARRIS, FAST, BRISK and SIFT and compare them with regard to number of keypoints and speed. 
* In the next part, you will then focus on descriptor extraction and matching using brute force and also the FLANN approach we discussed in the previous lesson. 
* In the last part, once the code framework is complete, you will test the various algorithms in different combinations and compare them with regard to some performance measures. 

See the classroom instruction and code comments for more details on each of these parts. Once you are finished with this project, the keypoint matching part will be set up and you can proceed to the next lesson, where the focus is on integrating Lidar points and on object detection using deep-learning. 

## Dependencies for Running Locally
1. cmake >= 2.8
 
2. make >= 4.1 (Linux, Mac), 3.81 (Windows)
 * Linux: make is installed by default on most Linux distros
 
3. OpenCV >= 4.1
 * All OSes: refer to the [official instructions](https://docs.opencv.org/master/df/d65/tutorial_table_of_content_introduction.html)
 * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors. If using [homebrew](https://brew.sh/): `$> brew install --build-from-source opencv` will install required dependencies and compile opencv with the `opencv_contrib` module by default (no need to set `-DOPENCV_ENABLE_NONFREE=ON` manually). 
 * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)

4. gcc/g++ >= 5.4

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./2D_feature_tracking`.

## Rubic

### MP.1 Data Buffer Optimization
<b>Implement a vector for dataBuffer objects whose size does not exceed a limit (e.g. 2 elements). This can be achieved by pushing in new elements on one end and removing elements on the other end.</b>

A ring buffer is a data structure in programming languages that is treated as a circular buffer or cyclic buffer. It is a memory buffer that is used for the temporary storage of data streams. The ring buffer stores data elements in a circular fashion, so the oldest element is replaced by the newest element when the buffer is full. The actual buffer size is set to nominal size + 1.
As shown in the figures below. When tail == head, the buffer is empty and when (tail + 1) % len == head, the buffer is full.

### MP.2 Keypoint Detection
<b>Implement detectors HARRIS, FAST, BRISK, ORB, AKAZE, and SIFT and make them selectable by setting a string accordingly.</b>

The function `detKeypointsClassic` was implemented for SHITOMASI and HARRIS detectors. Other detectors were implemented in the function `detKeypointsModern`

### MP.3 Keypoint Removal
<b> Remove all keypoints outside of a pre-defined rectangle and only use the keypoints within the rectangle for further processing. </b>

```
// only keep keypoints on the preceding vehicle
  bool bFocusOnVehicle = true;
  cv::Rect vehicleRect(535, 180, 180, 150);
  if (bFocusOnVehicle)
  {
      vector<cv::KeyPoint> kptsOnVehicle; // create empty feature list for current image

      for (cv::KeyPoint &keyPt : keypoints)
      {
        if (vehicleRect.contains(keyPt.pt) == true)
             kptsOnVehicle.push_back(move(keyPt));
      }
            keypoints = move(kptsOnVehicle);
  }
```

### MP.4 Keypoint Descriptors
<b> Implement descriptors BRIEF, ORB, FREAK, AKAZE and SIFT and make them selectable by setting a string accordingly. </b>

In `matching2D_Student.cpp` line 53 - 91.

### MP.5 Descriptor Matching
<b> Implement FLANN matching as well as k-nearest neighbor selection. Both methods must be selectable using the respective strings in the main function. </b>

In `matching2D_Student.cpp` line 7 - 50.

### MP.6 Descriptor Distance Ratio

Use the K-Nearest-Neighbor matching to implement the descriptor distance ratio test, which looks at the ratio of best vs. second-best match to decide whether to keep an associated pair of keypoints.

In `matching2D_Student.cpp` line 36 - 49.

### MP.7 Performance Evaluation 1
<b> Count the number of keypoints on the preceding vehicle for all 10 images and take note of the distribution of their neighborhood size. Do this for all the detectors you have implemented. </b>

| Detector | image0 | image1 | image2 | image3 | image4 | image5 | image6 | image7 | image8 | image9 | neighborhood size |
| :---    | :---  | :---  | :---  |  :--- | :---  | :---  | :---  | :---  | :---  | :---  | :---: |
| SHI-TOMASI | 125 | 118 | 123 | 120 | 120 | 113 | 114 | 123 | 111 | 112 | 4
| HARRIS | 50 | 54 | 53 | 55 | 56 | 58 | 57 | 61 | 59 | 57 | 4
| FAST | 121 | 115 | 127 | 122 | 111 | 113 | 107 | 103 | 112 | 117 | 7
| BRISK | 264 | 282 | 282 | 277 | 297 | 279 | 289 | 272 | 266 | 254 | 21
| ORB | 92 | 102 | 106 | 113 | 109 | 125 | 130 | 129 | 127 | 128 | 57
| AKAZE | 166 | 157 | 161 | 155 | 163 | 164 | 173 | 175 | 177 | 179 | 7
| SIFT | 138 | 132 | 124 | 137 | 134 | 140 | 137 | 148 | 159 | 137 | 5

### MP.8 Performance Evaluation 2

<b> Count the number of matched keypoints for all 10 images using all possible combinations of detectors and descriptors. In the matching step, the BF approach is used with the descriptor distance ratio set to 0.8. </b>
| Detector\Descriptor | BRISK | BRIEF | ORB | FREAK | AKAZE | SIFT |
| ---                 | :---  | :---  |:--- |:---   |:---   |:---  |
| SHITOMASI           | 767   | 944   | 907 | 768   | -     | 927  |
| HARRIS              | 393   | 460   | 449 | 396   | -     | 459  |
| FAST                | 744   | 873   | 879 | 737   | -     | 824  |
| BRISK               | 1570  | 1704  | 1510| 1524  | -     | 1646 |
| ORB                 | 751   | 545   | 761 | 420   | -     | 763  |
| AKAZE               | 1215  | 1266  | 1186| 1187  | 1259  |1270  |
| SIFT                | 592   | 702   | -   | 593   | -     |800   |

### MP.9 Performance Evaluation 3
<b> Log the time it takes for keypoint detection and descriptor extraction. The results must be entered into a spreadsheet and based on this data, the TOP3 detector / descriptor combinations must be recommended as the best choice for our purpose of detecting keypoints on vehicles. </b>