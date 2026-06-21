---
title: Youtube with yt-dlp & Widevine DRM decoding
date: 2025-01-03
redirectFrom: drafts/youtube-widevine-drm
---


First we must ask ourselves, what is Widevine?
* DRM solution acquired by google in 2010
* Three levels, L1, L2, L3
  * L1 (in android) is done inside the Trusted Execution Environment (TEE), so hardware dependant
    * and is often required for HD decryption
  * L2 doesn't exist in android
  * L3 is software based


## Terminology
* License Request: DRM fuckery talks to license server to get content keys
  * ex license server: youtube.com/youtubei/v1/player/get_drm_license
* DASH: [Dynamic adaptive streaming over HTTP](https://standards.iso.org/ittf/PubliclyAvailableStandards/c083314_ISO_IEC%2023009-1_2022\(en\).zip)
  I think this is basically the spec for an MPD manifest, which most streaming sites using widevine use?
  * PSSH: Protection System Specific Header
    * part of a license request
  * MPD: Media Presentation Description
    * I think this is an xml manifest that says what the content URLs are, supposed to define pssh, and maybe license servers
* CDM

## Open Quetions
* the L1/L3 device keys are genrated by the Root of Trust 128 AES device key, where does that come from (especially in an emulator?)
* Can we get the device key and generate the L1/L3 keys ourselves?

## Prior Art
I did the "make it kinda work" half before the "learn how it's supposed to work" half and I must say, cargo cult script kiddie shit sucks.
Did work though.
### Research
The explinations above are me butchering these
* [Exploring Widevine for Fun and Profit from Gwendal Patat, Mohamed Sabt, and Pierre-Alain Fouque](https://arxiv.org/abs/2204.09298)
* [Your DRM Can Watch You Too: Exploring the Privacy Implications of Browsers (mis)Implementations of Widevine EME from  Gwendal Patat, Mohamed Sabt, and Pierre-Alain Fouque](https://arxiv.org/abs/2308.05416)
### Application
I'm mostly summarizing these after a fair amount of trial and an unfair amount of error
* [Mo Ismailzai has a blog post that's close to what I did](https://www.ismailzai.com/blog/picking-the-widevine-locks)
* [cedric8528 on VideoHelp has a post](https://forum.videohelp.com/threads/408031-Dumping-Your-own-L3-CDM-with-Android-Studio)
* [[ss]vegeta on VideoHelp had helpful tips for using the dumper](https://forum.videohelp.com/threads/404219-How-To-Dump-L3-CDM-From-Android-Device-s-%28ONLY-Talk-About-Dumping-L3-CDMS%29/page6?s=da067ba2bab6ce841c77ae58772f9029#post2646150)
* [stabbedbybrick on VideoHelp showed how to get youtube keys](https://forum.videohelp.com/threads/408556-How-to-decrypt-YouTube-Movies-TV-files/page2#post2687354)

## Actually doing it
### Get L3 DDM
#### Tools
* android emulator running Oreo x86 Android 8.0 (Google APIs)
  * 10/Q didn't work, and without google APIs won't have chrome so we won't have widevine support
    I think the internet said Google Play vs Google APIs won't work but I don't remember why.
* https://github.com/frida/frida/releases/
  * show more, find frida-server-whatever
* https://github.com/Diazole/dumper


#### steps
* run emulator
  * install via studio, or maybe something like
    ```
    sdkmanager "platform-tools" "platforms;android-26"
    sdkmanager "system-images;android-26;google_apis;x86"
    sdkmanager --licenses
    ```
  * `emulator -avd Medium_Phone_API_26 -wipe-data -netdelay none -netspeed full`
* push frida-server to /sdcard, move to `/data/local/tmp` & chmod +x
* run frida-server as root
* find libwvhidl.so, copy to somewhere readable and pull it
* Look at it with `strings`, the functions names you want are between
  `_lcc43` and `_ZN6wvoec333OEMCrypto_Level3AndroidFileSystem4ReadEPKcPvj`(Second occourance of OEMCrypto_Level3 in the file)
* put those function names in dumper/Helpers/scripts.sh KNOWN_DYNAMIC_FUNCTION_NAMES
* run dumper, this will listen in on those functions for drm functions and log keys
  * I had a *lot* of issues with this I think it just started working once I went old enough? Don't really remmeber
* Open DRM in a chrome tab in the phone
  * `adb shell am start -n com.android.chrome/com.google.android.apps.chrome.Main -d "https://bitmovin.com/demos/drm"`
  * You will have to click accept a few times or whatever
* Hope dumper writes `key_dumps/Android\ Emulator\ */private_keys/*/*/client_id.bin` and `private_key.pem`

These steps are fairly well documented in the first two prior art posts.


### Something something youtube
Knowing anything about widevine or DASH would *really* have helped here.

#### Tools
* https://github.com/medvm/widevine_keys
* https://forum.videohelp.com/threads/408556-How-to-decrypt-YouTube-Movies-TV-files/page2#post2687354
  * copy it into a script in widivine_keys directory
* https://github.com/axiomatic-systems/Bento4

#### Steps
* `cp dumper/key_dumps/*/private_keys/*/*/client_id.bin widevine_keys/cdm/devices/device_client_id_blob`
  `cp dumper/key_dumps/*/private_keys/*/*/private_key.pem widevine_keys/cdm/devices/device_private_key`
* open youtube video in browser, find keyserver request in network
  * youtube keyserver: https://www.youtube.com/youtubei/v1/player/get_drm_license
* copy request, put into widivine_keys/headers.py
  * https://addons.mozilla.org/en-US/firefox/addon/copy-as-python-requests/
  * https://curlconverter.com/
* Put keyserver url in stabbedbybrick's script
* run stabbedbybrick's script, get keys output
* `yt-dlp --allow-unplayable-formats https://www.youtube.com/watch?v=CYV9LmANWOQ`
* `./mp4decrypt <--key whatever>... encrypted.mp4 decrypted.mp4`
* `ffmpeg -i decrypted_video.mp4 -i decrypted_audio.m4a -vcodec copy -acodec copy decrypted_bundle.mp4`


#### NEXT STEPS: Making this not suck to do
##### Expectations
* yt-dlp interface that handles this stupid stuff with no manual steps
