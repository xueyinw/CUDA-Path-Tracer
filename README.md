####University of Pennsylvania
####CIS 565: GPU Programming and Architecture

##Project 3 - CUDA Path Tracer
* Xueyin Wan
* Tested on: Tested on: Windows 10 x64, i7-6700K @ 4.00GHz 16GB, GTX 970 4096MB (Personal Desktop)
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
1. Refraction (e.g. glass/water) [PBRT 8.2] with Frensel effects using Schlick's approximation (yay)
   *Found great reference : http://graphics.stanford.edu/courses/cs148-10-summer/docs/2006--degreve--reflection_refraction.pdf*
2. Physically-based depth-of-field
3. Motion Blur

==================================================================
###Result In Progress

####Ideal Diffuse surfaces(without Anti-Aliasing)
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornell.2016-10-03_01-03-54z.5000samp.png "Ideal Diffuse surfaces")

####Perfectly specular-reflective surfaces(without Anti-Aliasing)
![alt text](https://github.com/xueyinw/Project3-CUDA-Path-Tracer/blob/master/results/cornell.2016-10-03_13-08-43z.5000samp.png "Perfectly specular-reflective surfaces")
