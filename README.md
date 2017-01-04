# Templatematching.JS 

Template matching implemented in Javascript

Demo: https://kantu.io/demo/templatematching.js 

Typical performance for the "BBC" test screenshot:

- PC (few years old): ~135ms
- Samsung S7 Mobile Phone (Chrome): ~500ms
- Virtual Box (no graphics card): ~1,700ms
- Samsung Chromebook: ~1,700ms
- SQDIFF *before* optimzations >> 20,000ms 

The demo works in Chrome and Firefox. It does not work with Microsoft Edge (yet).

********************

This project implements template matching using the SQDIFF (sum of squared differences) metric.

Multiple template matches are supported using a threshold value.

*[Planned Enhancements]*

Convert color images to their grayscale equivalents using an initial pass on the GPU before performing template matching.

Generate the fragment shader on-the-fly with the template image dimensions before being uploaded to the GPU. This is necessary 
since the for-loops in the fragment shader must use constants.

Speed improvements:

   The one major bottleneck in the code is the call to glReadPixels() in app.js, after having rendered the result matrix on the GPU.
   The contents of the framebuffer must be transferred to main memory (hence the call) and as such, large images will 
   have a significant impact on performance. In order to improve this, less data should be read via glReadPixels().

Stream reduction.

   Instead of executing the current fragment shader for every pixel, we instead process an entire row at a time, locating
   the minimum SQDIFF value in each row and storing it in the result matrix. This will result in a reduction of the original image 
   from MxN pixels to Mx1 pixels. 

   Note: Need to balance the additional execution time and complexity in the fragment shader against transfer times.

Image pyramids.

  Generate downsampled images and perform template matching from the lowest-to-highest level of detail.

   Example: 
        
    The original image is at level 0. Images at level 1 and 2 are generated (at 1/4 and 1/16 the amount of pixels).
        
   Template matching is performed on level 2 on the GPU and glReadPixels() is called to read the corresponding framebuffer data (result matrix).
        
    Based on the matched positions on level 2: 
        
      Calculate corresponding positions on the higher level. 
      For each position (= region of interest), do template matching on it and neighbouring positions.  
      Repeat up till and including level 0.
