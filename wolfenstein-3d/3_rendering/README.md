# 3 Rendering

Looking from above is awfully boring so time to do what we really came here to do which is to render this in 3D.

## 3.1 Theory

Here I will go through the steps from converting a 3D world into a 2D image using a method called raycasting.

### 3.1.1 Everything Is Cooler with Lasers

We already know our world can be represented in 2D as a top-down "flat world" so lets think about it that way for now. Imagine we have a 2D radar that shoots lazers from a single pinhole, where we project out rays at incremental angles, which can tell us how **far** and what **colour** is an ***object it hit*** (if it hit one at all)

![](./theory-projection.svg)

Which to the radar, looks like this:

![](./theory-radar.svg)

Which we can process by removing the borders between each "vertical pixel"

![](./theory-radar-cleaned.svg)

### 3.1.2 ~~New~~ Fake Horizons

Right now it doesnt look too convincing so we can add a (fake) horizon, sky and floor to the image.

![](./theory-fake-horizons.svg)

Looks a bit better but still awkward right?

### 3.1.3 You Are Looking at It Wrong: It’s All About Perspective

Remember I mentioned distance before? Well here we scale the vertical pixel's height with how far away the object was before a ray hit it.

![](./theory-perspective.svg)

Here is the layout for reference:
![](./theory-projection.svg)

### 3.1.4 RTX On (what if we just had more pixels)

To make it look even better, heres what it would look like with much more pixels, instead of having only a screen with ~30 pixels wide.

![](./theory-interpolated.svg)

Looks pretty good right? (I cheated a bit by using polygons but I am not drawing that many pixels)

## 3.2 Raycasting














## 2.2 How to Look at Things (Ray Casting)

- a radar scan per pixel
- image of over view
- demo

### 2.2.1 Scanning
- fov
- res

### 2.2.2 Lines on Grids (DDA)

### 2.2.4 Scaling by FOV

## 2.3 Graffiti-ing the Walls (Texture Mapping)








## 2.4 Further Reading:
1. Lode's Computer Graphics Tutorial - Raycasting *(https://lodev.org/cgtutor/raycasting.html)*
2. Wikipedia - Digital Differential Analyzer *(https://en.wikipedia.org/wiki/Digital_differential_analyzer_(graphics_algorithm))*