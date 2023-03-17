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

#### MP.1 Data Buffer Optimization
A ring buffer is a data structure in programming languages that is treated as a circular buffer or cyclic buffer. It is a memory buffer that is used for the temporary storage of data streams. The ring buffer stores data elements in a circular fashion, so the oldest element is replaced by the newest element when the buffer is full. The actual buffer size is set to nominal size + 1.
As shown in the figures below. When tail == head, the buffer is empty and when (tail + 1) % len == head, the buffer is full.

#### MP.2 Keypoint Detection
The function `detKeypointsClassic` was implemented for SHITOMASI and HARRIS detectors. Other detectors were implemented in the function `detKeypointsModern`

#### MP.3 Keypoint Removal
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

#### MP.4 Keypoint Descriptors
Implement descriptors BRIEF, ORB, FREAK, AKAZE and SIFT and make them selectable by setting a string accordingly.
```
  //"BRISK", "BRIEF ", "ORB ", "FREAK ", "AKAZE ", "SIFT" All binary descriptors except sift
  cv::Ptr<cv::DescriptorExtractor> descriptor;
  if (descriptorType == "BRISK"){
      int threshold = 30;        // FAST/AGAST detection threshold score.
      int octaves = 3;           // detection octaves (use 0 to do single scale)
      float patternScale = 1.0f; // apply this scale to the pattern used for sampling the neighbourhood of a keypoint.

      descriptor = cv::BRISK::create(threshold, octaves, patternScale);
  }
  else if (descriptorType == "BRIEF"){
      descriptor = cv::xfeatures2d::BriefDescriptorExtractor::create();
  }
  else if (descriptorType == "ORB"){
      descriptor = cv::ORB::create();
  }
  else if (descriptorType == "FREAK"){
      descriptor = cv::xfeatures2d::FREAK::create();
  }
  else if (descriptorType == "AKAZE"){
      descriptor = cv::AKAZE::create();
  }
  else if (descriptorType == "SIFT"){
      descriptor = cv::xfeatures2d::SiftDescriptorExtractor::create();
  }

  // perform feature description
  descriptor->compute(img, keypoints, descriptors);

```