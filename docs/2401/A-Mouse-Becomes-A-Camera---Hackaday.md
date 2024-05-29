<!--yml
category: 未分类
date: 2024-05-27 15:22:26
-->

# A Mouse Becomes A Camera | Hackaday

> 来源：[https://hackaday.com/2024/01/29/a-mouse-becomes-a-camera/](https://hackaday.com/2024/01/29/a-mouse-becomes-a-camera/)

If your pointing device is a mouse, turn it over. The chances are you’ll see a red LED light if you’re not seriously old-school and your mouse has a ball, this light serves as the illumination for a very simple camera sensor. The mouse electronics do their thing by looking for movement in the resulting image, but it should be possible to pull out the data and repurpose the sensor as a digital camera. [Doctor Volt] has a new video showing just that [with the innards of a Logitech peripheral](https://www.youtube.com/watch?v=qAlpt_XYkXI).

The mouse contains a microcontroller and the camera part, which fortunately has an SPI interface. The correct register to query the sensor information was deduced, and as if my magic, an image appeared. An M12 lens provided focus with a handy 3D printed mount, and the board went back into the mouse case as a housing. The pictures have something of the Game Boy camera about them, being low-res and monochrome, but it’s still a neat hack.

If you’d like to give it a go [you can find the code in a GitHub repository](https://github.com/michalin/mousecam). [You might find it worth finding a gaming mouse though](https://hackaday.com/2022/05/16/gaming-mouse-becomes-digital-camera/), for the much higher resolution sensor.

[https://www.youtube.com/embed/qAlpt_XYkXI?feature=oembed](https://www.youtube.com/embed/qAlpt_XYkXI?feature=oembed)

VIDEO