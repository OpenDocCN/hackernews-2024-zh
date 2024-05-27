<!--yml
category: 未分类
date: 2024-05-27 13:14:42
-->

# usbredir

> 来源：[https://www.spice-space.org/usbredir.html](https://www.spice-space.org/usbredir.html)

# usbredir

usbredir is the name of a network protocol for sending USB device traffic over a network connection. It is also the name of the software package offering a parsing library, a usbredirhost library and several utilities implementing this protocol.

The protocol is documented [here](https://gitlab.freedesktop.org/spice/usbredir/-/blob/main/docs/usb-redirection-protocol.md), this document also explains what is meant with a usbhost and a usbguest.

usbredir was created for use with Spice, which is why it is hosted on spice-space.org, but the protocol and the usbredirhost are completely independent of spice, they could for example also be used to create a vnc extension for redirecting USB devices over a vnc connection to a qemu virtual machine.

The usbguest side is currently only implemented for qemu, and shipped as part of qemu (enabling this in qemu requires the libusbredirparser library to be available when building qemu).

usbredir supports USB device filtering configurable by a filter string

## Host Configuration

For redirection to work, the virtual machine must have a supported USB controller. Qemu supports both USB2 and USB3, but at the time of writing, USB3 has had less testing. For USB2 support, the guest must have a EHCI controller and companion UHCI controller (companion UHCI is needed in order to support also USB1.x devices). For USB3 support, an XHCI controller is required. It also needs to have Spice channels for USB redirection. The number of such channels determine the number of USB devices that it's possible to redirect at the same time.

More information about USB controllers in qemu could be found [here](http://git.qemu-project.org/?p=qemu.git;a=blob_plain;f=docs/usb2.txt;hb=HEAD).

### Using virt-manager

Virtual machines created with virt-manager should have a USB controller by default. In the virtual machine details, select "Controller USB" in the left pane. If you only need to support USB2 devices, make sure its model is set to "USB2". For USB 3.0 support, select "USB3" for the model type.

You can then click on "Add Hardware" and add "USB Redirection" items as the number of USB devices you want to be able to redirect simultaneously.

### Using libvirt

The following libvirt XML will configure a guest with USB2 support and the ability to redirect 3 devices simultaneously:

```
<controller  type='usb'  index='0'  model='ich9-ehci1'/>
<controller  type='usb'  index='0'  model='ich9-uhci1'>
  <master  startport='0'/>
</controller>
<controller  type='usb'  index='0'  model='ich9-uhci2'>
  <master  startport='2'/>
</controller>
<controller  type='usb'  index='0'  model='ich9-uhci3'>
  <master  startport='4'/>
</controller>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/> 
```

For USB3 support, the configuration can be simplified to:

```
<controller  type='usb'  index='0'  model='nec-xhci'/>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/> 
```

### Using QEMU

The following qemu options will configure a guest with USB2 support and the ability to redirect 3 devices simultaneously

```
-device  ich9-usb-ehci1,id=usb  \
-device  ich9-usb-uhci1,masterbus=usb.0,firstport=0,multifunction=on  \
-device  ich9-usb-uhci2,masterbus=usb.0,firstport=2  \
-device  ich9-usb-uhci3,masterbus=usb.0,firstport=4  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev1  \
-device  usb-redir,chardev=usbredirchardev1,id=usbredirdev1  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev2  \
-device  usb-redir,chardev=usbredirchardev2,id=usbredirdev2  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev3  \
-device  usb-redir,chardev=usbredirchardev3,id=usbredirdev3 
```

For USB3 support, the configuration can be simplified to:

```
-device  nec-usb-xhci,id=usb  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev1  \
-device  usb-redir,chardev=usbredirchardev1,id=usbredirdev1  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev2  \
-device  usb-redir,chardev=usbredirchardev2,id=usbredirdev2  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev3  \
-device  usb-redir,chardev=usbredirchardev3,id=usbredirdev3 
```

## Client Configuration

The client needs to have support for USB redirection. In remote-viewer, you can select which USB devices to redirect under the "File->USB device selection" menu once the Spice connection is established. There are also various command line redirection options which are described below and when running remote-viewer with --help-spice.

To get USB redirection working on Windows clients, you need to install [UsbDk](/download/windows/usbdk/)

## Filter String Format

A filter is used to set blocking or autoconnect rules for USB devices. It consists of one or more rules, where each rule has the form of:

**<class>,<vendor>,<product>,<version>,<allow>**

Use -1 for class/vendor/product/version to accept any value.

And the rules themselves are concatenated like this:

**rule1|rule2|rule3**

A client's default auto-connect filter string is 0x03,-1,-1,-1,0|-1,-1,-1,-1,1

This filters out HID (class 0x03) USB devices from auto connect and auto connects anything else. Note the explicit allow rule at the end, this is necessary since by default all devices without a matching filter rule will not auto-connect.

## Client Filtering

Set a string specifying a filter to determine which USB devices, when plugged in, are allowed/blocked to **auto-redirect** USB traffic to the guest (client's window has to be in focus).

```
remote-viewer  --spice-usbredir-auto-redirect-filter="0x03,-1,-1,-1,0|-1,-1,-1,-1,1" 
```

Set a string specifying a filter to determine which USB devices, that are already plugged in, to **redirect on connect** once Spice connection is established.

```
remote-viewer  --spice-usbredir-redirect-on-connect="0x03,-1,-1,-1,0|-1,-1,-1,-1,1" 
```

## Host Filtering

Set a string specifying a filter to determine which USB devices are **allowed/blocked** to redirect USB traffic to the guest.

### Using QEMU

```
-device  usb-redir,filter='0x03:-1:-1:-1:0|-1:-1:-1:-1:1',chardev=usbredirchardev1,id=usbredirdev1 
```

Note that in a QEMU command, the filter string should use a **':'** character as a separator within the rule.

### Using libvirt

```
...
<devices>
  ...
  <redirfilter>
  <usbdev  class='0x08'  vendor='0x1234'  product='0xbeef'  version='2.56'  allow='yes'/>
  <usbdev  allow='no'/>
  </redirfilter>
</devices>
... 
```

## Download

Latest version: [usbredir-0.14.0.tar.xz](/download/usbredir/usbredir-0.14.0.tar.xz)

Older versions: [0.13](/download/usbredir/usbredir-0.13.0.tar.xz), [0.12](/download/usbredir/usbredir-0.12.0.tar.xz), [0.11](/download/usbredir/usbredir-0.11.0.tar.xz), [0.10](/download/usbredir/usbredir-0.10.0.tar.xz), [0.9](/download/usbredir/usbredir-0.9.0.tar.xz), [0.8](/download/usbredir/usbredir-0.8.0.tar.bz2), [0.7](/download/usbredir/usbredir-0.7.tar.bz2), [0.6](/download/usbredir/usbredir-0.6.tar.bz2), [0.5.2](/download/usbredir/usbredir-0.5.2.tar.bz2), [0.5.1](/download/usbredir/usbredir-0.5.1.tar.bz2), [0.5](/download/usbredir/usbredir-0.5.tar.bz2), [0.4.3](/download/usbredir/usbredir-0.4.3.tar.bz2), [0.4.2](/download/usbredir/usbredir-0.4.2.tar.bz2), [0.4.1](/download/usbredir/usbredir-0.4.1.tar.bz2), [0.4](/download/usbredir/usbredir-0.4.tar.bz2), [0.3.3](/download/usbredir/usbredir-0.3.3.tar.bz2), [0.3.2](/download/usbredir/usbredir-0.3.2.tar.bz2), [0.3.1](/download/usbredir/usbredir-0.3.1.tar.bz2), [0.3](/download/usbredir/usbredir-0.3.tar.bz2)

### Source Code

You can find the official usbredir git repository [here](https://gitlab.freedesktop.org/spice/usbredir).