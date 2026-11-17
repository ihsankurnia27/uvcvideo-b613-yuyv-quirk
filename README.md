# uvcvideo: quirk for the broken YUYV format of the SunplusIT IR camera (04f2:b613)

Minimal repository carrying a two-patch `uvcvideo` series for the linux-media
mailing list. Only `drivers/media/usb/uvc/` is included, taken from Linux
`v7.0.14`; the two patches at the tip of branch `b613-yuyv-quirk` are the series
submitted to <linux-media@vger.kernel.org>.

```
Baseline: drivers/media/usb/uvc from Linux v7.0.14
media: uvcvideo: Skip streaming interfaces with no usable formats
media: uvcvideo: Ignore the broken YUYV format of the SunplusIT IR camera
```

## What it fixes

The SunplusIT integrated IR camera of the Lenovo ThinkPad T480 (`04f2:b613`)
advertises two formats: an uncompressed YUYV one and MJPEG. The YUYV format is
broken: the device sends 10-bit packed sensor data that cannot be interpreted as
8-bit YUYV, and it keeps the IR emitter switched off, so the frames are almost
black. Applications that prefer an uncompressed format over MJPEG (Chrome and
anything else built on libwebrtc) therefore showed only dark noise, while
applications that pick MJPEG (Howdy for instance) worked.

The series adds `UVC_QUIRK_DROP_YUYV` and makes `uvc_parse_format()` skip that
format descriptor for this device, so userspace negotiates the working MJPEG
mode. A prerequisite patch makes `uvc_parse_streaming()` tolerate a streaming
interface whose format descriptors are all skipped instead of failing the
registration of the whole device.

## Fetching / testing the series

```sh
git fetch https://github.com/ihsankurnia27/uvcvideo-b613-yuyv-quirk b613-yuyv-quirk
git checkout FETCH_HEAD
```

To apply on a full kernel tree:

```sh
git format-patch -2 FETCH_HEAD      # or: b4 shazam <message-id of the cover letter>
```
