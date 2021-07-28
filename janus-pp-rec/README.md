# 录制

## 修改IM服务配置文件
打开总是录制开关，录制路径不要改。
```
#音视频服务录制策略。0 不允许录制，1 总是录制，2 客户端选择是否录制。
conference.record_strategy 1
conference.record_path /opt/janus/share/janus/recordings
```
录制的文件会存入到服务启动时指定的目录去。

## 录制文件的处理
录制文件的格式为```mjr```，这种格式是直接把RTP信息写入文件，这样就不会对服务器造成任何的计算压力。录制后需要进行后期处理才能够播放，下载janus-pp-rec，然后执行：
```
./janus-pp-rec  videoroom-${roomId}-user-${userId}-${timestamp}-audio.mjr videoroom-${roomId}-user-${userId}-${timestamp}.opus
./janus-pp-rec  videoroom-${roomId}-user-${userId}-${timestamp}-video.mjr videoroom-${roomId}-user-${userId}-${timestamp}.mp4
```
至此就转为常见格式了，再使用ffmepg把音频和视频结合成一个文件。janus只能支持这种格式，无法支持别的格式。

详情请参考[janus录制](https://janus.conf.meetecho.com/docs/recordings.html)

## 环境依赖
janus-pp-rec依赖ffmpeg-dev，在ubuntu上运行下面命令安装
```
sudo apt-get update
sudo apt-get install ffmpeg
```
在centos7上运行下面命令
```
sudo yum install epel-release
sudo yum localinstall --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm
sudo yum install ffmpeg ffmpeg-devel
```
其它系统或者遇到问题，请百度或者谷歌查找解决


## 鸣谢
感谢[janus](https://github.com/meetecho/janus-gateway)提供如此好的开源产品。
