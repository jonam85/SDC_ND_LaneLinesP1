# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**
Author : Manoj Kumar Subramanian

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


---

### Reflection

# Startoff
My pipeline started of the following steps. 

1. I started using the same sequences applied in the lecture videos on to the sample image to identify the Lanes on the image.
2. draw_lane_pipeline is the function which takes the image as an input and outputs an image with lane lines drawn on the image.
3. As per the sequence, the image is converted to grayscale, converted to blur_gray using guassian_blur function with kernel_size of 5.
4. Then used the canny function with thresholds of 50 and 150 to bring out the edges from image.
5. Kept the trapeziod vertices as Image lower left corner (0,Image_Height) to a short left of near center of image size (size/2 +/- correction), to Image lower right corner (Image_Width,Image_Height) for masking
6. Sent the masked image to hough_lines function to obtain the Hough lines list from the masked area.
7. Used the draw_lines function with the initial settings to draw the lines on to the image.

# Initial observations
1. The output had lots of lines drawn on to the original picture.
2. The images were capturing multiple small lines outside the required lane lines
3. The image contained dotted lanes, so the lines drawn were not joined

# Corrections 1
1. Adjusted the trapezoid vertices to crop out for only the required portion
2. Adjusted the parameters of Hough lines function to match for discarding smaller lines

# Editing draw_lines averaging lines and Extrapolating
1. The lines were with X1,Y1,X2,Y2 co-ordinates. Identified to use slope method to segregate the left and right and started appending the lines to their respective lists for averaging.

```python
    m = ((y2-y1)/(x2-x1))
    c = (y1 - (m*x1))
    if m > 0:
        #positive slope => Right lane
        sum_right_slope.append(m)
        sum_right_const.append(c)
    else: #3
        #negative slope => left lane
        sum_left_slope.append(m)        
        sum_left_const.append(c)
```

2. Used the numpy mean function to find the average of each lines

```python
    Right_Lane_slope = np.mean(sum_right_slope)
    Right_Lane_const = np.mean(sum_right_const)
```

3. Calculated the averaged co-ordinates for the extrapolated line considering the line would extend from the bottom of the image (Image_Height as Y) to the (near) center of the image (Image_Height/2 +/- correction).

```python
        Right_Lane_Y1 = imshape[0]
        Right_Lane_X1 = (Right_Lane_Y1 - Right_Lane_const)/Right_Lane_slope
        Right_Lane_Y2 = imshape[0]/2 + 50
        Right_Lane_X2 = (Right_Lane_Y2 - Right_Lane_const)/Right_Lane_slope
```

4. Drawn the lines on to the image and found the averaged out signals are pretty good on the image level.
```python
    cv2.line(img, (int(Right_Lane_X1), int(Right_Lane_Y1)), (int(Right_Lane_X2), int(Right_Lane_Y2)), color, thickness)
```

# Running on the video
1. Put up all the sequences in a function draw_lane_pipeline and called the function from process_image function already defined.
2. The lines started appearing - but 2 observations -
    a. The lines were not properly aligning on to the actual lane images in the video
    b. There were lots of jittery between the frames
    
# Corrections 2
1. Logic added to put more weights to longer lines and shorter weight to smaller lines. This is done because, there were more number of smaller lines at the center of the image where the lane horizon is appearing which brings the slope converging towards flat lane line. When averaging without weights, the number of smaller lines adds up to bring the extrapolated line towards flat lane.

```python
    #find distance to add weightage for bigger lines
    #Longer the line, higher the weight
    d = math.sqrt(((x2-x1)**2) + ((y2-y1)**2))
    weight = int(d/5)
```
2. The corrections were tested and the alignment issue was resolved.

# Addressing jittery
1. When run on the example videos, the line transitions were appearing jittery and not smooth between the frames.
2. Since the function was being called on each frame, the better way to avoid jittering seemed to be having using global variables and run through an average among the frames.

# Global variables and associated problems
1. Inserted global variables to the project to store the predicted lane line parameters over a circular queue.
2. Averaged out the queue to get a smooth curve between the frames.
3. But since global variables were used, the buffers used by global variables remains active over different calling periods. Hence when completing one video and running another, the second video started off with the previous videos values and shifted towards the lanes over averaging.
4. Hence introduced clear_buffer function to clear the global variables. This function needs to be called before using process_image for lane extraction from a new file.

# Outputs
1. The outputs are stored in the test_videos_output folder.

# Scope of Improvements
1. The lanes are curvy. Identifying the lanes as lines may not fit for most of the cases.
2. There should be a better structured way to handle between multiple frames rather than using global variables directly.


