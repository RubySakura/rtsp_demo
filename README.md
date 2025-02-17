# 生成本地rtsp推流并使用webrtc-streamer转换为webrtc推流

## 使用软件
- *ffmpeg* 用于将本地视频/桌面/软件界面等转换为rtsp推流并推送至mediamtx
- *mediartx* rtsp服务器，ffmpeg生成的rtsp推流需要推送至此服务，供外部访问
- *webrtc-streamer* webrtc服务器，用于获取mediartx返回的rtsp推流，转换为webrtc推流，
- *VLC播放器* 用于测试推流是否可用

## 软件下载tips
根据所在操作系统下载对应软件，比如以Windows为例，下载：
- mediamtx_v1.11.3_windows_amd64.zip
- ffmpeg-master-latest-win64-gpl-shared.zip
- webrtc-streamer-v0.8.9-dirty-Windows-AMD64-Release.tar.gz

## 安装
ffmpeg，mediartx和webrtc-streamer都是解压缩可用，其中ffmpeg一般会将bin目录放到path中，这样就不需要每次进入目录再执行命令了

## 运行
需要按照顺序执行

### step1
执行mediartx命令，启动rtsp服务器，默认的端口是8554
```bash
mediamtx
```

### step2
执行ffmpeg命令
```bash
ffmpeg -stream_loop -1 -re -i demo.mp4 -c:v libx264 -profile:v baseline -level 3.0 -pix_fmt yuv420p -flags:v +global_header -bsf:v h264_mp4toannexb -g 30 -keyint_min 30 -c:a aac -f rtsp -rtsp_transport tcp rtsp://127.0.0.1:8554/stream
```
#### 输入相关参数
##### -stream_loop -1
此参数用于设置输入文件的循环播放次数。-1 表示无限循环，即 demo.mp4 文件会不断重复播放，以保证 RTSP 服务器能够持续提供视频流，满足客户端随时接入观看的需求。
##### -re
该参数的作用是按照输入文件的原始帧率读取和发送数据，模拟真实的实时流场景。它会限制数据的发送速度，使得视频帧按照其原本的时间间隔被发送出去，避免数据一次性快速发送，确保接收端能以正常的速度播放视频。
##### -i demo.mp4
指定输入文件为 demo.mp4，即从该文件中读取音视频数据进行后续处理。
视频编码相关参数
##### -c:v libx264
c:v 是 codec:video 的缩写，用于指定视频编码器。这里选择了 libx264，它是一个广泛使用的 H.264 视频编码器，具有良好的压缩性能和兼容性。
##### -profile:v baseline
设置 H.264 视频编码的配置文件为 baseline。baseline 配置文件提供了基本的编码功能，支持实时传输和低复杂度解码，适合对解码性能要求较低的设备，如一些早期的移动设备或网络带宽有限的场景。
##### -level 3.0
设定 H.264 编码的级别为 3.0。编码级别定义了视频流的复杂度和性能要求，包括最大帧率、分辨率、比特率等参数。level 3.0 适用于一些中等分辨率和帧率的视频，例如标清视频。
##### -pix_fmt yuv420p
指定视频的像素格式为 yuv420p。这是一种常见的采样格式，它对亮度信息（Y）进行全采样，而对色度信息（U 和 V）进行 2:1 的水平和垂直下采样，在保证一定画质的同时可以有效减少数据量。
##### -flags:v +global_header
为视频流添加全局头部信息。全局头部包含了视频编码所需的重要参数，如 SPS（序列参数集）和 PPS（图像参数集），接收端可以通过这些信息正确解码视频流。添加全局头部有助于提高视频流的兼容性和可解析性。
##### -bsf:v h264_mp4toannexb
bsf 是 bitstream filter 的缩写，即比特流过滤器。h264_mp4toannexb 过滤器的作用是将 H.264 视频流从 MP4 封装格式所需的字节流格式转换为 Annex B 字节流格式。Annex B 格式在 RTSP 传输中更为常用，它使用起始码（如 0x00 0x00 0x00 0x01）来分隔不同的 NAL 单元（网络抽象层单元）。
##### -g 30
g 表示 GOP（Group of Pictures，图像组）的大小，这里设置为 30 帧。GOP 是一组连续的视频帧，以一个 I 帧（关键帧）开始，后面跟随若干个 P 帧和 B 帧。较小的 GOP 大小可以降低解码延迟，但会增加视频文件的大小；较大的 GOP 大小则相反。
##### -keyint_min 30
设置最小关键帧间隔为 30 帧。这意味着在编码过程中，至少每隔 30 帧会插入一个关键帧，确保视频流在一定间隔内有可独立解码的关键帧，方便接收端进行随机访问和错误恢复。
音频编码相关参数
##### -c:a aac
c:a 是 codec:audio 的缩写，用于指定音频编码器。这里选择了 AAC（Advanced Audio Coding，高级音频编码），它是一种广泛应用的音频编码格式，具有较高的音质和压缩比，适合网络传输。
输出相关参数
##### -f rtsp
指定输出格式为 RTSP（Real Time Streaming Protocol，实时流协议）。FFmpeg 会将处理后的音视频数据封装成 RTSP 流进行输出。
##### -rtsp_transport tcp
设置 RTSP 传输协议为 TCP。TCP 是一种面向连接的可靠传输协议，使用 TCP 进行 RTSP 传输可以保证数据的可靠传输，避免丢包和乱序问题，适合对数据完整性要求较高的场景，但可能会增加一定的传输延迟。
##### rtsp://127.0.0.1:8554/stream
指定 RTSP 服务器的地址和端口，以及流的名称。127.0.0.1 是本地回环地址，表示在本地启动 RTSP 服务器；8554 是服务器监听的端口；stream 是流的标识符，客户端可以通过该地址请求这个特定的视频流。

也可以捕获桌面/软件界面等，此处不做赘述，没有使用摄像头是因为摄像头每次都会报错，如有兴趣可以尝试一下。

### step3
测试rtsp推流，打开vlc播放器，媒体 - 打开网络串流，填写 rtsp://localhost:8554/stream，可以正常播放视频证明rtsp推流可用。

也可以直接使用ffmpeg提供的ffplay命令测试，执行命令
```
ffplay rtsp://127.0.0.1:8554/stream
```
如果 ffplay 成功打开并开始播放视频，说明 RTSP 流正常工作；若出现错误信息，如 “Failed to connect”，则表示连接失败，需要进一步排查问题。

### step4
执行 webrtc-streamer命令
```bash
webrtc-streamer rtsp://localhost:8554/stream
```
此时是直接指定了rtsp地址，也可以在后续操作中指定。

访问webrtc-streamer默认的localhost:8000，检查推流是否成功，成功可以进行后续操作了。
