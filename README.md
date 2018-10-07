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





