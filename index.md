# CS 184 Assignment 1 Writeup

## Task 1 (20 pts)
- Walk through how you rasterize triangles in your own words.
    - For each pixel that falls within the bounds of the three lines of the triangle with vertices at the given (x, y) pairs, we fill that pixel with the provided color.
- Explain how your algorithm is no worse than one that checks each sample within the bounding box of the triangle.
    - Our algorithm actually does just check each sample within the bounding box of the triangle, with nested for loops that run from the minimum to maximum given vertex x and y values.
- Show a *png* screenshot of *basic/test4.svg* with the default viewing parameters and with the pixel inspector centered on an interesting part of the scene.
![](/img/image1.png)

## Task 2 (20 pts)
- Walk through your supersampling algorithm and data structures. Why is supersampling useful? What modifications did you make to the rasterization pipeline in the process? Explain how you used supersampling to antialias your triangles.
    - Supersampling algorithm
        - nested for loop inside the loops from task 1
        - calculate the horizontal and vertical offsets corresponding to the sampling rate for each pixel in the bounding box of the triangle, and sample the color at each of those offsets inside the pixel
        - After sampling, update the expanded sample buffer (size updated to be `width * height * sample rate`)
    - Data Structures
        - we created no new data structures, just modified existing ones (ie. increased size of sample buffer)
    - Why supersampling is useful:
        - supersampling is particularly useful for antialiasing, because it essentially gets a better heuristic in finer grain to soften and blur edges between shapes so that the cutoffs are less sharp and jagged
    - Rasterization pipeline modifications
        - when actually putting rgb values into the frame buffer, we first had to average the rgb values for the `sample_rate` samples for each pixel, then actually pass in those values to the frame buffer
        - related to the above, but we had to expand our sample buffer size to accommodate all the samples we took for each pixel, which involved reorienting the indexing of the sample buffer so that it was ordered in the manner of `[pixel 1 sample 1, pixel 1 sample 2, … , pixel n sample m]`
    - How we used supersampling to antialias our triangles?
        - we used supersampling to antialias our triangles by taking more samples per pixel within the bounding box of the triangle to smooth out transitions in colors on the screen. It particularly helps smooth sharp edges and corners in the images.
- Show *png* screenshots of *basic/test4.svg* with the default viewing parameters and sample rates 1, 4, and 16 to compare them side-by-side. Position the pixel inspector over an area that showcases the effect dramatically; for example, a very skinny triangle corner. Explain why these results are observed.
![](/img/image2.png)
![](/img/image3.png)
![](/img/image4.png)

- Why we observe these results
    - the more samples we take per pixel, the better the final rendered color will blend into the surrounding pixels and the image’s edges will blur out. When we only sample once per pixel, there is clearly no gradual attempt to blend colors (as seen in the pixel inspector), so the corner of the triangle appears very jagged and even discontinuous. On the other hand, for 4 and 16 samples per pixel, we see pixels in the pixel inspector that take on colors between red and white, which allow for a “softer” appearance and more natural edges/lines than just singular sampling can achieve.


## Task 3 (10 pts)
- Create an updated version of *svg/transforms/robot.svg* with cubeman doing something more interesting, like waving or running. Feel free to change his colors or proportions to suit your creativity. Save your *svg* file as *my_robot.svg* in your *docs/* directory and show a *png* screenshot of your rendered drawing in your write-up. Explain what you were trying to do with cubeman in words.
    - **Air traffic controller who’s blasting jams through their noise-cancelling headphones**: I was trying to angle the left arm up in a waving position and have the right leg kick back at an angle
![](/img/image5.png)



## Task 4 (10 pts)
- Explain barycentric coordinates in your own words and use an image to aid you in your explanation. One idea is to use a *svg* file that plots a single triangle with one red, one green, and one blue vertex, which should produce a smoothly blended color triangle.
    - Barycentric coordinates are a coordinate system in which any point in two-dimensional space is described as the weighted combination of the vertices of some given triangle. These weights are known as `alpha`, `beta` and `gamma` values of the given coordinate. The sum of these coordinate weights is always 1.


![a triangle with the vertices being colored red, blue and green.](/img/image6.png)



- Show a *png* screenshot of *svg/basic/test7.svg* with default viewing parameters and sample rate 1. If you make any additional images with color gradients, include them.
![](/img/image7.png)




## Task 5 (15 pts)

Pixel sampling is, stated coarsely, the concept of getting Color (rgb) values for certain pixel locations of a texture. To get the actual pixel value of a texture is not a complicated process — one just has to return the `texel` value of that location. However, pixel sampling has different methods, and we shall discuss two of them here: nearest neighbor and bilinear sampling. The idea of nearest neighbor is to return the color of the nearest `texel` to the given location. The idea of bilinear sampling is to get the color of the four nearest pixels, then return the `texel` representing their weighted average. One implementation detail to mention here: The passed-in argument `uv` is technically in the range of [0, 1] – it’s a weight, not an actual measure of the location on the texture space, and must therefore be scaled by the height and width of the texture in both cases.

![Nearest sampling](/img/image8.png)
![Bilinear sampling](/img/image9.png)



![Nearest sampling, 1 sample/pixel](/img/image10.png)
![Nearest sampling, 16 samples/pixel](/img/image11.png)

![Bilinear sampling, 1 sample/pixel](/img/image12.png)
![Bilinear sampling, 16 samples/pixel](/img/image13.png)


I find that the biggest difference is in when there are sharp lines between two colors — like in the letters in the images above. Notice how the word “Berkeley” looks jagged in nearest sampling, and clearer and smoother in bilinear sampling.


## Task 6 (25 pts)

The idea of level-sampling is that, instead of downsampling from a given texture when we need it, we go ahead and create lower resolution levels of our texture image. These are known as our mipmap levels. In level sampling, we mathematically figure out what level of the map we wish to sample from, and then sample from that level of the mipmap. This was mainly implemented through `Texture::sample` and `Texture::get_level`.  `get_level` is a helper function that returns the level of mipmap we should be using given certain values of `uv`. `sample` is a function that toggles between various level sampling methods, using `get_level` to access the necessary information and returning a sample.

Tradeoff comparisons

- Pixel sampling: Bilinear very good at anti-aliasing, nearest sampling marginally faster than bilinear
- Level sampling: Linear level-sampling slower than nearest-level as well as level-zero. Any sort of level-sampling better at anti-aliasing than zero-level sampling.
- Number of samples/pixel: Gets substantially slower as this increases, and definitely keeps getting more and more memory intensive. Antialiasing power also increases.



![Zero Level, Nearest Pixel Sampling](/img/image14.png)
![Zero Level, Bilinear Pixel Sampling](/img/image15.png)

![Nearest Level, Nearest Pixel Sampling](/img/image16.png)
![Bilinear Level Interpolation, Nearest Pixel Sampling](/img/image17.png)




- Using a *png* file you find yourself, show us four versions of the image, using the combinations of `L_ZERO` and `P_NEAREST`, `L_ZERO` and `P_LINEAR`, `L_NEAREST` and `P_NEAREST`, as well as `L_NEAREST` and `P_LINEAR`.
    - To use your own *png*, make a copy of one of the existing *svg* files in *svg/texmap/* (or create your own modelled after one of the provided *svg* files). Then, near the top of the file, change the texture filename to point to your own *png*. From there, you can run ./draw and pass in that svg file to render it and then save a screenshot of your results.
    - **Note**: Choose a *png* that showcases the different sampling effects well. You may also want to zoom in/out, use the pixel inspector, etc. to demonstrate the differences

