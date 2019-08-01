# Gstreamer udp webcam

### group 1（本地端进行传输摄像头视频）

##### sender：

gst-launch-1.0 ksvideosrc ! "image/jpeg,width=1280, height=720,framerate=30/1" ! rtpjpegpay ! udpsink host=127.0.0.1 port=5001

##### receiver：

gst-launch-1.0.exe -v udpsrc port=5001 ! application/x-rtp, encoding-name=JPEG,payload=26 !  rtpjpegdepay ! jpegdec !   autovideosink



### Group 2:（本地摄像头视频传输，以RAW的形式）

##### sender:

gst-launch-1.0.exe autovideosrc ! videoconvert ! rtpvrawpay ! udpsink host=127.0.0.1 port=5000 -v

##### receiver:

gst-launch-1.0.exe udpsrc port=5000 ! application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)YCbCr-4:2:2, depth=(string)8, width=(string)640, height=(string)480, colorimetry=(string)BT601-5, payload=(int)96, ssrc=(uint)2537320568, timestamp-offset=(uint)2046095812, seqnum-offset=(uint)31224, a-framerate=(string)30 ! rtpvrawdepay ! autovideosink



### group 3:（无人机视频通信，以RAW的形式）

##### sender:

gst-launch-1.0 -v v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=2560,height=720 ! videoconvert ! rtpvrawpay ! udpsink host = 192.168.2.164 port=5600

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)YCbCr-4:2:2, depth=(string)8, width=(string)2560, height=(string)720, colorimetry=(string)BT709-2, payload=(int)96, ssrc=(uint)1729488896, timestamp-offset=(uint)378348761, seqnum-offset=(uint)14020, a-framerate=(string)30 ! rtpvrawdepay !   autovideosink -v



### Group4:（本地视频通信，以h264码流）

##### sender:

gst-launch-1.0.exe -v ksvideosrc ! autovideoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp  ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink



### Group5:（本地视频通信，以h264码流，换一种编码）

##### sender:

gst-launch-1.0.exe -v ksvideosrc ! autovideoconvert ! omxh264enc ! rtph264pay ! udpsink host=127.0.0.1 port=5600

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp  ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink



### Group6：（多线程，播放同时保存在本地avi文件中

##### sender:

gst-launch-1.0 -v -e v4l2src device=/dev/video0 ! videoconvert !  video/x-raw,framerate=30/1,width=2560,height=720 ! timeoverlay valignment=top ! omxh264enc ! tee name=vsrc vsrc.! queue ! rtph264pay ! udpsink host=192.168.2.164 port=5600 vsrc.! queue ! avimux ! filesink location=v0.avi

###### if needed:

gst-launch-1.0 -v -e v4l2src device=/dev/video0 ! videoconvert !  video/x-raw,framerate=30/1,width=2560,height=720,**format=I420** ! timeoverlay valignment=top ! omxh264enc ! tee name=vsrc vsrc.! queue ! rtph264pay ! udpsink host=192.168.2.164 port=5600 vsrc.! queue ! avimux ! filesink location=v0.avi

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp  ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink



## notice:

ksvideosrc为windows下的命令，获取webcam，v4l2src device=/dev/video0 为linux下的命令。

接收端的主要格式是udpsrc-caps-rtpxdepay-sink，其中caps是基于发送方的udpsink之后的详细信息 caps而来，几乎全部复制。linux下复制的时候因为命令行的原因要在 ''（ '' 前加 '' \ '' 。

一般不会直接传送raw，而是会将其编码成h264等格式，如例4

编制成h264格式后，可以有效地降低延迟，提高稳定性

decodebin可以自动将各种编码格式解码，通过后接videoconvert，可转制成可播放视频。

tee用于将数据分成相同的多份，queue用于新线程。

在进行保存时遇到了最大的问题是无法创建视频信息，这个时候就需要pipeline在结束的时候发送一个EOS，实现只需要在命令行加上-e即可



QGroundControl在接收视频时不支持YUY2格式，而支持I420格式



### 信号测试

18块砖，每块长度49cm，共距882cm时，丢包严重。

零距离时基础延迟约三秒。

但是使用了h264编码之后，基础延迟约半秒左右，相同距离不再产生强烈丢包，可接收距离约为原来两倍多



### 连接无人机

ssh -p 5222 nvidia@192.168.2.1

