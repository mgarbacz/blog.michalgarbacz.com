---
layout: post
title: "Canvas Rendering Blurry on High Pixel Density Displays"
---

# The Initial Head Scratcher

I was drawing very basic shapes in my canvas element and noticed that their edges rendered incredibly blurry on my Macbook's monitor, but clearly on my external monitor. A bit of Googling led me to discover the existence of the concept of device pixel ratios, and in particular the `devicePixelRatio` property of the `window` interface. It indicates the ratio of the device's physical pixels to the device's logical pixels. The classic 96 DPI display will have a ratio of 1, but Retina displays and newer smartphone displays will have a ratio of 2 or higher. This leads to our poor canvas pixels needing to scale up.

# Y THO

In the race to create better looking displays, device manufacturers are packing more and more pixels into the same space. This breaks traditional ideas about the size of content displayed on said devices. Your phone having a resolution in the ballpark of 1080x1920 doesn't mean that you can view content the same way you did on your physically much larger 1080p monitor. The device pixel ratio is used to define how much to "zoom" pixels in, and then those _logical_ pixels are painted using a larger amount of _physical_ pixels.

For example, the new Apple iPhone 15 Pro has a physical screen comprised of 1179x2556 pixels. These are the physical pixels. However, the size of the viewport reported to web browsers will be 393x852 pixels. These are the logical pixels. This corresponds to a device pixel ratio of 3. If you send a 300x300 image for display on this screen, it will actually need to fill 900x900 physical pixels. 

The HTML canvas element is a raster image (so are JPEGs, PNGs, and GIFs), which means it is defined by a grid of pixels. This is true even though some of the things you draw on the canvas are technically defined by vectors in your code (lines, arcs, fills). When those pixels are asked to fill more space than they originally define, simply scaling them up will cause jaggedness and applying anti-aliasing to prevent the jaggedness results in blurriness. This is what the browser attempts to do by default.

# The power of math and ctx.scale are on our side

We can do better than this default behavior by reading the `devicePixelRatio` and scaling our canvas accordingly before we even draw anything. This property is [well supported by all browsers](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio#browser_compatibility). There are a few things to understand before we look at the code for this:

1. The _internal canvas_ resolution of the canvas element is completely independent of the _external CSS style_ resolution of the canvas element. You can place a 10000x10000 drawing space inside a 100x100 box, resulting in what looks like a tiny window into a large world.
2. The `scale(x, y)` method for the `CanvasRenderingContext2d` interface can be used to scale that internal canvas resolution up or down. In our case, it can be scaled by the `devicePixelRatio` to bring the resolution of our internal canvas up to match the physical pixels of the device.
3. You draw onto this scaled canvas using the external/logical size, not the internal/physical size. This means you don't have to adapt the size of things you draw to the ratio. You may be used to keeping track of the width/height attributes you set for the canvas to draw on it, but for this setup we need to use the CSS style sizes instead. This tripped me up.

The resulting code will use the width/height attributes of the canvas to define an internal resolution that matches 1:1 the physical pixels of the device, and use the CSS style width/height of the canvas to define an external resolution that matches 1:1 the logical pixels of the device.

We start with the setup of the objects and values we will need:

```javascript
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');
const dpr = window.devicePixelRatio;
const width = 300; // just an example size
const height = 300; // just an example size
```

Now we set the width and height of the canvas to be relative to the devicePixelRatio:

```javascript
canvas.width = width * dpr;
canvas.height = height * dpr;
```

And we set the CSS size of the canvas to be the size we want it to be on the screen, independent of the devicePixelRatio:

```javascript
canvas.style.width = `${width}px`;
canvas.style.height = `${height}px`;
```

Finally, we have to scale the canvas by the devicePixelRatio:

```javascript
ctx.scale(dpr, dpr);
```

Now we can draw things on this canvas, based on the 300x300 logical size, and no matter the devicePixelRatio, it will render crisply on the screen!

# Demo

I've created a little demonstration that compares the various scenarios possible here in order to illustrate the points being made. You can view it at [https://mgarbacz.github.io/webdev-toolbox/scaling-canvas/](https://mgarbacz.github.io/webdev-toolbox/scaling-canvas/). I draw a shape with some lines and a curve in it in three different configurations. _Note: You can only get the full idea of what this demo is presenting by viewing it on a screen that has a DPR greater than 1._

The __With DPR, With Scaling__ section shows a 300x300 canvas that is scaled up by the DPR on both the internal size and the context scale. You can see that the lines are crisp and the shape is drawn as if the canvas was 300x300.

The __W/o DPR, W/o Scaling__ section shows a 300x300 canvas that hasn't been augmented by the DPR at all. You can see that the lines are blurred in places, but otherwise the shape is drawn as if the canvas was 300x300.

The __With DPR, W/o Scaling__ section shows a 300x300 canvas that is scaled up by the DPR on the internal size, but without the context scale being set. You can see that the lines are crisp, but the shape is drawn as if the canvas was 600x600.

Source code for this demo can be found [here on Github](https://github.com/mgarbacz/webdev-toolbox/tree/main/scaling-canvas).

# Followup

There are a few things I plan to follow up on.

- The DPR can change: It is worth making sure you change the scaling from the DPR dynamically as it can change when the window is moved to a screen with a different ratio or the screen is zoomed in. [MDN has an example](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio#monitoring_screen_resolution_or_zoom_level_changes) about this very thing.

- Rounding errors on canvas: It turns out that the CSS size for the canvas element supports fractions, but internal canvas dimensions do not. Those get rounded. [This answer on Stackoverflow](https://stackoverflow.com/a/54027313/937718) by [spenceryue](https://stackoverflow.com/users/3624264/spenceryue) goes into detail on this issue and proposes a solution.

- Throw `image-rendering: pixelated` at it: I tried this out briefly, and it seemed to work. You can toss all this DPR/scale stuff out and supposedly just add this property to the canvas element. It should prevent the anti-aliasing from creating the blurriness present from the default upscaling. I'd have to take the time to understand what it is doing and test it fully. A few vague details about it [here on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/image-rendering).