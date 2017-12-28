## Website Performance Optimization

### Critical rendering path
* [Understanding the critical rendering path, rendering pages in 1 second](https://medium.com/@luisvieira_gmr/understanding-the-critical-rendering-path-rendering-pages-in-1-second-735c6e45b47a)
* [Smooth as Butter: Achieving 60 FPS Animations with CSS3](https://medium.com/outsystems-experts/how-to-achieve-60-fps-animations-with-css3-db7b98610108)

Timeline(CRP):
Styles -> Layout -> Paint -> Composite

* Layout:
This stage is where the browser calculates the size and position of each visible element on the page,
every time an update to the render tree is made, or the size of the viewport changes, the browser has to run layout again.

* Paint:
*When we get to the paint stage, the browser has to pick up the layout result, and paint the pixels to the screen,*
beware in this stage that not all styles have the same paint times,
*also combinations of styles can have a greater paint time than the sum of their parts.*
For an instance mixing a border-radius with a box-shadow, can triple the paint time of an element instead of using just one of the latter.

*To achieve smooth animations we need to focus on changing properties that affect the Composite step, instead of adding this stress to the previous layers.*

Browser set width, height, margins, left/top/right/bottom properties on `Layout` layer.
Hence, if the CSS animation on those properties, browser will need to re-go through the `Layout`, `Paint` and `Comosite`.
It makes the animation not so smooth.

## Related Articles
### [Accelerated Rendering in Chrome](https://www.html5rocks.com/en/tutorials/speed/layers/)
DOM -> RenderLayers -> GraphicsLayers -> GPU

GraphicsLayers creation trigger:
* 3D or perspective transform CSS properties
* <video> elements using accelerated video decoding
* <canvas> elements with a 3D (WebGL) context or accelerated 2D context
* Composited plugins (i.e. Flash)
* Elements with CSS animation for their opacity or using an animated transform
* Elements with accelerated CSS filters
* Element has a descendant that has a compositing layer (in other words if the element has a child element that’s in its own layer)
* Element has a sibling with a lower z-index which has a compositing layer (in other words the it’s rendered on top of a composited layer)

So how does Chrome turn the DOM into a screen image? Conceptually, it:
1. Takes the DOM and splits it up into layers
2. Paints each of these layers independently into software bitmaps
3. Uploads them to the GPU as textures
4. Composites the various layers together into the final screen image.

As should now be clear, the layer-based compositing model has deep implications for rendering performance.

Compositing is comparably cheap when nothing needs to be painted,
so avoiding repaints of layers is a good overall goal when trying to debug rendering perf.

### [CSS Paint Times and Page Render Weight](https://www.html5rocks.com/en/tutorials/speed/css-paint-times/)
### [GPU Accelerated Compositing in Chrome](http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome)
### [Fastersite on web performance](http://gent.ilcore.com/2011/03/how-not-to-trigger-layout-in-webkit.html)
### [Rendering: repaint, reflow/relayout, restyle](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/)
### [Profiling Long Paint Times with DevTools' Continuous Painting Mode](https://developers.google.com/web/updates/2013/02/Profiling-Long-Paint-Times-with-DevTools-Continuous-Painting-Mode)

### Chrome devtool
* [Performance Analysis Reference](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)

### Online free course
* [Udacity](https://classroom.udacity.com/courses/ud884)


