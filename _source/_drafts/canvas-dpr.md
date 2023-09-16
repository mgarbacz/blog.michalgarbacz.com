---
layout: post
title: "Canvas Rendering Blurry on High Pixel Density Displays"
---

# The Initial Head Scratcher

I was drawing very basic shapes in my canvas element, and noticed that their edges rendered incredibly blurry on my Macbook's monitor, but clearly on my external monitor. A bit of Googling led me to discover the existence of the concept of device pixel ratios, and in particular the `devicePixelRatio` property of the `window` interface. It indicates the ratio of the device's physical pixels to the device's logical pixels. The classic 96 DPI display will have a ratio of 1, but Retina displays and newer smartphone displays will have a ratio of 2 or higher. This leads to our poor canvas pixels being defaced by this need to scale up.

# Y THO

In the race to create better looking displays, device manufacturers are packing more and more pixels into the same space. This breaks traditional ideas about the size of content displayed on said devices. Your phone having a resolution in the ballpark of 1080x1920 doesn't mean that you can view content the same way you did on your physically much larger 1080p monitor. The device pixel ratio is used to "zoom" pixels in, then paint those _logical_ pixels using more _physical_ pixels.

For example, the new Apple iPhone 15 Pro has a 6.1 inch physical screen packing 1179x2556 pixels. These are the physical pixels. However, the size of the viewport reported to web browsers will be 393x852 pixels. These are the logical pixels. This corresponds to a device pixel ratio of 3. If you send a 300x300 image for display on this screen, it will actually need to fill 900x900 physical pixels. 

The HTML canvas element is a raster image, which means it is defined by pixels, like JPEG, PNG, and GIF images. This is true even though some of the things you draw on the canvas are technically vectors in the code (lines, arcs, fills). When those pixels are asked to fill more space, simply scaling them up will cause jaggedness and applying anti-aliasing to prevent the jaggedness results in blurriness. This is what the browser attempts to do by default.

# The power of math and ctx.scale are on our side

We can do better than this default behavior by reading the `devicePixelRatio` and scaling our canvas accordingly before we even draw anything. This property is [well supported by all browsers](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio#browser_compatibility). There are a few things that I tripped over when I tried to understand the code for this.

1. The _internal canvas_ resolution of the canvas element is completely independent of the _external CSS style_ resolution of the canvas element. You can place a 10000x10000 drawing space inside a 100x100 box, resulting in a tiny window into a large world.
2. `ctx.scale(x, y)` can be used to scale that internal resolution up or down, which in our case can be scaled by the `devicePixelRatio` to bring the resolution of our internal canvas up to match the physical pixels of the device.
3. You draw onto this scaled canvas using the logical size, not the pixel size. This means you don't have to adapt the size of things you draw to the ratio. But you do have to keep track of the CSS/logical size of the canvas rather than the width/height attributes you give the canvas, which you may be used to. This tripped me up.

We start with the setup of the objects and values we will need:

```javascript
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');
const rect = canvas.getBoundingClientRect();
const dpr = window.devicePixelRatio;
const size = 300; // example size, we'll use a square canvas
```

Now we set the width and height of the canvas to be relative to the devicePixelRatio:

```javascript
canvas.width = size * dpr;
canvas.height = size * dpr;
```

And we set the CSS size of the canvas to be the logical size, independent of the devicePixelRatio:

```javascript
canvas.style.width = `${size}px`;
canvas.style.height = `${size}px`;
```

We have to scale the canvas by the devicePixelRatio:

```javascript
ctx.scale(dpr, dpr);
```

Now we can draw things on this canvas, based on the 300x300 logical size, and no matter the devicePixelRatio, it will render crisply on the screen!

# Followup 
Change DPR dynamically as it changes (screen move, zoom in)

Rounding errors on canvas - CSS size supports fractions, but Canvas does not.

Throw `image-rendering: pixelated` at it