# McUsd
Simple [USD](https://graphics.pixar.com/usd/release/index.html) scene geometry with a variety of [UsdPreviewSurface](https://graphics.pixar.com/usd/release/spec_usdpreviewsurface.html) materials applied, for casual material and light testing.

Download this repository and then load the McUsd.usda file in the models directory into your favorite USD file viewer. Or, [view the model in your browser](https://skfb.ly/oxyUE) through Sketchfab. Or, have an phone? Try out [Sketchfab's USDZ translation](https://erich.realtimerendering.com/mcusd/McUsd_sketchfab.usdz). You'll have to shrink it down in AR mode (pinch, on an iPhone) - each block is a meter in size!

The "Mc" is for Minecraft, not McDonalds. Short URL for this page: http://bit.ly/gitmcusd

![McUsd: JG-RTX textures, rendered in Omniverse](/images/ov_interactive.png "McUsd: JG-RTX textures, rendered in Omniverse")

Clockwise from "12 o'clock": diamond block, iron block, gold block, fern, prismarine, sunflower, purple stained glass, rails, chiseled quartz block atop quartz pillar, piston, and in the middle is lava.

## Goals

The overarching goal is to help the community strive to give similar or the same renderings, as possible. Consistent interpretation of [the UsdPreviewSurface material description](https://graphics.pixar.com/usd/release/spec_usdpreviewsurface.html) benefits us all. While, as of August 2022, this description is still a "proposal," numerous companies have implemented it in their systems.

This material test file has a few purposes:
* Help people implementing UsdPreviewSurface material viewers, providing a simple model with some reasonably complex surfaces that can test for common bugs.
* Give examples in the human-readable USDA format to help understanding.
* Note loose areas of the current specification, to help encourage these becoming fully specified.
* Show the state of various implementations of UsdPreviewSurface, in order to determine areas where the specification is not yet followed.

There are [more serious efforts at compatibility test suites](https://developer.nvidia.com/blog/universal-scene-description-as-the-language-of-the-metaverse/) happening in the USD community, e.g., see [this video](https://www.nvidia.com/en-us/on-demand/session/siggraph2022-sigg22-s-20/) at 23:09 on. My small effort here is to give a _simple_ test scene, now, with some interesting materials, and note some problems I've seen in testing with it.

You're encouraged to move around the model and render it from different viewpoints, to change the lights or other attributes to suit you (an area where USD will someday have more physical lighting units, making interchange cleaner), and in general modify what you need. The model is first and foremost meant as a aid in testing and debugging UsdPreviewSurfaces.

You're also very much encouraged to [_send me your own results_](mailto:erich@acm.org), or submit as a pull request. I am happy to add to the gallery any test images you generate and notes on them you might have.

The two lights included are a sun "DistantLight", Euler angle order ZXY, angles of rotation are X 235 degrees, Y 325 degrees, intensity 30, width angle 1 degree, and a light "DomeLight" with sRGB texture _domelight.png, Z rotate 270, intensity 6. As noted in "Observations", these lights are almost always interpreted differently in different applications, if translated at all.

This model was generated by laying down a few blocks in Classic (Java) Minecraft, then using [Mineways](http://mineways.com) to export the model to USDA format. For the exact export settings, see the top of the McUsd.usda file itself. The [JG-RTX resource pack](https://github.com/jasonjgardner/jg-rtx), released under the Creative Commons Attribution-ShareAlike 4.0 International Public
License, is used at a resolution of 256x256 textures for the surfaces. Textures are of four possible types:
* Albedo/Opacity - sRGB color RGBA file with no suffix
* Normal - linear RGB vector directions in tangent space, mapped from (-1,-1,-1) to (1,1,1), suffix "_n.png"
* Roughness - grayscale roughness mapped to 0.0 (smooth) to 1.0 (rough), suffix "_r.png"
* Metallic - metalness, typically binary, mapped to 0.0 (non-metallic) to 1.0 (metallic), suffix "_m.png"
* Emissive - sRGB emissive color, suffix "_e.png"

Parts of the UsdPreviewSurface specification not tested by this model include the path "useSpecularWorkflow 1", and the clearcoat-related, displacement, and occlusion values.

Note: I currently work for NVIDIA. However, I am making this simple USD file as an independent, free test model. Having worked through the UsdPreviewSurface proposal myself for my free program Mineways, I hope this test file will save others time and effort, while also helping to independently push for a more fleshed-out specification of USD, to widen its appeal. I would be happy to see a UsdPreviewSurface conformance suite of USD files and images generated from some more knowledgeable source. If there is such a thing, please point me at it.

I was inspired to take a bit of time to make this project due to hearing at SIGGRAPH 2022 about the pickup of USD in [open metaverse initiatives](https://cesium.com/building-the-open-metaverse-siggraph-2022/) and [ASWF's USD efforts](https://www.aswf.io/news/academy-software-foundation-launches-digital-production-example-library-as-newest-project-to-house-production-grade-content/). More important, the [USD Working Group](https://github.com/AcademySoftwareFoundation/wg-usd) is open to participation by all. Their openness inspired this effort. In particular, I'm told that the [Assets working group](https://github.com/usd-wg/assets/) is best placed to discuss these issues.

## Observations on UsdPreviewSurface

I'm putting a summary of my observations first, since what follows is an extensive set of tests for a variety of applications.

One notable problem with [UsdPreviewSurface](https://graphics.pixar.com/usd/release/spec_usdpreviewsurface.html) as of August 2022 is that the "emissiveColor" is minimally specified as "Emissive component." It is unclear whether the color is meant to be specified as the material's on-screen appearance, or meant to be specified in, say, nits. These are actually two questions: 1) how does an object with an emissive color appear when directly viewed? and 2) how does this emission color work with other lights? For the first question, it is simple to say that the emissiveColor should be treated as a fixed color for the surface, the color that is always shown. However, USD has [an elaborate camera model](https://graphics.pixar.com/usd/dev/api/class_usd_geom_camera.html), including an exposure attribute, which implies that the appearance of the light should change as the exposure changes.

In McUsd I use a [nits interpretation](http://www.realtimerendering.com/blog/physical-units-for-lights/), since that is what worked well with Omniverse. The emissive texture is scaled up by 1000 (nits) by using the "scale" input so that it gives off a reasonable amount of light to surrounding objects. This is unlikely to be the standard way in the future, as can be seen in the tests that follow, though currently the feature is not specified.

This question of magnitude for lighting is part of a larger question, how physical lights are specified in USD. Currently [UsdLux](https://graphics.pixar.com/usd/release/api/usd_lux_page_front.html) and related light specifications use a film-related relative pair of values, ["exponent and intensity"](https://rmanwiki.pixar.com/display/REN23/PxrMeshLight), not tied to any physical units.

The "roughness" input is loosely specified as of August 2022. It says "This value is usually squared before use with a GGX or Beckmann lobe." Choosing the square of the roughness, which is the common usage in the Burley model (see [page 14](https://disneyanimation.com/publications/physically-based-shading-at-disney/)), is the common way to go. GGX is also the common choice, from my experience. The standard setters need to choose, for better consistency among applications. As an example, [glTF has chosen roughness-squared and GGX](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#material-structure) - see that section for their reasons. I would suggest simply lifting glTF's equations and descriptions of their implementation, saving work and also making USD and glTF more compatible.

Mentioning how opacityThreshold should normally be set for cutouts in the proposal would help new implementers and users. A value of 0.0 means that the alpha (transparency) value is indeed treated as semitransparency. An opacityThreshold greater than 0.0 gives a level where the alpha is compared and judged to be fully transparent (if below this value) or fully opaque (if above or equal to this value). This is clear enough, but I suggest that the specification note an opacityThreshold of 0.5 is a common cutoff value. As an example, using an RGBA texture authored for use as a billboard, here is it rendered with an opacityThreshold of 0.01, 0.5, and 0.99. The 0.5 value is, in my opinion, the best render of the three, and about the most logical.

![opacityThreshold 0.01](/images/opacity_0.01.png "opacityThreshold 0.01")
![opacityThreshold 0.5](/images/opacity_0.5.png "opacityThreshold 0.5")
![opacityThreshold 0.99](/images/opacity_0.99.png "opacityThreshold 0.99")

The application of normal textures to surfaces seems a little off by default. Some guidance from the proposal would be helpful. For example, the McUsd.usda model needs to set the bias and scale for every normal texture as follows:

    float4 inputs:bias = (-1, 1, -1, -1)
    float4 inputs:scale = (2, -2, 2, 2)

Having to negate the second, Y, value in each is confusing. The normal textures used (created by someone else, and used in Minecraft RTX as-is) seem fairly standard to me, but I am not an expert. Some warning about this possible negation would be useful.

## Application Test Results

What follows are images generated by various rendering systems, for comparison. I have tried to use the newest version of the software for each package. My focus is running applications on Windows 10.

Note that few images will match perfectly from system to system. Reasons include:
* Lighting is not translated in a compatible manner, or not at all, so is added manually.
* Camera position and orientation not translated.
* The opacityThreshold material attribute is not implemented fully.
* The renderer does not support some reflective materials.
* The renderer does not support emissive materials.
* The renderer supports emissive materials, but the emissive values are too bright/dim.

These various conditions and others will be noted after each rendering, as best as I can determine them. The last problem, emissive material definition, is explained in detail in the "Observations" section above.

Please note that the purpose of this project is not to show problems in a particular application, but rather for me to see what features there might be confusion on (due to the specification or implementation) and to understand and snapshot the level of progress at this time. Noting problems is not meant as criticism, but rather as things to be aware of if you use a package. Many of the applications have improved their import capabilities considerably in the past two years.

## USDView

The usdview program from the [USD Toolset](https://graphics.pixar.com/usd/release/toolset.html) includes a basic hydra GL rasterizing renderer. It's about as basic a render you can make, but it's also the standard, in that it's the renderer Pixar provides. As such, it properly renders semitransparency, cutouts, roughness, metalness, etc.

It is possible to build usdview from scratch, but in that way lies madness (at least for me). Happily, [NVIDIA's Omniverse Launcher](https://www.nvidia.com/en-us/omniverse/) provides a pre-built USDView. I tested with version 0.22.8.

Load procedure: File -> Open, then hold down Alt and use the mouse buttons to rotate, pan, and dolly.

![UsdView](/images/usdview.png "UsdView")

As expected from a basic rasterizer, shadows, reflections, and emitted light from surfaces are not rendered.

By default, USDView adds a light "at the eye", which is shown in the rendering above. This additional light can be turned off via View -> Lights -> Camera Light.

Without shadows, it is a little difficult to tell if the DistantLight in McUsd.usda is being used. By pressing "F11" (or View -> Toggle Viewer Mode), we can see that the DistantLight and DomeLight have been read in (the camera has not). These lights can be toggled off by right-clicking and selecting "Make Invisible". Doing so, the DomeLight appears to have no effect, neither to direct illumination nor as a background environment map.

The lava light source seems oversaturated. As discussed in "Observations", the emissive texture is scaled up by a factor of 1000. Removing this scale factor, which is done in the file McUsd_unscaled_lava.usda, the lava looks more reasonable:

![UsdView unscaled lava](/images/usdview_unscaled_lava.png "UsdView unscaled lava")

From what I can see, the emissiveColor is used to shade the lava. More experimentation could be done here, e.g., removing the diffuseColor texture and setting the diffuseColor to white or black, to see the effect. I've done this experiment and get the same rendering, showing that the emissionColor, when present, is used instead of the diffuseColor. However, it seems better to have the emissiveColor's role in rendering specified, rather than needing to reverse engineer how the effect is implemented in USDView. I believe this program should not be held up as an oracle and final arbiter for how attributes are applied. Better, for me, would be the explicit set of equations used to compute local shading, such as how [glTF does it](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#appendix-b-brdf-implementation).

## Omniverse Create

I focused on [Omniverse Create](https://www.nvidia.com/en-us/omniverse/apps/create/) as the target for this USD test file, as Mineways is a "connector" for the Omniverse system, and [Omniverse is free](https://www.nvidia.com/en-us/omniverse/download/) and far along in its rendering of UsdPreviewSurface. This is important to understand: this McUsd test file is tailored to look particularly good in Omniverse. Some application had to be targeted, as a starting spot. Doing so helps point out the differences between its interpretation of materials and lights compared to other applications. No application is particularly "right" at this point.

Load procedure: drag and drop McUsd.usda file into the viewport of Omniverse Create.

There are [a few renderers in Omniverse](https://docs.omniverse.nvidia.com/prod_kit/prod_materials-and-rendering/render-settings_overview.html), along with the hydra renderer Pixar Storm being included as an option. All results are from Omniverse Create 2022.3.0-beta.5.

### Omniverse RTX - Interactive (Path Tracing)

Load procedure: Nothing further. By default, the "RTX - Interactive (Path Tracing)" is used.

This renderer is progressive, shooting more and more frames of rays and blending these results in. Here is the render after around 140 frames or so:

![Omniverse RTX - Interactive (Path Tracing)](/images/ov_interactive.png "Omniverse RTX - Interactive (Path Tracing)")

One notable difference with USDView is that, in these Omniverse renderings, the prismarine block (the blue stone block) does not have as strong specular highlights in this and other Omniverse views. I think the USDView version looks more convincing, but do not know which is correct.

The lava is an emitter and affects how the other objects are illuminated. Here is a rendering with the sun and surrounding dome light off:

![Omniverse RTX - path traced lava](/images/ov_interactive_lava.png "Omniverse RTX - path traced lava")

### Omniverse RTX - Real-Time

This renderer includes some ray tracing elements.

Load procedure: with the model loaded, in the upper left corner of the viewport is a light-bulb icon with a renderer name. Choose the "RTX - Real-Time" renderer from this dropdown list.

![Omniverse RTX - Real-Time](/images/ov_real_time.png "Omniverse RTX - Real-Time")

Note simplifications occur, such as opaque shadows for semitransparent objects.

### Omniverse RTX - Accurate (Iray)

Uses the Iray ray tracer. Here is the render after around 140 frames:

Load procedure: with the model loaded, in the upper left corner of the viewport is a light-bulb icon with a renderer name. Choose the "RTX - Real-Time" renderer from this dropdown list.

![Omniverse RTX - Accurate (Iray)](/images/ov_accurate.png "Omniverse RTX - Accurate (Iray)")

Differences with the interactive version include less color on the semitransparent glass block on the left. Though not obvious, the lava is a bit dimmer. In the "Accurate" render a bit of a noisy caustic can be seen on the grass to the right of the gold block.

### Omniverse Pixar Storm

Load procedure: with the model loaded, in the upper left corner of the viewport is a light-bulb icon with a renderer name. Choose the "Pixar Storm" renderer from this dropdown list.

![Omniverse Pixar Storm](/images/ov_pixar_storm.png "Omniverse Pixar Storm")

Some clear problems: lights are ignored, opacityThreshold is not implemented for cutout objects, metallic seems to have no effect, etc.

## Sketchfab

Load procedure: upload a zip file of the whole "model" directory to your account. Modifications: the directional light was set to 250 degrees, the camera FOV to 30, and the camera view itself manually adjusted to about match.

The Sketchfab rendering can be [directly examined on their site](https://skfb.ly/oxyUE).

![Sketchfab](/images/sketchfab.png "Sketchfab")

Sketchfab does not translate the camera or lights. It uses rasterization and related techniques for interactive rendering, so giving typical limitations: the lava does not emit light, the glass block does not cast a shadow, true reflections are not generated for shiny surfaces. There are some interesting specular highlights on the glass block that are not visible in the Omniverse renderings.

Sketchfab lets you download different translations of your model. I downloaded the [Sketchfab USDZ translation](https://erich.realtimerendering.com/mcusd/McUsd_sketchfab.usdz) - click that link on an iPhone to view it. I will not swear this translation is perfect, but it works surprisingly well on the next app. The camera position set in Sketchfab is exported. The default Sketchfab light sources are not.

## iPhone

On Safari and Chrome (and perhaps other browsers), if you are using an iPhone or iPad and click on a ".usdz" extension file, the file displays in an viewer called AR Quick Look. [Some examples are here](https://developer.apple.com/augmented-reality/quick-look/).

The [Sketchfab USDZ translation](https://erich.realtimerendering.com/mcusd/McUsd_sketchfab.usdz) mentioned above can also be viewed - click the link to see it (other phones and non-Safari browsers will just download the file instead; if you know how to hook the usdz file to immediately view, let me know).

This viewer has two modes: AR and Object, shown at the top. "Object" lets you view the model in isolation. In this mode, one finger rotates, two finger pinch dollies (aka "zooms", but not really) the camera in and out. There appears to be no way to change the center of focus. Here's an example:

![iPhone Object view](/images/iphone_object.png "iPhone Object view")

Understandably, the camera setting in the Sketchfab usdz file is not used in either mode. If you look closely, there is some "ghosting" around the sunflower, an area around it where the cut-away parts of the texture should have no effect, but instead leave a faint white trace.

Try AR mode. Because this model is actually to scale, the blocks are each 1 meter across by default, so the model is likely much larger than where you are. You made need to first move your phone or tablet around so that it understands your environment before placing the model. After this, use a two-finger pinch gesture to shrink the model down and make it fit your environment. One finger lets you move the model horizontally. Here is the scene shrunk to about 10% and viewed in place, using "AR":

![iPhone AR view](/images/iphone_ar.png "iPhone AR view")

In both views you can see that the cutout sunflower head has rendering problems. If you rotate the view, various parts of the sunflower disappear and appear. My guess is that this artifact is likely caused by cutouts being rendered by z-sorting them with other objects in the scene, but I can't say I understand. The "ghosting" visible in Object mode is not present in AR mode.

## Blender

Being free and open source, Blender is easy to test. I tried the Blender 3.3 beta and 3.4 alpha from [here](https://builder.blender.org/download/daily/), and also the Blender 3.2 alpha USD branch, installer downloadable through the [Omniverse Launcher](https://www.nvidia.com/en-us/omniverse/). These all gave the same renderings shown below.

Load procedure: File -> Import -> Universal Scene Description. Choose Camera.001 in the hierarchy control in the upper right, click the green camera icon next to it, then type [Ctrl+Numpad 0](https://blender.stackexchange.com/questions/3502/how-can-i-make-a-camera-the-active-one) to set the view to this camera. Choose "viewport shading" in the upper right corner of the viewport. The result:

![Blender initial view](/images/blender_initial.png "Blender initial view")

Blender added a light on import, at the bottom of the hierarchy list in the upper right. Turn it off by clicking on the eye icon to the right of this "Light" object to turn it off. The Sun (DistantLight) is imported by Blender, but is too bright. Find it in the hierarchy and change its strength below to, say, 1.0. This at least gives a colorful image:

![Blender adjusted view](/images/blender_adjusted.png "Blender adjusted view")

Clearly, Blender is currently removing textures attached to surfaces. Interestingly, it appears to read the color textures applied and computes average colors to use in their stead. The DistantLight was mostly translated properly, with angles X=-125 and Y=-35, equivalent to the specified angles of 235 and 325. The Euler order for the angles was not changed to ZXY (though in this case it doesn't matter, since Z==0). A view showing the shadows produced confirms that the light's location is correct:

![Blender shadow view](/images/blender_shadow.png "Blender shadow view")

## Unreal Editor

You can install the Unreal Editor by running the [Epic Games Launcher](https://store.epicgames.com/en-US/download) and then selecting "Unreal" in the upper left column, "Library" along the top, then the "+" to the right of "engine versions". Note that the 5.0.3 editor needs 114 GB of disk space. Images here were generated with Unreal Engine 5.0.3, using the Beta Version 1.0 Usd Importer plugin.

Load procedure: To install [the native USD plugin](https://docs.unrealengine.com/5.0/en-US/universal-scene-description-in-unreal-engine/), choose "Edit -> Plugins" in the menus at the top. Search "USD" and you'll find two (or four, if the Omniverse Connector is installed - both plugins evidently can co-exist, unlike Maya or Max). Enable the USD Importer by checking its box and restart the editor, as directed. [See here](https://docs.unrealengine.com/5.0/en-US/universal-scene-description-in-unreal-engine/) for more help.

To load In Unreal Editor create a New Project. Choose the Games template on the left, Blank, and for Project Defaults in the lower right you could check the Raytracing box. After you click "Create" the editor may take a few minutes to compile shaders. You'll get a scene with some chairs, etc. I go to the Outliner in the upper right and delete all these StaticMeshes.

Use File -> Import Into Level... You should see "*.usd" in the lower right of the dialog that appears (if you don't, the USD plugin is not enabled). Choose wherever you like to put the imported USD assets. Hit "Import". Mouse scroll to dolly out of the scene.

For my renders I deleted the given Light Source and added a distant light source, angle Y=235 and Z=325, to match the McUsd.usda's lighting, which does not import. I also used the default SkyLight and BP_Sky_Sphere in the scene, turning off much else. The imported camera has the right position, but I could not figure out how to set its field of view to be narrower (Focal Length and Aperture were not taken from the McUsd.usda file, forcing a FOV of 92.67 degrees), so I dollied in.

The result, after positioning the camera view manually:

![Unreal Editor lit mode](/images/ue_lit.png "Unreal Editor lit mode")

One minor flaw is simply a rendering artifact, that cutouts appears to be sorted from back to front, leading the stem of the sunflower to be rendered in front of the sunflower's flower. Also, the cutouts do not cast shadows in this mode (they used to cast shadows for the whole billboard in 4.27, so this is an improvement, IMO).

In the upper left area of the viewport is a "Lit" setting for the View Mode. Clicking and changing this to "Path Tracing" gives the following:

![Unreal Editor path tracing mode](/images/ue_path_trace.png "Unreal Editor path tracing mode")

There is some auto-exposure system in place that I have not figured out how to turn off. The lava is actually emitting light in path tracing mode, which can be seen by turning off the other lights in the scene:

![Unreal Editor lava path tracing](/images/ue_lava.png "Unreal Editor lava path tracing")

No intensity adjustments were needed for the lava emission values or scaling (I can't say whether texture scaling is actually used; I did not see it in the user interface for the lava materials). Auto-exposure is occurring, as the lava itself is brighter than above.

Overall USD import has improved considerably in UE 5.0. The main problem I see with all these renders is with the semitransparent purple stained glass block. In Unreal Editor 4.27.2, for example, the path trace of this block looked like this:

![Unreal Editor 4.27.2 path tracing mode](/images/ue_4_27_path_tracing.png "Unreal Editor 4.27.2 path tracing mode")



## Cinema 4D

[Maxon's Cinema 4D](https://www.maxon.net/en/cinema-4d) has a 14-day trial. Loading McUsd.usda is simple: drag and drop the file into the viewport. In the "USD Import Settings" change the "Lights" setting to "Legacy Intensity" (the default is "Nit", a physical unit, but this does not work properly with the Sun in McUsd). Once loaded, you can then pick the camera imported by clicking in the top middle of the viewport where it says "Default Camera" and choose "Camera". Doing so with C4D version R26.107, you get this:

![Cinema 4D](/images/c4d.png "Cinema 4D")

The camera is properly translated. There are some texture mapping problems, but these are more easily seen in the next render.

By default the Sun does not cast a shadow. Open up _Simple_Material_Test in the Objects hierarchy control in the upper right, then select the Sun, then under General at the bottom change Shadow to Area. Press Control-R to perform Render View:

![Cinema 4D render](/images/c4d_render.png "Cinema 4D render")

The diamond block has a color texture mapping problem, with the faces of each block showing a thin gold band a quarter of the way across the face - compare to other renderings here. There are normal mapping mismatches with the prismarine (wrong scale), and with the piston and chiseled quartz block (normals flipped upside down).

On further investigation, the DomeLight in the scene washes out the effect of normal maps in the interactive renderer. I did not find a control to turn down its effect. You can delete the DomeLight, or toggle off its effect by clicking on the two dots, turning them red, in the Objects hierarchy menu. The interactive render is then:

![Cinema 4D no domelight](/images/c4d_no_dome.png "Cinema 4D no domelight")

The Render View render doesn't look as good - the DomeLight helps there:

![Cinema 4D no domelight render](/images/c4d_no_dome_render.png "Cinema 4D no domelight render")

This render more strongly highlights the normal texture mismatches, especially on the prismarine.

## TODO

Some of the many viewers, alphabetically:
* 3DS MAX
* [Activision](https://github.com/Activision/USDShellExtension)
* [Autodesk open-source web-based viewer](https://autodesk-forks.github.io/USD/) - [background info](https://www.keanw.com/2022/02/autodesk-open-sources-web-based-usd-viewing-implementation.html). My attempts to build this repo on Windows have failed.
* Houdini
* Maya
* [Unity](https://docs.unity3d.com/2020.1/Documentation/Manual/com.unity.formats.usd.html)
* [Unreal Editor](https://docs.unrealengine.com/4.26/en-US/WorkingWithContent/USDinUE4): [Omniverse connector](https://docs.omniverse.nvidia.com/con_connect/con_connect/ue4.html)

---
## License

**[CC-NC-BY-SA](LICENSE)**

Textures from the [JG-RTX resource pack](https://github.com/jasonjgardner/jg-rtx), which has the same license.

---
# Contact
Email [me](http://erichaines.com) at [erich@acm.org](mailto:erich@acm.org).