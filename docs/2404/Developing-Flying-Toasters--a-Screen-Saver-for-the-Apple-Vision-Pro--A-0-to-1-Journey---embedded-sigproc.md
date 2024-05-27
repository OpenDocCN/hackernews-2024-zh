<!--yml
category: 未分类
date: 2024-05-27 12:50:17
-->

# Developing Flying Toasters, a Screen Saver for the Apple Vision Pro: A 0 to 1 Journey · embedded sigproc

> 来源：[https://abhipray.com/posts/flying_toasters/](https://abhipray.com/posts/flying_toasters/)

I recently decided to make an [app for the Apple Vision Pro (AVP)](http://apps.apple.com/us/app/flying-toasters/id6479964879) during a two-week break between jobs. I wanted to explore its potential and learn some new skills. My day job primarily involves low-level firmware; I hadn’t built a user-facing app in a while–my last iOS app experience was back in high school in 2011! This was a chance to see how app development has evolved in 2024, especially with modern code generation tools.

## Inspiration and the “Screensaver” Idea [*Link to heading*](#inspiration-and-the-screensaver-idea)

*After brainstorming with friends, one suggested I look into the classic “Flying Toasters” screensaver by After Dark (released in 1989). I watched a video of it and thought it would be a perfect starter project!*

 *[https://www.youtube.com/embed/Ft5DIBAvXIU](https://www.youtube.com/embed/Ft5DIBAvXIU)

VIDEO*

*I was especially excited to bring the concept of a “screensaver” to this new platform. Historically, screensavers served a practical purpose on CRT monitors—preventing screen burn-ins. From Wikipedia:*

> *Screen burn-in, image burn-in, ghost image, or shadow image, is a permanent discoloration of areas on an electronic display such as a cathode ray tube (CRT) in an old computer monitor or television set. It is caused by cumulative non-uniform use of the screen. One way to combat screen burn-in was the use of screensavers, which would move an image around to ensure that no one area of the screen remained illuminated for too long.*

*Today, screensavers primarily serve aesthetic and entertainment purposes, activating during periods of user inactivity. My aim was to incorporate a similar, inactivity-based trigger for the AVP screensaver. Initially, my plan was to employ gaze tracking to identify moments when a user might be “zoned out”. Due to privacy considerations, Apple restricts access to such sensor data. As a workaround, users can signal their activity by periodically tapping a button within the app’s main control window, effectively resetting the inactivity timer. The app also allows users the flexibility to customize the timeout duration, transforming the screensaver into a gentle nudge to take breaks from using the AVP.*

*The screensaver features flying toasters soaring across your real-world environment. They emerge from a portal showing the sun and disappear into a portal showing the moon. It was fun adding silly little features like:*

*   *Gesture controls: Scale and rotate the portals to your liking.*
*   *Toaster interaction: Tap a toaster to trigger a whimsical phrase. Baby toasters occasionally make an appearance too!*
*   *Customization: Adjust toaster count, toastiness level, colors, and music. Or, activate “ghost mode” to prevent collisions if the on-screen chaos gets overwhelming.*

*Check out the app previews and screenshots on the app store to get a visual: [https://apps.apple.com/us/app/flying-toasters/id6479964879](https://apps.apple.com/us/app/flying-toasters/id6479964879)*

*Here are some specific challenges I overcame during the development process:*

*   *Learning Swift: Although I’d used Objective-C for an iOS app back in high school, I wanted to learn Apple’s recommended way of writing apps. I spent time reading [Swift tutorials](https://carlosicaza.com/swiftbooks/SwiftLanguage.pdf) and practicing in [Swift Playgrounds](https://developer.apple.com/swift-playgrounds/). ChatGPT and Gemini were also great resources for code editing and debugging.*

*   *3D Animation/Art: I had no prior experience with 3D animation. I started with a 3D-printable toaster model and followed [tutorials](https://www.youtube.com/watch?v=VuMu4tAzFjw) to animate the wings. Creating those toasts was trickier! I couldn’t find the right textures for different toastiness levels – after a lot of trial and error (and a hundred attempts with Photoshop’s generative fill), I finally got those realistic-looking textures.*

*   *Portal Gestures: Apple’s [example code](https://developer.apple.com/documentation/realitykit/transforming-realitykit-entities-with-gestures?changes=_8) was key for implementing intuitive scaling and rotation gestures for the portals.*

*   *Toaster Physics vs. Animation: I found out the hard way that RealityKit’s physics engine and strict animation paths (using FromToByAnimation()) don’t always play nicely together. My chaotic collision solution? Random launch points with a minimum distance constraint, random [cubic-bezier](https://cubic-bezier.com) curve paths, and plenty of linear/angular damping on those toasters. If the toasters’ movements are still subjectively chaotic, the user can opt out of collisions by turning on ghost mode.*

*   *Head Tracking: I wanted the toasters’ whimsical phrases to appear in speech bubbles that always faced the user. RealityKit doesn’t directly track head position, but I adapted a clever [ARKit workaround](https://stackoverflow.com/questions/77577395/how-to-know-users-position-in-surrounding-space-in-visionos/77616297#77616297) to achieve this.*

*This project was a blast! If you have an AVP and end up seeing the flying toasters in action, please let me know what you think! Just as the original Flying Toasters screensaver evolved over a decade (see [the YouTube playlist here](https://www.youtube.com/watch?v=Ft5DIBAvXIU&list=PLxRwNKfqQOI1_pp6Cp4p1MnpCsoqCx2cK&index=2)), I anticipate if there is enough interest and feedback, I will improve the experience. So do share feedback!*