# Gstreamer udp webcam

### group 1

##### sender：

gst-launch-1.0 ksvideosrc ! "image/jpeg,width=1280, height=720,framerate=30/1" ! rtpjpegpay ! udpsink host=127.0.0.1 port=5001

##### receiver：

gst-launch-1.0.exe -v udpsrc port=5001 ! application/x-rtp, encoding-name=JPEG,payload=26 !  rtpjpegdepay ! jpegdec !   autovideosink



### group 2:

##### sender:

gst-launch-1.0 -v -v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=2560,height=720 ! videoconvert ! rtpvrawpay ! udpsink host = 192.168.2.164 port=5600

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)YCbCr-4:2:2, depth=(string)8, width=(string)2560, height=(string)720, colorimetry=(string)BT709-2, payload=(int)96, ssrc=(uint)1729488896, timestamp-offset=(uint)378348761, seqnum-offset=(uint)14020, a-framerate=(string)30 ! rtpvrawdepay !   autovideosink -v



## notice:

接收端的主要格式是udpsrc-caps-rtpxdepay-sink，其中caps是基于发送方的udpsink之后的详细信息 caps而来，几乎全部复制。linux下复制的时候因为命令行的原因要在 ''（ '' 前加 '' \ '' 。

一般不会直接传送raw，而是会将其编码成h264等格式，于是乎会在rtpvrawpay前加上omxh264enc，将rtpvrawpay改为rtph264pay，接收端则为udpsrc-caps-rtph264depay-omxhdec-sink。//待验证

主要是这个webcam太毒了，他只能支持2560 * 720和3840 * 1080，头大。

