## Auto Mosaicing Algorithm
`Environment: Python 3.9.12, dependencies and command for installing with anaconda in env.txt`

This program loosely follows the paper "Recognizing Panoramas" by Matthew Brown.  Originally I set out to implement the entire panorama pipeline as laid out in the paper, but due to time constraints have implemented the main thrust of the paper, "recognizing panoramas" amidst mixed sets of unordered images.  My general process follows this outline sequence:

1.  Compute each image's features (SIFT features are used for resilience against camera zoom effects causing scale changes in features)
2.  Image pairing:
    - a. For each image, compute feature matches between itself and each other image (use KD-Tree for speed, find good balance of speed and quality of results)
    - b. Compute a homography for each of these image pairings with the resulting matches.
    - b. Eliminate image pairings that either did not succeed in RANSAC homography estimation or do not meet some basic quality criteria (inlier ratio, number of inliers).
    - c. Rank each image's potential homographies in order of quality.
3. Finding connected sets.  Starting with the first image:
    - a. Consider this image and its best match a set.
    - b. Recursively search through all images finding images whose best match is included in the set already, adding these images to the set.
    - c. Continue recursion until an iteration does not add any new images to the set.
    - d. With remaining images, start the process again creating another connected set until all images have been placed in a set.
4. Stitching images.
    - a.  Compute each image's global homography
        * find first image to place(image with best homography quality) and set its homography to a transform to place it in the center of the canvas
        * recursively find the next image by searching for images whose best pair image has already had its global homography calculated
        * calculate each image's global homography by taking the dot product of the already transported image with the current image
        * repeat loop until every image has a global homography
    - b.  Since each image's original homography was calculated in a forward direction, the same in which it is placed on the canvas, no further homography transformations are needed, just use cv2's warpTransform to place each image in the canvas
    - c.  Crop out unecessary space and render



#### Things to improve after this course:
* perform better homography quality control to completely eliminate false positives
    * compare overlapped areas of images and throw away homographies that have too small of an inlier ratio within the overlap area
* pre-determine image grouping with a time threshold using any file metadata (images taken more than several days apart won't be considered)
* include option to write image match data to files after computation so that extremely large sets of images can be stitched without maxxing out RAM
* perform bundle optimization, gain compensation, and mosaic straightening

#### Test Runs
At the end of this file are multiple test runs that arrange the images in a number of different ways to demonstrate the program working fine in most scenarios, including
* ordered/randomized images
* small/large sets of images
* 1D/2D panoramas
* clean and noisy image sets

### Results
This algorithm works fairly well for all the given scenarios, it could be faster by utilizing a more reliable homography quality check so that once m number of pairs are found that meet the high standard, then an image can be considered as matched rather than comparing to every other image.  One very large image sets, it can begin to experience false negatives and the mosaics that would normally stitch well when done independently will fail to stitch.  There likely needs to be a better tuning of the flann tree and homography threshold parameters, or an adaptive set of parameters.
