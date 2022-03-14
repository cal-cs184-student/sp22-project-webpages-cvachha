# CS184 Project 3-1 Rasterizer
## Cyrus Vachha and Siming Liu
### Spring 2022 

### URL: https://cal-cs184-student.github.io/sp22-project-webpages-cvachha/proj3/

## Overview
In this project, 


## Part 1: Ray Generation and Scene Intersection

**Walk through the ray generation and primitive intersection parts of the rendering pipeline**

Ray generation is an important part of geometric modeling. We generate a ray by setting its original position and ray direction. In our implementation for class Camera’s `generate_ray()` function, we first find the corresponding position in the camera's sensor plane given a pixel and calculate the direction vector in the camera’s coordinates. Then we convert this direction vector from the camera's coordinates to the world coordinates by left-multiplying the `o2w` matrix. Note that, as cameras have clipping planes, we also have to set the `nClip` and `fClip` for each ray we generate.

Primitive intersection is usually needed in ray tracing. It tells us whether a ray we generate intersects with any primitives in our scene and thus determines how we perform rendering. In our code base, we implemented the ray-triangle intersection, ray-sphere intersection and ray-bbox intersection functions.

**Explain the triangle intersection algorithm you implemented in your own words**

In the Ray-Triangle intersection function, our implementation can be described in the following steps:

1. First check if the ray intersects with the plane that contains the target triangle. If there is no intersection, for example the ray is parallel to the plane, we return false. 

2. If there is an intersection, we get the corresponding t value of the intersection and check if the t value is between the ray’s `min_t` and `max_t`. If so, we go to step 3; otherwise, we return false. 

3. In this step, we test if the intersection point is within the triangle using Möller–Trumbore ray-triangle intersection algorithm described in lecture. If so, we go to step 4; otherwise, we return false.

4. We set the ray’s `max_t` to this intersection’s t value and update the `*isect` object correspondingly.


**Show images with normal shading for a few small .dae files**


## Part 2: Bounding Volume Hierarchy


**Walk through your BVH construction algorithm**

We implemented the BVH construction algorithm in a recursive way. For a given node, we first check if the number of primitives it has exceeds the `max_leaf_size` we set. If not, then we treat it as a leaf node and set its bounding box, start and end primitive iterator and then return the node; otherwise, we split the node into two child nodes and perform BVH construction on the left and right node.


**Explain the heuristic you chose for picking the splitting point.**

We choose the splitting point in the following way:

1. First, we calculate the size of the node’s bounding box and choose its longest dimension as the dimension that we will split on.

2. Secondly, we sort the primitives by their center’s projected value along the dimension we choose and pick the median value.

3. At last, we use the median value to split the primitives into two groups. 


**Show images with normal shading for a few large .dae files that you can only render with BVH acceleration.**



**Compare rendering times on a few scenes with moderately complex geometries with and without BVH acceleration. Present your results in a one-paragraph analysis.**

| File name   | Without BVH | With BVH |
| ----------- | ----------- | -------- |
| cow.dae      | 38.71s     | 0.05s    |
| teapot.dae   | 21.27s     | 0.06s    |

As we can see from the table, rendering with bvh can boost the rendering speed by 3~4 magnitude. Actually, the rendering time for some very complex .dae files (like the `maxplanck.dae` file) is quite similar to those moderately complex files (also within 0.1s). So the bvh structure is very efficient in calculating ray-primitive intersection.




## Part 3: Direct Illumination

**Walk through both implementations of the direct lighting function**




**Show some images rendered with both implementations of the direct lighting function**

**Focus on one particular scene with at least one area light and compare the noise levels in soft shadows when rendering with 1, 4, 16, and 64 light rays (the -l flag) and with 1 sample per pixel (the -s flag) using light sampling, not uniform hemisphere sampling**

**Compare the results between uniform hemisphere sampling and lighting sampling in a one-paragraph analysis**


## Part 4: Global Illumination

**Walk through your implementation of the indirect lighting function**

We implement the indirect lighting function in a recursive way. Within each recursive call:

1. We first calculate the intersection point of the input ray. If there is no intersection between the ray and the scene, we return false; otherwise, we calculate the `one_bounce_radiance` at this intersection point (`hit_p`) and assign the value to L_out.

2. Then we get the depth of the input ray and increment it by 1. If the ray_depth reach the `max_ray_depth` we set, return `L_out`; otherwise, we create a `new_ray` starting at `hit_p` and random sample its direction;

3. We calculate the intersection between this new_ray and the whole scene. If there is no intersection, we return `L_out`; otherwise, we update the intersection information and do a Russian Roulette;

4. In Russian Roulette, we use the given function called `coin_flip` to decide whether to recurse again or stop. The termination probability we choose is 0.3.

5. If the function is not gonna terminate, we call the function itself again and input the `new_ray` and the updated intersection information.


**Show some images rendered with global (direct and indirect) illumination. Use 1024 samples per pixel**


**Pick one scene and compare rendered views first with only direct illumination, then only indirect illumination. Use 1024 samples per pixel. (You will have to edit PathTracer::at_least_one_bounce_radiance(...) in your code to generate these views.)**

**For CBbunny.dae, compare rendered views with max_ray_depth set to 0, 1, 2, 3, and 100 (the -m flag). Use 1024 samples per pixel**

**Pick one scene and compare rendered views with various sample-per-pixel rates, including at least 1, 2, 4, 8, 16, 64, and 1024. Use 4 light rays**

## Part 5: Adaptive Sampling

**Walk through your implementation of the adaptive sampling**

In order to implement the adaptive sampling, we made some modifications to the `raytrace_pixel()` function we implemented in part 1. We count the number of samples we’ve already sampled and when it reaches the multiple of `samplesPerBatch`, we calculate the average and deviation of the samples we get. Then we calculated the `I` and compare it to `maxTolerance⋅μ`, if `I` < `maxTolerance⋅μ`, then we stop sampling and store the number of samples into `sampleCountBuffer`; otherwise, we keep sampling until the number of samples reaches the max limit we set.


**Pick one scene and render it with at least 2048 samples per pixel. Show a good sampling rate image with clearly visible differences in sampling rate over various regions and pixels. Include both your sample rate image, which shows your how your adaptive sampling changes depending on which part of the image you are rendering, and your noise-free rendered result. Use 1 sample per light and at least 5 for max ray depth**


As is shown in the right figure, ceiling, bunny and shadow areas need more samples to converge than the wall and floor.






