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

###Extra Coolness...
1. Refraction (e.g. glass/water) [PBRT 8.2] with Frensel effects using Schlick's approximation(finally...)  
*Found great reference: http://graphics.stanford.edu/courses/cs148-10-summer/docs/2006--degreve--reflection_refraction.pdf*
2. Physically-based depth-of-field
3. Motion Blur

==================================================================
###Result In Progress

####Ideal Diffuse surfaces(without Anti-Aliasing)
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornell.2016-10-03_01-03-54z.5000samp.png "Ideal Diffuse surfaces")

####Perfectly specular-reflective surfaces(without Anti-Aliasing)
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornell.2016-10-03_13-08-43z.5000samp.png "Perfectly specular-reflective surfaces")

==================================================================
###ON or OFF? SORT_BY_MATERIAL, CACHE_FIRST_BOUNCE, STREAM_COMPATION
STATUS | SORT_BY_MATERIAL | CACHE_FIRST_BOUNCE | STREAM_COMPATION | 5000 INTERATIONS TOTAL TIME (s) 
--- | --- | --- | --- | ---
    | ON | OFF | OFF | 670.627    
    | OFF| ON  | OFF | 100.689
    | OFF| OFF | ON  | 169.234
    | OFF| OFF | OFF | 117.215

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
