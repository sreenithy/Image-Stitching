## Image-Stitching

**Overview**

The goal is to automatically stitch images acquired by a panning camera into a mosaic. Thus we aim  to combine a series of horizontally overlapping photographs into a single panoramic image. Using the given keypoints and descriptors we wil find the best matching features in the other images. Then, using RANSAC, we will automatically align the photographs (determine their overlap and relative positions) and then blend the resulting images into a single panorama.

**Outline**
 
1. Choose one image as the reference frame.
 
2. Estimate homography between each of the remaining images and the reference
image. 

3. To estimate homography between two images, we use the
following procedure:

    (a) Detect local features in each image 

    (b) Extract feature descriptor for each feature point 

    (c) Match feature descriptors between two images.

    (d) Robustly estimate homography using RANSAC.

4. Warp each image into the reference frame and composite warped images into a single mosaic.

**Detailed Description**

*Feature Detection*

In this step, we identify points of interest in the image using the Harris corner detection method. For each point in the image, consider a window of pixels around that point.  Compute the Harris matrix H for (the window around) that point, defined as
![alt text](https://github.com/sreenithy/Image-Stitching/blob/master/misc/harriseq-structuretensor.png )

here the Ixp is the is the x derivative of the image at point p, the notation is similar for the y derivative. You should use the Sobel operator to compute the x, y derivatives. The weights  should be chosen to be circularly symmetric (for rotation invariance).  

H is a 2x2 matrix.  To find interest points, first compute the corner strength function as below
![alt text](https://github.com/sreenithy/Image-Stitching/blob/master/misc/harriseq1.png)

Once you've computed c for every point in the image, choose points where c is above a threshold. 
The keypoint detction code is given in file HaarisCorner.ipynb

Here the keypoints have been extracetd  for the three images and arefound in 
corners.mat – there are 3 arrays inside this file: corners left, corners
center and corners right for the three images.

If the keypoint is not given then use the following command to extract keypoints

```python 
dst = cv2.cornerHarris(gray,blockSize,ksize,alpha)
'''
img - Input image, it should be grayscale and float32 type.
blockSize - It is the size of neighbourhood considered for corner detection
ksize - Aperture parameter of Sobel derivative used.
k - Harris detector free parameter in the equation.

'''

#result is dilated for marking the corners, not important
dst = cv2.dilate(dst,None)

# Threshold for an optimal value, it may vary depending on the image.
img[dst>0.01*dst.max()]=[0,0,255]
```

*Feature Descriptors*

Now that you've identified points of interest, the next step is to come up with a descriptor for the feature centered at each interest point.  This descriptor will be the representation one would use to compare features in different images to see if they match. The 128-dimensional SIFT descriptors for each feature
point above is given in  sift features.mat – there
are 3 arrays inside this file: sift left, sift center and sift right. 


*Feature Matching*

1. Tentative Matching 

As the first step, use Euclidean distance to compute pairwise distances
between the SIFT descriptors. Then, you need to perform
”thresholding”. For thresholding, use the approach adopted in the paper “Multi-Image
Matching using Multi-Scale Oriented Patches” by Brown et al.[1]. 
The results are given below

![alt text](https://github.com/sreenithy/Image-Stitching/blob/master/misc/tentative1%2C2.png "Tentative Correspondence between the reference image and the left image")

![alt text](https://github.com/sreenithy/Image-Stitching/blob/master/misc/tentaive2%2C3.png "Tentative Correspondence between the reference image and the right image")

2. Robust Matching 

Here we robustly estimate homography using RANSAC algorithm. The steps are as below
```
    ALGORITHM for RANSAC
    H = eye(3,3); nBest = 0;
    for (int i = 0; i < nIterations; i++) #Repeat for nRANSAC iterations:
            {
              P4 = SelectRandomSubset(P); #Choose a minimal set of feature matches.
              Hi = ComputeHomography(P4); #Estimate the transformation implied by these matches
              nInliers = ComputeInliers(Hi); #count the number of inliers.
              if (nInliers > nBest)
                  {
                    H = Hi;                  
                     nBest = nInliers;
                   }
              }
     
```


![alt text](https://github.com/sreenithy/Image-Stitching/blob/master/misc/Robust1%2C2.png "Robust Correspondence between the reference image and the left image")

![alt text](https://github.com/sreenithy/Image-Stitching/blob/master/misc/robust2%2C3.png "Robust Correspondence between the reference image and the right image")

3. Once the the Homography is robustly estimated. We perform warping and stitch the images together into one seamless panaroma. It is important to be careful in this step since during the warping stage there might be some negative values of coordinated which might end up cropping the image when acumulated with the refernece. Hence we multiply by a translation matrix that moves the warped image to the appropirate position on the mosaic.

![alt text](https://github.com/sreenithy/Image-Stitching/blob/master/misc/stitchedimages.png "Robust Correspondence between the reference image and the left image")

