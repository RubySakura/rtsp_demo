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
ffmpeg -stream_loop -1 -re -i "your_local_video.mp4" -c:v libx264 -preset ultrafast -tune zerolatency -pix_fmt yuv420p -f rtsp rtsp://127.0.0.1:8554/stream
```
#### 参数解释
- stream_loop -1：表示无限循环播放指定的视频文件。如果将 -1 替换为正整数 n，则视频会循环播放 n 次。
- re：以本地视频的帧率读取视频文件，模拟实时流的传输速度，避免推流速度过快。
- i "your_local_video.mp4"：指定要推流的本地视频文件路径。你需要将 "your_local_video.mp4" 替换为实际的视频文件名称和路径。
- c:v libx264：使用 libx264 作为视频编码器。
- preset ultrafast：设置编码预设为 ultrafast，以提高编码速度。
- tune zerolatency：针对实时流进行优化，减少编码延迟。
- b:v 1000k：设置视频比特率为 1000kbps。你可以根据实际需求调整该值。
- f rtsp：指定输出格式为 RTSP 流。rtsp://localhost:8554/stream：指定 RTSP 流的输出地址。你可以根据实际情况修改地址和端口。

也可以捕获桌面/软件界面等，此处不做赘述，没有使用摄像头是因为摄像头每次都会报错，如有兴趣可以尝试一下

### step3
测试rtsp推流，打开vlc播放器，媒体 - 打开网络串流，填写 rtsp://localhost:8554/stream，可以正常播放视频证明rtsp推流可用

### step4
执行 webrtc-streamer命令
```bash
webrtc-streamer rtsp://localhost:8554/stream
```

此时访问webrtc-streamer默认的localhost:8000，检查推流是否成功，成功可以进行后续操作了