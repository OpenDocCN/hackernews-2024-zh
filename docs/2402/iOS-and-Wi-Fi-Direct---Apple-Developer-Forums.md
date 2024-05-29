<!--yml
category: 未分类
date: 2024-05-29 13:20:08
-->

# iOS and Wi-Fi Direct | Apple Developer Forums

> 来源：[https://forums.developer.apple.com/forums/thread/12885](https://forums.developer.apple.com/forums/thread/12885)

> How does it do this when the two devices need not be on the same network?

Via an Apple-specific (not Wi-Fi Direct) peer-to-peer Wi-Fi protocol.

> It seems the WiTap sample code requires all instances to be on the same network already, is that correct?

No. Although don’t take my word for it, try it yourself:

1.  grab two iOS devices

    **Note** the only requirement is that they each must be new enough to have a Lightning connector. Peer-to-peer Wi-Fi support is not *tied* to the Lightning connector, it's just a helpful coincidence that the hardware that supports peer-to-peer Wi-Fi also happens to have a Lightning connector and thus it's an easy way to identify that support.

2.  on each, forget the infrastructure Wi-Fi network

3.  on each, disable Bluetooth (which implements a different form of Apple-specific, Bonjour + TCP/IP peer-to-peer networking)

4.  on each, run WiTap and start a ‘game’

> This is the problem we're trying to solve: there's no infrastructure Wi-Fi network. Our peripheral can host a network, but we'd like to avoid making the user leave our app to go to Settings to connect to our peripheral.

Ah, this is the first mention of non-Apple hardware in this thread. The situation with non-Apple hardware is more challenging because the peer-to-peer networking protocols (both Wi-Fi and Bluetooth) are not documented for third-party use.

Your options:

In a typical home accessory setup you want the accessory to join the same infrastructure Wi-Fi as all the other devices in the home. Apple explicitly supports this via the Wireless Accessory Configuration (WAC) mechanism. This lets an iOS device pass its Wi-Fi configuration to the accessory so that the accessory can join the same Wi-Fi network as the iOS device is currently on.

You can learn more about this in WWDC 2014 Session 701 "Designing Accessories for iOS and OS X".

[https://developer.apple.com/videos/](https://developer.apple.com/videos/)

WAC is part of [MFi](https://developer.apple.com/programs/mfi/).

*   HomeKit accessories, also created under the aegis of MFi, have their own set of capabilities. Let me know if you’re interested in that angle.

*   Most other ad hoc and peer-to-peer Wi-Fi scenarios do not have good solutions. You can generally make things work, but it tends to be a manual process.

Share and Enjoy
—
Quinn "The Eskimo!"
Apple Developer Relations, Developer Technical Support, Core OS/Hardware

```
let myEmail = "eskimo" + "1" + "@apple.com"
```