<!--yml
category: 未分类
date: 2024-05-27 14:38:33
-->

# Picking the Widevine Locks: Acquiring and Using an L3 CDM | Mo Ismailzai

> 来源：[https://www.ismailzai.com/blog/picking-the-widevine-locks](https://www.ismailzai.com/blog/picking-the-widevine-locks)

The world of digital rights management (DRM) is a deliberately murky one, relying in part on security through obscurity. This poses a challenge for developers tasked with delivering paid-media, especially because much of the documentation is only delivered through vendor-specific enterprise portals. Nonetheless, DRM and secure content delivery is key to the modern Internet and important to understand. I recently had to dive into [Widevine](https://www.widevine.com/solutions/widevine-drm), probably the most popular DRM platform given that it's native to Chrome and Android devices, but the basics are effectively the same across the major vendors, including Microsoft's [PlayReady](https://www.microsoft.com/playready/) and Apple's [Fairplay](https://developer.apple.com/streaming/fps/).

DRM is the management of legal access to digital content, which is trickier than managing access to physical media. One of the fundamental qualities of digital files is that they can be duplicated with perfect fidelity, such that the copy is indistinguishable from the original. Despite this, moving to a digital world is extremely lucrative because it's significantly cheaper than producing, storing, distributing, and selling physical media.

As far as Widevine is concerned, the overall process is pretty straightforward. The media to be distributed is stored in an encrypted format and is effectively impossible to access without a valid key. When you log into a streaming service like Netflix and attempt to play a video, your browser requests a license from the content's license server. The license server evaluates this request and checks to see if your device is authorized and meets the content’s security requirements. Widevine categorizes devices into three security levels: L1, L2, and L3, based on how securely the cryptographic operations and keys are handled. Most web browsers and some devices only support L3, which means all cryptographic operations are performed in software rather than hardware. L3 is considered the least secure level as the content and keys are processed in the potentially more vulnerable software environment, and for this reason, there are usually limitations on what content can be played on L3 devices, such as not allowing 1080p or 4K playback.

If your request passes the license check, decryption keys are sent to your Widevine DRM client which begins the decryption process. The decryption occurs in real-time by the Widevine client on your device, using the keys obtained from the license server. The media player renders the decrypted content for playback. On L3 devices, the content is not as securely protected during rendering and output, making it more susceptible to piracy compared to L1 devices. Throughout the playback, the Widevine client may continually ensure the validity of the license and adherence to the usage rules. This can include checks like maintaining a secure environment and verifying that the content is not being illegally recorded or transmitted.

In this post, I'll run through a sample Widevine workflow, including how to acquire an L3 Content Decryption Module from an emulated Android device. Unfortunately, this is a trivially simple process highlighting that security through obscurity really is no security at all. Before I dive in, a quick heads-up: this post is all about the nitty-gritty of DRM and is meant to spark a broader conversation about creating *user experiences* that cannot be pirated. ***I'm not endorsing or suggesting you go around breaking DRM protections or laws***. This stuff can land you in hot water legally and this post is meant as a developer resource that peeks into the complex world of DRM. Always play it cool and legal folks.

## Acquire an L3 CDM from an Android device

### Setting Up the Environment

The first step is to establish the necessary environment. I work on Linux and run Manjaro,these days so those are the instructions I'll include, but the overall process is the same for any operating system. We'll need to install [Android Studio](https://developer.android.com/studio), which is freely available from most package managers or as a web download. We also need the `android-tools` and `xz` packages to interface with the emulated Android device and decompress the [Frida Server](https://frida.re/) archive.

```
yay -Syu android-studio android-tools xz
```

Once Android Studio has been successfully installed, launch it, create a new project, then navigate through `Tools > Device Manager` to set up a new Android device emulator. I choose the `Pixel 7 Pro` with API Level `28` and Target `Android 9 (Google APIs)` as this is a known working configuration for generating L3 CDMs. The IDE should begin to download the necessary dependencies; once it's done, start the emulated Android devices and ensure it's recognized by your system using the `adb devices` command in the terminal.

### Working with Frida Server

Frida is a dynamic instrumentation toolkit used by security researchers and reverse engineers who need to hook into black box undocumented features. It allows you to monitor application traffic, for instance, before it's encrypted and sent over the wire, so it can be helpful in places where WireShark falls short. You can install Frida Server directly from their [GitHub release](https://github.com/frida/frida/releases/)[s](https://github.com/frida/frida/releases/) page. The version you choose must correspond to the version of the Widevine `wvdumper` orchestration script downloaded in the next step, so be sure to update both if you deviate from my examples:

```
wget https://github.com/frida/frida/releases/download/16.0.8/frida-server-16.0.8-android-x86_64.xz
unxz frida-server-16.0.8-android-x86_64.xz 
```

Having downloaded and unzipped the release, you'll need to push the file to your emulated android device:

```
 adb push frida-server-16.0.8-android-x86_64 /sdcard
```

Finally, you'll need to shell into the emulated device, elevate privileges, move Frida Server to a suitable directory, ensure it has execution permissions, and then run it to start listening for instrumentation commands.

```
adb push frida-server-16.0.8-android-x86_64 /sdcard
adb shell
su
mv /sdcard/frida-server-16.0.8-android-x86_64 /data/local/tmp
chmod  +x /data/local/tmp/frida-server-16.0.8-android-x86_64
/data/local/tmp/frida-server-16.0.8-android-x86_64
```

### Dumping Keys with Widevine Key Dumper

With Frida Server now running, we need to setup our Python orchestration script to interface with it (obviously, you'll need to keep that terminal open and the process running, so fire up a new terminal).

Begin by creating a dedicated project directory. Within this directory, set up a Python virtual environment and install all the necessary dependencies:

```
using python -m venv ./.
bin/pip3 install frida==16.0.8 frida-tools==12.0.4 protobuf==3.19.0 pycryptodome
```

These versions matter and if you want to use a later version of Frida Server, some trial and error with dependencies may be necessary, unless you write your own orchestration script instead.

With the environment now set up, clone the Widevine `wvdumper/dumper` repository, then run the `dump_keys.py` script:

```
git clone https://github.com/wvdumper/dumper l3keydump
cd l3keydump
../bin/python3 dump_keys.py
```

Ignoring all the helpers, this script really just boils down to these lines:

```
device = frida.get_usb_device()
scanner = Scan(device.name)
for process in device.enumerate_processes():
    logging.debug(process)
    if 'drm' in process.name:
        libraries = scanner.find_widevine_process(device, process.name)
        if libraries:
            for library in libraries:
                scanner.hook_to_process(device, process.name, library) 
```

It hooks into any Widevine DRM processes that the Frida Server can identify and begins to scan for keys. All you need to do now is to launch the Google Chrome browser on the emulated Android device and navigate to a site that triggers the Widevine DRM workflow, for instance, [https://bitmovin.com/demos/drm](https://bitmovin.com/demos/drm).

If everything is configured correctly, this will trigger the script to create a new `./key_dumps` directory and dump your CDM's client ID and private key, `client_id.bin` and `client_id.pem` respectively. For all intents and purposes, you can now act as a legitimate L3 CDM, effectively bypassing the Widevine security model.

## Use the L3 CDM to decrypt Widevine encrypted content

Using the CDM to request and decrypt content from a streaming service depends on how that specific service has implemented their Widevine workflow. Widevine requires the client to identify themselves to the license server and request a license for a specific piece of content. Once identified and authorized, the license server returns the corresponding decryption keys. Everything outside of this is entirely up to the implementing web developers and can vary significantly across services.

Nonetheless, there are only a handful of ways to transmit information via HTTP, be it GET parameters, POST payloads, or HTTP headers, so with a little trial and error, a motivated client can quickly identify what's required and spoof the same request outside of the browser, thus bypassing any built-in browser security. To demonstrate this, we can use another Python script. In the same directory as you created above, clone the decryption codebase and install the relevant dependencies:

```
git clone https://github.com/medvm/widevine_keys.git l3decrypt
bin/pip3 install pycryptodomex xmltodict requests google google-api-python-client
```

The [Widevine keys repository](https://github.com/medvm/widevine_keys) is rough and dated, and the included L3 CDM has long-since been blacklisted, so for the purposes of this demonstration, you will need to replace the CDM in the codebase with the one we acquired from the emulated Android device above:

```
cp l3keydump/key_dumps/*/private_keys/*/*/client_id.bin l3decrypt/cdm/devices/device_client_id_blob
cp l3keydump/key_dumps/*/private_keys/*/*/private_key.pem l3decrypt/cdm/devices/device_private_key
```

Now we need to acquire our encrypted content and their decryption keys. Navigate to the Widevine-protected content you want to watch and identify the relevant Media Presentation Description (MPD) and license server web requests.

The MPD is a manifest file that describes the structure of a media presentation, including details about available bitrates, resolutions, and subtitles. This tells us what content is available to us and where to download it in its encrypted form. Unsurprisingly, streaming platforms want to make this difficult for you to do, but ultimately, it's trivial using tools like [mitmproxy](https://github.com/mitmproxy/mitmproxy), [zaproxy](https://github.com/zaproxy/zaproxy), [httptoolkit](https://github.com/httptoolkit/httptoolkit), or even your web browser's basic network requests tab. In the simplest cases, both MPD and the license requests are simple GET calls to an endpoint. In more complex cases, they are known to require certain headers, payloads, or HTTP verbs, but at the end of the day, these are network calls being made from your browser and are thus completely transparent, so with some trial and error, you can always identify the appropriate requests.

In the case of a straightforward GET call, all you need is the copy the relevant URLs. For requests that require specific headers, you'll need to modify the `headers.py` file in this repository to include the necessary headers before running the `l3.py` script.

Once you've identified the necessary components, you can obtain the decryption keys from the Widevine license server using `l3.py`:

```
 cd l3decrypt
../bin/python3 l3.py 
```

You'll be prompted for the MPD and license URLs, and if you've done everything correctly, you'll receive a set of keys for your trouble:

```
PSSH obtained. 

TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQsIGNvbnNlY3RldHVyIGFkaXBpc2NpbmcgZWxpdCwgc2VkIGRvIGVpdXNtb2QgdGVtcG9yIGluY2lkaWR1bnQgdXQgbGFib3JlIGV0IGRvbG9yZSBtYWduYSAg

license response status: Response [200];

KID:KEY 47b53cd2729d42eb92c2ec9f24a0375f:1d58376bbb424d75a16f9ab20cc7bba8
```

The Protection System Specific Header (PSSH) is a block of data used in encrypted media streams. It contains information that a DRM system needs to decrypt the content, such as the key system, key IDs, and other DRM-specific data. Below that you'll see the relevant key ID and corresponding key(s).

### Downloading and Decrypting the Content

With the keys in hand, we can now download the encrypted content and decrypt it. We could look at the MPD file manually and identify the media files we're interest in, but it's much easier to use a tool like [yt-dlp](https://github.com/yt-dlp/yt-dlp) which supports MPD URLs. We'll also need the [bento4](https://www.bento4.com/) tools to decrypt our downloaded media and [ffmpeg](https://ffmpeg.org/) to combine our decrypted audio and video into a single file.

First, we'll need to install these dependencies:

```
yay -Syu yt-dlp ffmpeg bento4
```

Now download the encrypted media using `yt-dlp`:

```
yt-dlp --allow-unplayable-formats https://www.example.com/some/media.mpd
```

By default, this will fetch the highest resolution video and audio listed, but you can of course modify that with flags. In this example, we are limited to bit-rates that are available to the L3 CDM, but a real hardware Android device can be forced to give up an L1 key, allowing access to the highest quality content. To decrypt these files, we'll use`mp4decrypt` (installed as part of the `bento4` package), and repeat the `--key` flag for each key identified by `l3.py` in the previous step:

```
mp4decrypt --key KID:KEY encrypted_video.mp4 decrypted_video.mp4
mp4decrypt --key KID:KEY encrypted_audio.m4a decrypted_audio.m4a
```

At this point, we have decrypted audio and video streams that we can bundle together like so:

```
ffmpeg -i decrypted_video.mp4 -i decrypted_audio.m4a -vcodec copy -acodec copy decrypted_bundle.mp4
```

## Parting Thoughts

Ignoring the obvious, like why do emulated Android devices generate real Widevine keys, my main question is who is DRM targeting? Obviously, for someone who is not a developer, Widevine creates a major hurdle, but it introduces limitations for legitimate paying customers. For instance, the L3 CDMs that are built into browsers are limited by design — understandable since software run on your machine can be manipulated to give up its CMD — but for a paying customer, this means being unable to stream 4K or HDR content on their computer. Ironically, pirates won't have this issue because their content will be DRM-free.

The same is true of DRM in the world of PC gaming, wherein publishers introduce obtrusive DRM systems that routinely fail to work, cause crashes, require an internet connection, or otherwise grief legitimate owners of the software. Users who are inclined to do so simply download the DRM-free pirated versions and have a better overall user experience than paying customers, which is clearly a sub-optimal outcome. As developers, we must prioritize the UX and think more in terms of carrots than sticks.

There are clear success stories to learn from. iTunes for instance, was able to compete with the likes of Napster and Limewire piracy because the actual digital media, the music, was just one part of the overall offering. iTunes introduced deep integrations with other Apple products and was much easier and safer for end users. It added value rather than remove it, and the value was so compelling that it reshaped the entire music industry. This same industry has been upended again by streaming services that have become gateways to infinite music, a way to discover songs we never knew we would love, and a source of playlists that are perfectly curated to our tastes.

And in the world of PC gaming, Steam evolved from a necessary install for anyone who wanted to play Half-Life 2 to being a deeply profitable ecosystem that games live and die by. Steam significantly enhances the user experience by providing timely updates and patches, synced game saves across computers and devices, the ability to stream to other devices on your network, community reviews and curation, and even built-in support for mods. This changes the calculus for end-users, who *could* pirate the game, but can't pirate the experience, and thus decide the cost is worth the price.

Ultimately, those who want to pirate will do so. If they simply can't afford the product you're selling there is very little you can do to make them a paying customer. But for the subset of folks who don't see a clear value proposition, it is up to us to create user experiences that are worth paying for.