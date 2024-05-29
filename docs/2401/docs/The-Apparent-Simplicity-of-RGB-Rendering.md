<!--yml
category: 未分类
date: 2024-05-27 15:24:29
-->

# The Apparent Simplicity of RGB Rendering

> 来源：[https://thomasmansencal.substack.com/p/the-apparent-simplicity-of-rgb-rendering](https://thomasmansencal.substack.com/p/the-apparent-simplicity-of-rgb-rendering)

The primary goal of physically-based rendering (PBR) is to create a simulation that accurately reproduces the imaging process of electro-magnetic spectrum radiation incident to an observer. This simulation should be indistinguishable from reality for a similar observer.

Great care was taken to not use the words *human* and *colour* (or even *light)* in the previous description: Indeed, often and especially in the Visual Effects industry, the observer is a motion picture or stills camera. A typical camera is not colorimetric, i.e., its sensitivities are not a linear combination of the human observer cone fundamentals.

Nikon 5100 sensitivities compared to the human visual system cone fundamentals for the 2°.

Because a camera is not sensitive to incident light the same way than a human observer, the images it captures are transformed to be colorimetric. A project might require infrared imaging simulation, a portion of the electro-magnetic spectrum that is invisible to us. Radically different observers might image the same scene but the act of observing does not change the intrinsic properties of the objects being imaged. Consequently, the physical modelling of the virtual scene should be independent of the observer.

The physical modelling of the virtual scene should be independent of the observer.

When RGB textures are used to describe the physical properties of objects in the virtual scene, it becomes correlated to the observer because the textures encoding RGB colourspace is derived from the observer sensitivities. Thus, changing the encoding RGB colourspace modifies the virtual scene.

This image illustrates the effect of multiplying various colours by themselves into different RGB colourspaces: the resulting colours are different. The various samples are generated as follows: 3 random sRGB colourspace values are picked and converted to the three studied RGB colourspaces, they are exponentiated, converted back to sRGB colourspace, plotted in the CIE 1931 Chromaticity Diagram on the left and displayed as swatches on the right.

In general terms, using photometric quantities to describe surface characteristics induces observer correlation because they are weighted by the human spectral sensitivities.

Renders of the same scene using ITU-R BT.709 primaries (first row), 47 spectral bins (second row), ITU-R BT.2020 primaries (third row), spectral minus BT.709 primaries renders residuals (fourth row), spectral minus BT.2020 primaries renders residuals (fifth row). The last row showcases composite images assembled with three vertical stripes of respectively the BT.709 primaries, spectral and, BT.2020 primaries renders. Direct illumination tends to match between the renders. Areas that show the effect of multiple light bounces, i.e., the ceiling, in the BT.709 and BT.2020 primaries renders tend to exhibit increased saturation, especially in the BT.709 primaries render or slight loss of energy, especially in the BT.2020 render. Excluding outliers, e.g., the visible light source, the RMSE with the spectral render are **0.0083** and **0.0116** for respectively the BT.2020 primaries and BT.709 primaries renders.

The ideal physically-based rendering engine would use radiometric quantities as input, e.g., spectral reflectance, transmittance or absorptance, and spectral irradiance. We have, unfortunately, engineered a mountain of non-trivial issues by adopting RGB as the model for most rendering engine to operate with.

The “apparent” simplicity of RGB rendering, its speed and the unfathomable quantity of input material drive cost effectiveness. However, behind the seducing power of RGB rendering lurks an unconquerable wall preventing accurate modelling of the real world. When a photograph is used as a texture in a rendering engine, the scene illuminant is always “baked” into the texture. Each of the texture’s RGB pixel takes its root in the integration of the result of the multiplication of the spectral bidirectional scattering distribution function (BSDF) of the objects in the scene with the environment irradiance, the lens transmittance, and, the camera spectral sensitivities:

\(R=\int_{\lambda}\beta(\lambda)S(\lambda)T(\lambda)r(\lambda)d\lambda\ \ \ \ \ G=\int_{\lambda}\beta(\lambda)S(\lambda)T(\lambda)g(\lambda)d\lambda\ \ \ \ \ B=\int_{\lambda}\beta(\lambda)S(\lambda)b(\lambda)d\lambda\ \ \ \ \ (1) \)

where **β** is the spectral BSDF the surface of interest, **S** is the environment irradiance, **T** is the lens transmittance and **r**, **g**, and **b** are the camera red, green and blue spectral sensitivities, respectively.

The human observer follows a similar process:

Integration of the spectral reflectance distribution of a sample of sand for X tristimulus value. Note that this process must be repeated for Y― and Z― shown in dashed green and blue lines to obtain the corresponding Y and Z tristimulus values.

Computer graphics artists author “albedo” textures to define the “colour” of objects in the virtual scene. Albedo, in the *context of the Visual Effects industry*, could be defined as follows:

Albedo is the measure of the reflectivity of a surface or object. It is the ratio of the reflected light from an object to the incident light upon it.

Albedo is typically expressed as a percentage or a decimal value between 0 and 1, where 0 represents complete absorption of light (no reflection) and 1 represents complete reflection (no absorption). A high albedo means that an object reflects a large portion of the incident light, whereas a low albedo indicates that most of the light is absorbed.

This is not a standard definition, and in scientific research, albedo is often a measure of the fraction of solar radiation reflected by an object. [ISO 9488:2022](https://www.iso.org/obp/ui/#iso:std:iso:9488:ed-2:v1:en) defines it as follows:

**albedo**

ratio of the *[solar radiation (3.2.13)](https://www.iso.org/obp/ui/#iso:std:iso:9488:ed-2:v1:en:term:3.2.13)* (radiant or luminous energy) reflected by a surface, to that incident on it

Note 1 to entry: This is a generalized term for the average *[reflectance (3.4.3)](https://www.iso.org/obp/ui/#iso:std:iso:9488:ed-2:v1:en:term:3.4.3)* of a defined surface area (usually of the earth or clouds); its use is discouraged in technical applications, where the preferred term is "reflectance".

After reaching this point in the article, feeling mildly unsettled at the thought of describing the reflectivity of an object using an albedo texture whose pixels have been subjected to equation (1) is fully expected!

To continue along that path, most materials and shaders used in RGB rendering engines have **.*color** parameters. Colour, as defined by the [CIE](https://cie.co.at/eilvterm/17-22-040), is the “characteristic of visual perception that can be described by attributes of hue, brightness (or lightness) and colourfulness (or saturation or chroma).” Therefore, colour is not an intrinsic physical property of objects but the interpretation our brains make about a specific characteristic of objects. Describing a virtual tomato as red conveys its appearance for a human observer, but it does not model its physical properties, in contrast, the spectral BSDF does, unequivocally.

We certainly ought to use precise terminology and be sensible about the words we use to describe material parameterisation. The [OpenPBR white paper](https://academysoftwarefoundation.github.io/OpenPBR/) would be a good place to define important terms.

Eventually, more questions will emerge, here are a few cherry-picked ones to reflect upon:

*   What illuminant should be used when authoring textures? That of the working colourspace or CIE Illuminant E?

*   Are albedo textures relative to solar radiation?

*   How a blackbody should be converted to RGB for rendering?

*   What does changing the white balance setting of the virtual camera means to the blackbody appearance?

*   What white balance settings should be use to process HDR images for image-based lighting?

In summary, RGB rendering introduces observer correlation and coupling throughout the virtual scenes making it challenging to accurately simulate the real world. Under its shadow, it becomes arduous to cognise about the appearance of virtual objects. Fortunately, there is a glimmer of hope at the end of the tunnel: All these challenges are addressed by the elegance of spectral rendering, which eliminates the aforementioned correlation.

I wrote about [the importance of terminology](https://www.colour-science.org/posts/the-importance-of-terminology-and-srgb-uncertainty/) almost 10 years ago and Mark Fairchild’s quote is still relevant:

Why should it be particularly difficult to agree upon consistent terminology in the field of color appearance? Perhaps the answer lies in the very nature of the subject. Almost everyone knows what color is. After all, they have had firsthand experience of it since shortly after birth. However, very few can precisely describe their color experiences or even precisely define color.