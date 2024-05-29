<!--yml
category: 未分类
date: 2024-05-27 14:34:16
-->

# iPod 'squeaks' betray software secrets | New Scientist

> 来源：[https://www.newscientist.com/article/dn7085-ipod-squeaks-betray-software-secrets/](https://www.newscientist.com/article/dn7085-ipod-squeaks-betray-software-secrets/)

Computer enthusiasts have worked out how to reprogram Apple’s iPod music player with their own code using an ingenious acoustic trick.

They adapted the component that generates clicks – or “squeaks” – as a user scrolls through the on-screen menu in order to extract vital information from the latest generation of the device. This allowed them to install an alternative operating system and make their iPods run games and other new programs.

The project began when Nils Schneider, a 17-year-old computer science student from Germany, received an iPod for Christmas. Unlike most new iPod owners, he decided to install Linux – a freely available computer operating system not used as standard in iPods – on his device.

The existing version of Linux for the iPod would not install easily, however, as the latest generation of player features new hardware. Undeterred, Schneider decided to figure out how these components worked by himself. He found he could control some parts of the device but not those containing details about the way the unit starts up, which is vital to getting Linux installed.

## Piezoelectric component

Instead of going through the usual process of trial and error to work out the code, Schneider realised that listening to it could provide a shortcut. Bernard Leach, a UK software engineer who helped set up the so-called iPod Linux project, had already worked out how to control the piezoelectric component within the iPod that produces the click.

 To decipher the bootloader code – the program which allows the iPod to start up – Schneider decided to use Leach’s system to play the bootloader code as sound.

“I just tried to encode a single byte as a click sound with different spaces between the clicks,” Schneider told **New Scientist**. “It seemed to work but it was slow so I played around with the code and found out how to make the clicks faster.”

After encoding the bootloader data he recorded the resulting sounds onto another PC programmed to convert it back into computer code. The whole process took more than 20 hours and Schneider had to construct a sound proof box for the recording. But, in the end, he had extracted the information intact.

## Serious purpose

This made it possible to get Linux running on the device, along with a variety of compatible software programs such as simple games and audio recorders.

“After extracting the bootloader it was only a couple of days’ worth of work to get iPod Linux booting,” Leach told **New Scientist**. “Otherwise it would have taken months.”

Leach explains that the iPod Linux project is partly for fun but also has a more serious purpose. “It changes the iPod from a consumer device, where the manufacturer sets the rules about what it will and won’t do, into a general purpose device,” he says. “Much of the interest has been to develop various games, but things like a simple calculator, drawing program and even a GPS/mapping interface are all possible.”