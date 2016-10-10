####University of Pennsylvania
####CIS 565: GPU Programming and Architecture

##Project 3 - CUDA Path Tracer
* Xueyin Wan
* Tested on: Windows 7, Xeon(R) E5-1630 @ 3.70GHz 32GB, GTX 1070 8192MB (Moore 103 SigLab)
* Compiled with Visual Studio 2013 and CUDA 7.5

==================================================================
##My Path Tracer Features
###Core Features 
1. A shading kernel with BSDF evaluation:
  * Ideal Diffuse surfaces
  * Perfectly specular-reflective (mirrored) surfaces
2. Path continuation/termination using Stream Compaction
3. toggle between sorting path/intersection continuous by material type
4. toggle between cache first bounce

###Extra Coolness
1. Refraction (e.g. glass/water) [PBRT 8.2] with Frensel effects using Schlick's approximation(finally...)  
*Found great reference: http://graphics.stanford.edu/courses/cs148-10-summer/docs/2006--degreve--reflection_refraction.pdf*
2. Physically-based depth-of-field
3. Motion Blur
4. More is coming!

==================================================================
###Result In Progress

####Fresnel Refraction
Using Schlick's Approximation(http://graphics.stanford.edu/courses/cs148-10-summer/docs/2006--degreve--reflection_refraction.pdf)
* `Left Ball` : RGB = (0.98, 0.98, 0.98), Specular RGB = (0.25, 0.25, 0.8),  Reflection = 0.3, Refraction = 0.2, Diffuse = 0.5
* `Middle Cube`: Reflection = 0.8, Refraction = 0.2, No Diffuse
* `Right Pink Ball`: Only Diffuse
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornellTestFresnel.2016-10-09_23-18-37z.5000samp.png "Fresnel Refraction")

####Diffuse, Perfect Specular Wall, Transmissive and Depth of Field 
* `Left Wall` : Reflection = 1
* `Left Blue Ball`: Refraction = 1
* `Middle Purple Ball`: Refraction + Reflection = 1, No Diffuse
* `Right Pink Ball`: Only Diffuse
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornellTestDOF.2016-10-09_19-18-26z.5000samp.png "Depth Of Field and Mirror")

####Motion Blur
Motion Blur: The left sphere and middle cube is moving during the whole render process
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornellTestFresnel.2016-10-10_00-02-09z.5000samp.png "Motion Blur")

|![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornellTestFresnel.2016-10-09_18-51-14z.5000samp.png  "Motion Blur")


####Depth of Field Comparison
* `Left Ball` : Refraction = 1
* `Middle Cube`: Reflection = 0.4, Refraction = 0.6, Refraction + Reflection = 1, No Diffuse,
* `Right Ball`: Reflection = 0.2, Refraction = 0.2, Diffuse = 0.6

####Original Pic
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornellTestFresnel.2016-10-10_00-49-35z.5000samp_original.png "Without DOF") 

|Focal Length = 10.5 | Focal Length = 9 |
|------|------|
|![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornellTestFresnel.2016-10-10_00-57-14z.5000samp_10_5.png "DOF, FOCAL LENGTH = 10.5") | ![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornellTestFresnel.2016-10-10_01-01-39z.5000samp_9.png "DOF, FOCAL LENGTH = 9") |

####SEE WHERE I START FROM...
####BE CONFIDENT ABOUT YOUR HARD WORK!!! BE CONFIDENT ON THE WAY OF GPU!!! BE CONFIDENT WHEN FINDING A JOB!!!

|Ideal Diffuse surfaces | Perfectly specular-reflective surfaces |
|------|------|
|![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornell.2016-10-03_01-03-54z.5000samp.png "Ideal Diffuse surfaces") | ![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornell.2016-10-03_13-08-43z.5000samp.png "Perfectly specular-reflective surfaces") |

==================================================================
###ON or OFF? SORT_BY_MATERIAL, CACHE_FIRST_BOUNCE, STREAM_COMPATION
STATUS | SORT_BY_MATERIAL | CACHE_FIRST_BOUNCE | STREAM_COMPATION | 5000 INTERATIONS TOTAL TIME (s) 
--- | --- | --- | --- | ---
    | ON | OFF | OFF | 670.627    
    | OFF| ON  | OFF | 100.689
    | OFF| OFF | ON  | 169.234
    | OFF| OFF | OFF | 117.215
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/performance_form.png "Table")

Above results is based on my cornellTestDOFLAB.txt.
I used Siglab Machine to run all the results.

###How to implement these Toggle?
In pathtrace.cu, using Macro definition to realize this :)
```java
#define SORT_BY_MATERIAL 0
#define CACHE_FIRST_INTERSECTION 0
#define STREAM_COMPACTION 0
```
Thanks google groups!

For `STREAM_COMPACTION`, I used `thrust::partition()` method to calculate number of paths alive.

For `CACHE_FIRST_INTERSECTION`, I made two helper variables:
```java
bool cache_first_intersection 
static ShadeableIntersection * dev_first_intersections 
```
and use cudaMemcpy() method to copy the first batch of intersections into dev_first_intersection memory. 

For `SORT_BY_MATERIAL`, I made helper comparator `MaterialComparator()` and used `thrust::sort_by_key()` method to realize sorting by mateiral.  

###What can we learn from the table above?
* SORT_BY_MATERIAL is the slowest one.. If there're a lot of materials in this code, I think that should be better. 
* CACHE_FIRST_INTERSECTION : Totally went wrong when I opened DOF. But without DOF, result is good.
* STREAM_COMPACTION : Slower than all-OFF, I think maybe the ray's amount is huge, so the method takes unnecessary time. The method itself is good, yet sorting/compaction is slow (which is a pity :(.

###Dive into CUDA
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/CUDA_SUMMARY.PNG "CUDA_SUMMARY")
From the picture above, we could see ComputerIntersections is the most time wasted(72.79%), since this function is used to calculate all the intersections, and as we noticed, the ray's amount is huge.

