<!--yml
category: 未分类
date: 2024-05-29 12:44:22
-->

# Introducing Babylon.js 7.0\. Our mission is to build one of the most… | by Babylon.js | Medium

> 来源：[https://babylonjs.medium.com/introducing-babylon-js-7-0-a141cd7ede0d](https://babylonjs.medium.com/introducing-babylon-js-7-0-a141cd7ede0d)

# Introducing Babylon.js 7.0

Our mission is to build one of the most powerful, beautiful, simple, and open web rendering engines in the world, and we are excited to announce that mission takes another step forward today, with Babylon.js 7.0.

Babylon.js 7.0 is a [celebration](https://aka.ms/bayblon7Celebration) of a year’s worth of new features, optimizations, and performance improvements that unlock new capabilities for web developers everywhere.

So let’s dive in and see what new goodness awaits you.

# Procedural Geometry (Node Geometry)

We are absolutely thrilled to announce that Babylon.js now features procedural geometry. We call it “Node Geometry.” An advanced system that allows you to create procedural geometry using a non-destructive node tree system.

If you’re already familiar with Babylon’s [Node Material](https://doc.babylonjs.com/features/featuresDeepDive/materials/node_material) and the [Node Material Editor (NME)](https://nme.babylonjs.com/), then you’ll feel right at home in the [Node Geometry Editor](https://nge.babylonjs.com/). This simple to use tool lets you assemble a non-destructive geometry tree, allowing you to create anything from tiny geometric variations, to entire procedural worlds/landscapes.

Procedural geometry offers the ability to create complex geometry at run/build time. This means that the end-user doesn’t have to download large 3D assets. Instead, the local machines/devices can use the CPU to create those assets. This means rather than requiring your web visitors to download hundreds of MBs worth of 3D assets, they can instead download a few KBs worth of Node Geometry data and allow their own machine to create the geometry. This adds a tremendous layer of flexibility for creators to offer the perfect loading/performance optimization for their unique web experience!

There’s really no better way to experience all that Node Geometry will offer other than for you to dive in and try it out yourself!

Try it out yourself: [Node Geometry Editor](https://nge.babylonjs.com/)

Check out a demo: [https://aka.ms/babylon7NgeDemo](https://aka.ms/babylon7NgeDemo)

Learn more: [https://aka.ms/babylon7NgeDoc](https://aka.ms/babylon7NgeDoc)

# Global Illumination

Empowering web creators to conjure up more beautiful and immersive experiences is a foundational part of the Babylon Mission. With Babylon.js 7.0, we are excited to introduce support for basic Global Illumination. This highly desired and advanced feature allows Babylon.js scenes to render even more lifelike experiences by allowing light and shadows to “bounce” around environments in a way that much closer matches reality.

Global Illumination represents a major advancement in web rendering and, just like everything that comes with Babylon.js 7.0, is completely free and open source. To truly appreciate the beauty this adds to scenes, you just need to check it out.

Global Illumination demo: [https://aka.ms/babylon7GIDemo](https://aka.ms/babylon7GIDemo)

Learn more here: [https://aka.ms/babylon7GIDoc](https://aka.ms/babylon7GIDoc)

# Gaussian Splat Rendering

[Gaussian Splatting](https://en.wikipedia.org/wiki/Gaussian_splatting) is a new technique to capture and display volumetric data using [Neural Radiance Fields](https://en.wikipedia.org/wiki/Neural_radiance_field), point clouds and billboards. In more simple terms, it’s an advanced way of allowing people to capture and display the real world with a level of unmatched visual fidelity and performance.

Babylon.js 7.0 adds support for rendering Gaussian Splats on the web, across all devices, running at 60 fps!

Try it out: [https://aka.ms/babylon7GSplatDemo](https://aka.ms/babylon7GSplatDemo)

Learn more here: [https://aka.ms/babylon7GSplatDoc](https://aka.ms/babylon7GSplatDoc)

# Ragdoll Physics

Babylon.js 7.0 builds on the momentum (see what we did there?) of Havok physics support in Babylon, by adding support for ragdoll animation. This allows any skeletal rigged asset to collapse and flop around with limp lifelessness at the push of a button. What are you waiting for? Try it out!

Try it here: [https://aka.ms/babylon7RagdollDemo](https://aka.ms/babylon7RagdollDemo)

Learn more here: [https://aka.ms/babylon7RagdollDoc](https://aka.ms/babylon7RagdollDoc)

# State of Art WebXR Support

Babylon.js continues to support the full WebXR spec, making it as simple as possible to create incredibly immersive web experiences. With this release, Babylon.js 7.0 adds support for several new WebXR features including: full-screen GUI, touchable UI elements, world scale, antialiased multiviews, and the ability to use hands and controllers at the same time! If you are excited about creating immersive experiences on the web, then this version has your name on it!

Check it out: [https://aka.ms/babylon7WebXRDemo](https://aka.ms/babylon7WebXRDemo)

Learn more here: [https://aka.ms/babylon7WebXRLayers](https://aka.ms/babylon7WebXRLayers), [https://aka.ms/babylon7WebXRHC](https://aka.ms/babylon7WebXRHC)

# Apple Vision Pro Support

Apple’s Vision Pro is an exciting product that provides a blending of the real and virtual worlds in fascinating new ways. We are excited to share the Babylon.js 7.0 adds full support for the Vision Pro allowing Apple fans to experience immersive worlds through the web at the push of a button.

If you have an Apple Vision Pro, dive into a Babylon.js world here: [https://aka.ms/babylon7VPDemo](https://aka.ms/babylon7VPDemo)

# Advanced Animation System Updates

The world of computer animation is always evolving and Babylon.js evolves hand in hand with it. Babylon.js 7.0 brings some exciting new features to the underlying animation engine, unlocking powerful new capabilities for real-time animations on the web. These updates add the ability to blend animation groups and mask specific portions of animations allowing creators to fine tune their experiences like never before. Interested in a forward walk cycle, blended with a sideways strafe, all with active morph target lip sync? It’s now all possible thanks to the exciting updates to the animation system.

Learn more: [https://aka.ms/babylon7AnimGroupDoc](https://aka.ms/babylon7AnimGroupDoc), [https://aka.ms/babylon7AnimMaskDoc](https://aka.ms/babylon7AnimMaskDoc)

# State of the Art glTF support

Every version of Babylon.js comes with updated support for the full glTF spec. This means you can always count on the Babylon platform to render the very state-of-the-art and cutting edge rendering advancements on the web. With Babylon.js 7.0, that rich tradition continues with the added support of the Dispersion and Anisotropy glTF extensions.

KHR_materials_dispersion demo: [https://aka.ms/babylon7Dispersion](https://aka.ms/babylon7Dispersion)

KHR_materials_anisotropy demo: [https://aka.ms/babylon7Anisotropy](https://aka.ms/babylon7Anisotropy)

# One Incredible Community

Babylon.js is powered and supported by one of the most incredible open source communities in the world. Over 500 people from across the globe have contributed to making it one of the most powerful, beautiful, simple, and open web rendering engines out there. With Babylon.js 7.0, this community continues to shine bright, by adding some incredible new features and extensions to this passionately shared and nurtured platform.

# The Greased Line

Built right into the core engine, the new Greased Line system unlocks some exciting new possibilities for web creators. This new special type of line is built using the mesh system to display lines of any specified width. The extra sprinkling of non-trademarked fantasy dust is that these lines are equipped with a special shader that allows the line to always face the camera so they can be viewed consistently no matter where the camera moves to.

See it in action: [https://aka.ms/babylon7GLDemo](https://aka.ms/babylon7GLDemo)

Learn more: [https://aka.ms/babylon7GLDoc](https://aka.ms/babylon7GLDoc)

# Advanced Ground Projection

This one is honestly just too cool to put into words, but we’ll do our best. Imagine taking a 360 skybox/environment and then magically transforming the lower half into a “fake” ground that seems to support the 3D objects in your scene. This illusion provides a perfectly smooth transition from the ground to the sky within your scene. Sound like wizardry? Well what are you waiting for? Check it out!

See it in action: [https://aka.ms/babylon7GProjDemo](https://aka.ms/babylon7GProjDemo)

Learn more: [https://aka.ms/babylon7GProjDoc](https://aka.ms/babylon7GProjDoc)

# Seamless Texture Decals

Introduced with Babylon.js 6.0, texture decals offered a powerful new option of projecting images onto an existing texture of a 3D object in your scene. With Babylon.js 7.0, that same system is given superpowers by allowing the decals to seamlessly map across UV boundaries! That means that no matter how pretty or less-pretty your UV layout might be, decals will work perfectly on any 3D object!

Check it out: [https://aka.ms/babylon7SeamTsDemo](https://aka.ms/babylon7SeamTsDemo)

Learn more: [https://aka.ms/babylon7SeamTsDoc](https://aka.ms/babylon7SeamTsDoc)

# MMD Support Community Extension

This exciting extension to Babylon.js 7.0 offers creators the ability to import 3D assets and animations from the popular 3D Creation Software: [MikuMikuDance](https://mikumikudance.en.softonic.com/) or MMD for short. In addition to loading assets and animations, it also has support for IK solvers, a morph system, synced audio playback, player controls, and much much more.

Check it out: [https://aka.ms/babylon7MMDDemo](https://aka.ms/babylon7MMDDemo)

Learn more here: [https://aka.ms/babylon7MMDDoc](https://aka.ms/babylon7MMDDoc)

# Thank You

With each evolution of Babylon.js comes a revolution in web rendering technology and an overwhelming feeling of gratitude. The Babylon platform simply wouldn’t be possible without the incredible community of developers, the 500+ contributors, and the steadfast advocates that contribute their knowledge, expertise, help, and passion to this amazing technology. “Thank you” to each one of you for all that you do to help make Babylon.js one of the most powerful, beautiful, simple, and open web rendering platforms in the world.