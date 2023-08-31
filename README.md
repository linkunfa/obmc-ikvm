# OpenBMC IpKVM Server

The obmc-ikvm application is a VNC server that provides access to the host
graphics output. The application interfaces with the video device on the BMC
that captures the host graphics, and then serves that video data on the RFB
(remote framebuffer, also known as VNC) protocol. The application also
interfaces with the BMC USB gadget device to pass HID events from the BMC to
the host, allowing the user to interact with the host system.

## Usage

Once the host is running and an appropriate HID gadget device is instantiated
on the BMC, the application can be started with the following command:
``` obmc-ikvm -v <video device path> -i <HID gadget device path> ```

For example:

``` obmc-ikvm -v /dev/video0 -i /dev/hidg0 ```


OK. Will drop them in the next version.


Actually driver will store the frames and counts for each buffer index till userspace dequeues them.

Ex. assume that driver has captured 3 frames now:
1st capture (buffer index = 0):
  video->list[0] => store the list of HEXTILE rects for the 1st frame
  video->rect[0] => store the rect count of video->list[0]
  
2nd capture (buffer index = 1):
  video->list[1] => store the list of HEXTILE rects for the 2nd frame
  video->rect[1] => store the rect count of video->list[1]

3rd capture (buffer index = 2):
  video->list[2] => store the list of HEXTILE rects for the 3rd frame
  video->rect[2] => store the rect count of video->list[2]

When userspace dequeues the 1st buffer (video->list[0]), it needs to know the count of
HEXTILE rectangles in the buffer, so after dequeueing the buffer it will call this control to get the rect count (video->rect[0]).
And when a buffer is dequeued, npcm_video_buf_finish() will be called, in which the buffer index (in this example, buffer index = 0) will be stored to video->vb_index.
Then when userspace calls this control, npcm_video_get_volatile_ctrl() will return the rect count of vb_index = 0.
In this way, I think userspace is always reading the correct control's value even if userspace is slow.
Does it make sense to you or is there anything I missed?
