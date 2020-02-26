# Semi-automated registration of GSC (in 5 minutes)
This document describes the steps to register time-lapse movies of *C. elegans* germline.

#### Materials
- a time-lapse tiff movie (only the TBB-2::GFP channel is required).


### 1 Generate a low resolution movie
This step is optional but highly recommended to speed up the registration process. We do not really need the high resolution details for registration purpose, so we trade resolution for smaller file size (from ~3GB to ~100 MB) to optimize the computation. The following steps can also be easily written into a ImageJ macro.

1. In Fiji, open your tiff file. If your file has more than one channel, split them and only keep the TBB-2::GFP channel.
2. Go to Image>>Adjust>>Size.. In the pop-up *resize* window, the new width should be 1/3 of the original width. For example, if the original width is 871 pixels, the new width should be 290 pixels. Check "Constrain aspect ratio" and "Average when downsizing". See Figure 1.
3. Click "OK".
4. Save the low resolution tiff with a different name.

![Figure 1](https://github.com/yifnzhao/Semi-automated-GSC-registration/blob/master/figures/Figure%201.png)

### 2 ROI (The only manual step)
1. With your low-res tiff opened in Fiji, go to Image>>Stacks>>Z project.. Select max intensity and "OK".
2. Open ROI Manager in Fiji.
3. Look for a pair of prominently congressing centrosomes. You might need to watch the movie a few
times to identify such a pair. But it gets very easy with a little practice.
4. At t = 0, draw a straight line in the middle of the two centrosomes that you identified in the previous step. This line should approximate the metaphase plate (see Figure 2). Then, enter "t" on your keyboard (shortcut for "Add" in ROI manager).
6. Go to the next time point. If the metaphase plate position changes, move your line accordingly using "->" or "<-" on your keyboard. Enter "t" to add the line. Repeat this step until the last time point is reached.
7. In the ROI Manager, check "Show All". You should see something similar to Figure 3. Go to "More>>List", and save the x, y coordinates as a csv file.

**Important Note**: the selection of the congressing centrosome pair is essential. It is ok if the selected pair disappears in the middle of the movie. In that case, you need to save the current ROI result, go to one time point back, and find another prominently congressing pair. The remaining steps are identical.


![Figure 2](https://github.com/yifnzhao/Semi-automated-GSC-registration/blob/master/figures/Figure%202.png)
![Figure 3](https://github.com/yifnzhao/Semi-automated-GSC-registration/blob/master/figures/Figure%203.png)


### 3 Generate the transformation matrix
1. Now you have your ROI csv and low-res tiff ready. With your favorite python IDE open, run get_transformation_matrix.py, input the file names when prompted and wait.
2. Open the registered low-res tiff. It is now a 2D stack. Go to Image>>Hyperstacks>>stack to hyperstack. In the popup window, enter the number of Slices and Frames of your original movie.
3. If the movie doesn't look alright, check the time points that looks strange. You can either directly approximate and fix the number in the csv file, or redo the ROI step if necessary. If the low-res movie looks alright, you can use the same matrix to register the high-res movie. 

#### Now, what function does the whole registration thing?
1. ```roi2mat(roi_df)``` reads your x-y coodinate input (i.e. the ROI csv) and returns a translation matrix.
2. ```combine_roi(mat1, mat2)``` is optional -- it combines the translation matrices in cases where you have multiple csv files for distinct centrosome pairs (see Part 2 Important Notes).
3. ```translate(im_in, translation, hi_res = False, compression = 3, padzeros = True)``` this one translates your movie! 
  - If you did step 0 and used 3 fold compressed low-res movie for ROI generation, to register a low-res movie, just run ```register(tiff_path, trans_mat)``` will work; to register a high-res movie, ```register(tiff_path, trans_mat, highres = True, compress = 3)``` .
  - If you set ```metadata = register(...)```, you will be able to see what metadata looks like in variable explorer in your favorite IDE. You can also save it in any format you like.
  - If you did something differently, just remember to specify ```highres``` and ```compress```. Basically, the algorithm multiplies your translation matrix by integer *compress* if boolean *highres* is True.
  - setting ```padzeros = True``` will give you uncropped movies (just pad periphery with black pixels) and ```padzeros = False``` will discard the portions out of "registered frame". Try it out!
  
Note: a good way to check is to do a double z-projection (the second projection is actually on t, not z). Ideally, the output image should look like the upper panel, Figure 4A, not lower panel, Figure 4B).

![Figure 4A](https://github.com/yifnzhao/Semi-automated-GSC-registration/blob/master/figures/Figure%204A.png)
![Figure 4B](https://github.com/yifnzhao/Semi-automated-GSC-registration/blob/master/figures/Figure%204B.png)
