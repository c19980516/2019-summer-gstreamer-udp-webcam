# Gstreamer udp webcam

### group 1

##### sender：

gst-launch-1.0 ksvideosrc ! "image/jpeg,width=1280, height=720,framerate=30/1" ! rtpjpegpay ! udpsink host=127.0.0.1 port=5001

##### receiver：

gst-launch-1.0.exe -v udpsrc port=5001 ! application/x-rtp, encoding-name=JPEG,payload=26 !  rtpjpegdepay ! jpegdec !   autovideosink



### Group 2:

##### sender:

gst-launch-1.0.exe autovideosrc ! videoconvert ! rtpvrawpay ! udpsink host=127.0.0.1 port=5000 -v

##### receiver:

gst-launch-1.0.exe udpsrc port=5000 ! application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)YCbCr-4:2:2, depth=(string)8, width=(string)640, height=(string)480, colorimetry=(string)BT601-5, payload=(int)96, ssrc=(uint)2537320568, timestamp-offset=(uint)2046095812, seqnum-offset=(uint)31224, a-framerate=(string)30 ! rtpvrawdepay ! autovideosink



### group 3:

##### sender:

gst-launch-1.0 -v v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=2560,height=720 ! videoconvert ! rtpvrawpay ! udpsink host = 192.168.2.164 port=5600

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)RAW, sampling=(string)YCbCr-4:2:2, depth=(string)8, width=(string)2560, height=(string)720, colorimetry=(string)BT709-2, payload=(int)96, ssrc=(uint)1729488896, timestamp-offset=(uint)378348761, seqnum-offset=(uint)14020, a-framerate=(string)30 ! rtpvrawdepay !   autovideosink -v



### Group4:（h264

##### sender:

gst-launch-1.0.exe -v ksvideosrc ! autovideoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp  ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink



## notice:

ksvideosrc为windows下的命令，获取webcam，v4l2src device=/dev/video0 为linux下的命令。

接收端的主要格式是udpsrc-caps-rtpxdepay-sink，其中caps是基于发送方的udpsink之后的详细信息 caps而来，几乎全部复制。linux下复制的时候因为命令行的原因要在 ''（ '' 前加 '' \ '' 。

一般不会直接传送raw，而是会将其编码成h264等格式，如例4

编制成h264格式后，可以有效地降低延迟，提高稳定性

decodebin可以自动将各种编码格式解码，通过后接videoconvert，可转制成可播放视频。



### 信号测试

18块砖，每块长度49cm，共距882cm时，丢包严重。

零距离时基础延迟约三秒。

但是使用了h264编码之后，基础延迟约半秒左右，相同距离不再产生强烈丢包，可接收距离约为原来两倍多



### 多线程

##### 播放同时保存在本地avi文件中

##### 本地测试

gst-launch-1.0.exe autovideosrc num-buffers=50 ! videoconvert ! tee name=vsrc vsrc.! queue ! autovideosink vsrc.! queue ! avimux ! filesink location=test.avi -v

##### sender:

gst-launch-1.0.exe -v ksvideosrc ! autovideoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 -e

gst-launch-1.0.exe -v v4l2src device=/dev/video0 ! autovideoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 -e

##### receiver:

gst-launch-1.0.exe -v udpsrc port=5600 ! application/x-rtp  ! rtph264depay ! avdec_h264 ! videoconvert ! tee name=vsrc vsrc.! queue ! autovideosink vsrc.! queue ! x264enc tune=zerolatency ! mp4mux ! filesink location=video.mp4 -e



### notice：

tee用于将数据分成相同的多份，queue用于新线程。

在进行保存时遇到了最大的问题是无法创建视频信息，这个时候就需要pipeline在结束的时候发送一个EOS，实现只需要在命令行加上-e即可



gst-launch-1.0.exe -v ksvideosrc ! video/x-raw,width=640,height=480 ! videoconvert ! x264enc tune=zerolatency ! tee name=vsrc vsrc.! queue ! rtph264pay ! udpsink host=183.173.81.133 port=5600 vsrc.! queue ! mp4mux ! filesink location=v.mp4 -e



gst-launch-1.0.exe -v ksvideosrc ! autovideoconvert ! clockoverlay valignment=top halignment=left font-desc="Sans, 72" ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=183.173.81.133 port=5600 buffer-size=2147483647 blocksize=max



gst-launch-1.0 -v -e v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=2560,height=720 ! videoconvert ! timeoverlay valignment=top halignment=left font-desc="Sans, 72" ! x264enc tune=zerolatency ! tee name=vsrc vsrc.! queue ! rtph264pay ! udpsink host=192.168.2.164 port=5600 vsrc.! queue ! avimux ! filesink location=v0.avi



4294967295

2147483647

3min-10s

3min40s-10s



35s-40s

45s-1min10s

1min20s-1min55s

2m50s-3m05s



50s-1m15s

2m35s-3m10s000	



5s-10s

30s-35s

50s-55s

1:25-1:30

1:50-2:00



gst-launch-1.0 -v -e v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=2560,height=720 ! videoconvert ! timeoverlay valignment=top halignment=left font-desc="Sans, 72" ! omxh264enc ! tee name=vsrc vsrc.! queue ! rtph264pay ! udpsink host=192.168.2.164 port=5600 vsrc.! queue ! avimux ! filesink location=v0.avi

gst-launch-1.0 -v -e v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=2560,height=720 ! videoconvert ! timeoverlay valignment=top ! omxh264enc ! tee name=vsrc vsrc.! queue ! rtph264pay ! udpsink host=192.168.2.164 port=5600 vsrc.! queue ! avimux ! filesink location=v0.avi

gst-launch-1.0 -v -e v4l2src device=/dev/video0 ! video/x-raw,framerate=30/1,width=2560,height=720 ! videoconvert ! timeoverlay valignment=top ! omxh264enc ! tee name=vsrc vsrc.! queue ! rtph264pay ! udpsink host=192.168.2.164 port=5600 vsrc.! queue ! avimux ! filesink location=v3.avi