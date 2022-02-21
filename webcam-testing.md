# webcam testing

## Check webcam capabilities

### Method 1

```shell
 # First, find Bus and Device info of webcam
 lsusb
 
 # Then list device info
 lsusb -s <bus>:<device> -v
```

### Method 2

```shell
# First, install v4l-utils, if not installed
 sudo apt install v4l-utils
 
 # Show device format info
 v4l2-ctl --list-formats-ext -d /dev/video2
```

## Test webcam with Gstreamer

### View webcam output

When the webcam output is in MJPEG format

```shell
# Live view
gst-launch-1.0 v4l2src device=/dev/video2 ! image/jpeg,width=2560,height=1440 ! jpegdec ! xvimagesink

# Save output into a image file
gst-launch-1.0 v4l2src device=/dev/video2 ! image/jpeg,width=2560,height=1440 ! jpegparse ! filesink location=./foo.jpg
```
