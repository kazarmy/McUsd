# McUsd
Simple [USD](https://graphics.pixar.com/usd/release/index.html) scene geometry with a variety of [UsdPreviewSurface](https://graphics.pixar.com/usd/release/spec_usdpreviewsurface.html) materials applied.

Download this repository and then load the McUsd.usda file in the models directory into your favorite USD file viewer.

The "Mc" is for Minecraft, not McDonalds. Though that's fine if you think the latter. I'd be happy if this file was served billions of times to help others.

## Goals

The overarching goal is to help the community strive to give similar or the same renderings, as possible. Consistent interpretation of [the UsdPreviewSurface material description](https://graphics.pixar.com/usd/release/spec_usdpreviewsurface.html) benefits us all. While, as of August 2022, this description is still a "proposal," numerous companies have implemented it in their systems.

This material test file has a few purposes:
* Help people implementing UsdPreviewSurface material viewers, providing some reasonably complex surfaces that can highlight common bugs.
* Give examples in the human-readable USDA format to help understanding.
* Note loose areas of the current specification, to help encourage these becoming fully specified.
* Show the state of various implementations of UsdPreviewSurface, in order to determine areas where the specification is not yet followed.

You're encouraged to move around the model and render from different viewpoints, to change the lights to suit you (an area where USD will someday have more physical lighting units, making interchange cleaner), and in general modify what you need. The model is first and foremost meant as a aid in testing and debugging UsdPreviewSurfaces.

You're also very much encouraged to [_send me your own results_](mailto:erich@acm.org), or submit as a pull request. I am happy to add to the gallery any test images you generate and notes on them you might have.

The two lights included are a sun "DistantLight", Euler angles of rotate around X 235 degrees, Y 325 degrees, intensity 30, width angle 1 degree, and a light "DomeLight" with sRGB texture _domelight.png, Z rotate 270, intensity 6. As noted in "Conclusions", these lights are almost always interpreted differently in different applications, if translated at all.

This model was generated by laying down a few blocks in Classic (Java) Minecraft, then using [Mineways](http://mineways.com) to export the model to USDA format. For the exact export settings, see the top of the McUsd.usda file itself. The [JG-RTX resource pack](https://github.com/jasonjgardner/jg-rtx), released under the Creative Commons Attribution-ShareAlike 4.0 International Public
License, is used at a resolution of 256x256 textures for the surfaces. Textures are of four possible types:
* Albedo/Opacity - sRGB color RGBA file with no suffix
* Normal - linear RGB vector directions in tangent space, mapped from (-1,-1,-1) to (1,1,1), suffix "_n.png"
* Roughness - grayscale roughness mapped to 0.0 (smooth) to 1.0 (rough), suffix "_r.png"
* Metallic - metalness, typically binary, mapped to 0.0 (non-metallic) to 1.0 (metallic), suffix "_m.png"
* Emissive - sRGB emissive color, suffix "_e.png"

Parts of the UsdPreviewSurface specification not tested by this model include the path "useSpecularWorkflow 1", and the clearcoat-related, displacement, and occlusion values.

Note: I currently work for NVIDIA. However, I am making this simple USD file as an independent, free test, separate from NVIDIA. Having worked through the UsdPreviewSurface proposal myself for my free program Mineways, I hope this test file will save others time and effort, while also helping to independently push for a more fleshed-out specification of USD, to widen its appeal. I would be happy to see a UsdPreviewSurface conformance suite of USD files and images generated from some more knowledgeable source. If there is such a thing, please point me at it.

I was inspired to take a bit of time to make this project due to hearing at SIGGRAPH 2022 about the pickup of USD in [open metaverse initiatives](https://cesium.com/building-the-open-metaverse-siggraph-2022/) and [ASWF's USD efforts](https://www.aswf.io/news/academy-software-foundation-launches-digital-production-example-library-as-newest-project-to-house-production-grade-content/). More important, the [USD Working Group](https://github.com/AcademySoftwareFoundation/wg-usd) is open to participation by all. Their openness inspired this effort.

## Conclusions on UsdPreviewSurface

I'm putting conclusions first, since what follows is an extensive set of tests for a variety of applications.

One notable problem with UsdPreviewSurface as of August 2022 is that the "emissiveColor" is minimally specified as "Emissive component." It is unclear whether the color is meant to be specified as the material's on-screen appearance, or meant to be specified in, say, nits. These are actually two questions: 1) how does an object with an emissive color appear when directly viewed? and 2) how does this emission color work with other lights? For the first question, it is simple to say that the emissiveColor should be treated as a fixed color for the surface, the color that is always shown. However, USD has [an elaborate camera model](https://graphics.pixar.com/usd/dev/api/class_usd_geom_camera.html), including an exposure attribute, which implies that the appearance of the light should change as the exposure changes.

In McUsd I use a [nits interpretation](http://www.realtimerendering.com/blog/physical-units-for-lights/), since that is what worked well with Omniverse. The emissive texture is scaled up by 1000 (nits) by using the "scale" input so that it gives off a reasonable amount of light to surrounding objects. This is unlikely to be the standard way in the future, though currently there is no standard way.

This question of magnitude for lighting is part of a larger question, how physical lights are specified in USD. Currently [UsdLux](https://graphics.pixar.com/usd/release/api/usd_lux_page_front.html) and related light specifications use a film-related relative pair of values, ["exponent and intensity"](https://rmanwiki.pixar.com/display/REN23/PxrMeshLight), not tied to any physical units.

The "roughness" input is loosely specified as of August 2022. It says "This value is usually squared before use with a GGX or Beckmann lobe." Choosing the square of the roughness, which is the common usage in the Burley model (see [page 14](https://disneyanimation.com/publications/physically-based-shading-at-disney/)), is the common way to go. GGX is also the common choice, from my experience. The standard setters need to choose, for better consistency among applications. As an example, [glTF has chosen roughness-squared and GGX](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#material-structure) - see that section for their reasons.

I would appreciate some recommendations about how opacityThreshold should be set for cutouts. A value of 0.0 means that the alpha (transparency) value is indeed treated as semitransparency. An opacityThreshold greater than 0.0 gives a level where the alpha is compared and judged to be fully transparent (if below this value) or fully opaque (if above or equal to this value). This is clear enough, but I suggest that the specification note an opacityThreshold of 0.5 is a common cutoff value. As an example, using an RGBA texture authored for use as a billboard, here is it rendered with an opacityThreshold of 0.01, 0.5, and 0.99. The 0.5 value is, in my opinion, the best render of the three.

![opacityThreshold 0.01](/images/opacity_0.01.png "opacityThreshold 0.01")
![opacityThreshold 0.5](/images/opacity_0.5.png "opacityThreshold 0.5")
![opacityThreshold 0.99](/images/opacity_0.99.png "opacityThreshold 0.99")

## Application Test Results

What follows are images generated by various rendering systems, for comparison. I have tried to use the newest version of the software for each package.

Note that few images will match perfectly from system to system. Reasons include:
* Lighting is not translated in a compatible manner, or not at all, so is added manually.
* Camera position and orientation not translated.
* The opacityThreshold material attribute is not implemented fully.
* The renderer does not support some reflective materials.
* The renderer does not support emissive materials.
* The renderer supports emissive materials, but the emissive values are too bright/dim.

These various conditions and others will be noted after each rendering, as best as I can determine them. The last problem, emissive material definition, is explained in detail in the "Conclusions" section above.

Please note that the purpose of this project is not to show problems in a particular application, but rather for me to see what features there might be confusion on (due to the specification or implementation) and to understand and snapshot the level of progress at this time. In my perfect world all applications would get almost the same results, within rendering algorithm limitations.

### Omniverse Create

I focused on [Omniverse Create](https://www.nvidia.com/en-us/omniverse/apps/create/) as the target for this USD test file, as Mineways is a "connector" for the Omniverse system, and [Omniverse is free](https://www.nvidia.com/en-us/omniverse/download/) and far along in its rendering of UsdPreviewSurface. This is important to understand: this McUsd test file is tailored to look particularly good in Omniverse. Some application had to be targeted, as a starting spot. Doing so helps point out the differences between its interpretation of materials and lights compared to other applications. No application is particularly "right" at this point.

Load procedure: drag and drop McUsd.usda file into the viewport of Omniverse Create.

There are [a few renderers in Omniverse](https://docs.omniverse.nvidia.com/prod_kit/prod_materials-and-rendering/render-settings_overview.html), along with the hydra renderer Pixar Storm being included as an option. All results are from Omniverse Create 2022.3.0-beta.5.

#### Omniverse RTX - Interactive (Path Tracing)

Load procedure: Nothing further. By default, the "RTX - Interactive (Path Tracing)" is used.

This renderer is progressive, shooting more and more frames of rays and blending these results in. Here is the render after around 140 frames or so:

![Omniverse RTX - Interactive (Path Tracing)](/images/ov_interactive.png "Omniverse RTX - Interactive (Path Tracing)")

#### Omniverse RTX - Real-Time

This renderer includes some ray tracing elements.

Load procedure: with the model loaded, in the upper left corner of the viewport is a light-bulb icon with a renderer name. Choose the "RTX - Real-Time" renderer from this dropdown list.

![Omniverse RTX - Real-Time](/images/ov_real_time.png "Omniverse RTX - Real-Time")

Note simplifications occur, such as opaque shadows for semitransparent objects.

#### Omniverse RTX - Accurate (Iray)

Uses the Iray ray tracer. Here is the render after around 140 frames:

Load procedure: with the model loaded, in the upper left corner of the viewport is a light-bulb icon with a renderer name. Choose the "RTX - Real-Time" renderer from this dropdown list.

![Omniverse RTX - Accurate (Iray)](/images/ov_accurate.png "Omniverse RTX - Accurate (Iray)")

Differences with the interactive version include less color on the semitransparent glass block on the left. Though not obvious, the lava is a bit dimmer. In the "Accurate" render a bit of a noisy caustic can be seen on the grass to the right of the gold block.

#### Omniverse Pixar Storm

Load procedure: with the model loaded, in the upper left corner of the viewport is a light-bulb icon with a renderer name. Choose the "Pixar Storm" renderer from this dropdown list.

![Omniverse Pixar Storm](/images/ov_pixar_storm.png "Omniverse Pixar Storm")

Some clear problems: lights are ignored, opacityThreshold is not implemented for cutout objects, metallic seems to have no effect, etc.

### Sketchfab

Load procedure: upload a zip file of the whole "model" directory to your account. Modifications: the directional light was set to 250 degrees, the camera FOV to 30, and the camera view itself manually adjusted to about match.

The Sketchfab rendering can be [directly examined on their site](https://skfb.ly/oxyUE).

![Sketchfab](/images/sketchfab.png "Sketchfab")

Sketchfab does not translate the camera or lights. It uses rasterization and related techniques for interactive rendering, so giving typical limitations: the lava does not emit light, the glass block does not cast a shadow, true reflections are not generated for shiny surfaces. There are some interesting specular highlights on the glass block that are not visible in the Omniverse renderings.

---
## License

**[CC-NC-BY-SA](LICENSE)**

Textures from the [JG-RTX resource pack](https://github.com/jasonjgardner/jg-rtx), which has the same license.

---
# Contact
Email [me](http://erichaines.com) at [erich@acm.org](mailto:erich@acm.org).