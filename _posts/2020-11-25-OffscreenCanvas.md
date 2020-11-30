---
layout: post
title: How I optimized image cropping process via OffscreenCanvas
---

AIMMO Enterprise, where I had been working, provides various annotating tools.
I took a major role in creating Box, Polygon, Landmark, Polyline, Rotatable-Box, Semantic-Segmentation, and Object Tracking annotation tool.

The Object Tracking tool is a tool where the user can annotate an object, frame by frame from sequential images, easily and effectively.
It consists of two major areas, the workspace area where the user interacts with the mouse to create certain shapes(aka. Item),
the side area where it displays annotated Items with the thumbnail image of the shape drawn area and metadata that describes its attribute.

At first, I cropped an image via Canvas based on the corresponding coordinate of its item.
There were no performance issues until the to-be annotated image resolution exceeded more than 3000*3000 pixels
and where it had a large amount of annotated Items such as 300 shapes in one image.

A bottleneck occurred when the user entered the tool initially.
Since image cropping was processed on the main thread synchronously,
the user had to wait for the thumbnail to be prepared and also had a lagging moment interacting with the workspace area while on it.

![_config.yml]({{ site.baseurl }}/images/mainThreadBoxGif.gif)
*Image cropping processed on the main thread*

Above GIF shows what users have to go through when image cropping is processed on the main thread.
Mouse moving some point to point diagonally indicates that the user is attempting to create a new box.
Even though the tool seems to be ready for user interactions right after the main image is loaded,
it is not possible for the user to create one.
Thus, leads to a bad user experience.

![_config.yml]({{ site.baseurl }}/images/workerThreadBoxGif.gif)
*Image cropping processed by OffscreenCanvas on the worker thread*

On contrary, the above GIF show what happens if image cropping is processed on a worker thread.
Right after the main image is loaded, the tool is ready for user interactions and the user does not have to wait to create a new box
while more than 300 thumbnails are being created via OffscreenCanvas.

#### Worth reading documentations
[https://developers.google.com/web/updates/2018/08/offscreen-canvas](https://developers.google.com/web/updates/2018/08/offscreen-canvas)

[https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)

[https://developers.google.com/web/updates/2011/12/Transferable-Objects-Lightning-Fast](https://developers.google.com/web/updates/2011/12/Transferable-Objects-Lightning-Fast)

[https://developer.mozilla.org/en-US/docs/Web/API/NavigatorConcurrentHardware](https://developer.mozilla.org/en-US/docs/Web/API/NavigatorConcurrentHardware)
