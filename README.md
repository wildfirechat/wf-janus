# wf-janus
野火音视频高级版服务，基于[janus](https://github.com/meetecho/janus-gateway)二次开发而来，开发仅限于与野火IM对接，没有做任何功能的修改。使用方法如下：

## 服务器的选用
由于使用的是SFU架构，所有流量都经过媒体服务，对带宽的要求非常高。如果使用固定带宽，价格会非常高昂。建议使用按照流量计费，大部分云服务器都能达到100Mbps，由于是按量计费，费用反而节省很多。

## 导入docker镜像
仅支持docker方式，x64镜像在[这里](https://static.wildfirechat.cn/wildfire_janus_amd64.tar)，下载完之后检查[md5](https://static.wildfirechat.cn/wildfire_janus_amd64.md5)；arm64镜像在[这里](https://static.wildfirechat.cn/wildfire_janus_arm64.tar)，下载完之后检查[md5](https://static.wildfirechat.cn/wildfire_janus_arm64.md5)

镜像下载之后通过下属命令导入镜像:
```
sudo docker load -i wildfire_janus_amd64.tar
```
> arm服务器请导入对应的arm镜像

## 修改配置
下载```janus_config```到服务器。
1. 修改```janus.transport.mqtt.jcfg```
    ```
    general: {
      enabled = true            # Whether the support must be enabled
      im_host = "imdev.wildfirechat.cn"  # Wildfire IM server host
      im_port = 80  # Wildfire IM server http port。请保持不变，仅当改动过客户端端口时修改
      client_id = "conference_server_1"				# Client identifier
      subscribe_topic = "to-janus"		# Topic for incoming messages，需要和im server配置里面的 conference.to_topic 一致
      publish_topic = "from-janus"		# Topic for outgoing messages，需要和im server配置里面的 conference.from_topic 一致
      ...
    }
    ```
    >im_host 要使用专业版的授权域名，client_id为了安全，请使用一个随机的uuid，client_id和subscribe_topic和publish_topic要和IM服务配置中的值对应。
2. 修改```janus.jcfg```
    ```
    media: {
      ...
      rtp_port_range = "20000-40000"
      ...
    }

    nat: {
      ...
      ice_lite = true
      ...
      # 多网卡时需要打开并指定网卡
      #ice_enforce_list = "eth0"
      ...
    }

    ```
    > rtp_port_range 为媒体流使用的UDP端口范围，端口至少5000个。UDP端口范围默认是20000-40000，如果修改端口范围请确保在这个区间内。需要确保服务器防火墙和安全组放开权限，需要确保客户端网络防火墙放开权限。

    > 如果宿主机上有多于一个网卡，需要指定使用那个网卡，请打开配置文件中的```ice_enforce_list```配置，设置上外网IP所对应的网卡。

3. 修改```janus.plugin.videoroom.jcfg```
    ```
    string_ids = true

    ```

## 修改IM服务
IM服务配置文件中修改音视频服务的client_id、subscribe_topic和publish_topic。然后启动IM服务。

## 启动媒体服务
IM服务启动之后才可以启动媒体服务。请使用下面命令启动：
```
sudo docker run -it -e DOCKER_IP=YOUR_DOCKER_IP --name wf_janus_server --net host -v PATH_TO_janus_config:/var/janus/janus/etc/janus -v PATH_TO_RECORDS_FOLDER:/opt/janus/share/janus/recordings wildfire_janus
```
注意```YOUR_DOCKER_IP```为服务器的外网IP，```PATH_TO_janus_config```为配置文件的路径，```PATH_TO_RECORDS_FOLDER```为录制文件保存目录，**这三个占位符本身需要修改为具体的值，而不是修改```:```后面那部分，其他的不用修改**, 下面是修改后的命令示例:

```
sudo docker run -it -e DOCKER_IP=46.8.147.195 --name wfc_janus_server --net host -v /Users/userName/wf-janus/config:/var/janus/janus/etc/janus -v /Users/userName/records_folder:/opt/janus/share/janus/recordings wildfire_janus
```

> docker 挂载主机目录，请参考 [docker run -v](https://www.cnblogs.com/starfish29/p/10653960.html), [docker volums](https://docs.docker.com/storage/volumes/)

## 客户端
确保客户端能够正常运行，能够收发消息。替换音视频高级版的SDK，测试音视频通话。

## 水平扩展
客户部署多个媒体服务来水平扩展媒体服务。当用户发起音视频通话时，IM服务会hash分配媒体服务。

## 录制文件的处理
录制文件的格式为```mjr```，这种格式是直接把RTP信息写入文件，这样就不会对服务器造成任何的计算压力。录制后需要进行后期处理才能够播放，下载[janus-pp-rec](./janus-pp-rec)，然后执行：
```
./janus-pp-rec  videoroom-${roomId}-${timestamp}-audio.mjr videoroom-${roomId}-${timestamp}.opus
./janus-pp-rec  videoroom-${roomId}-${timestamp}-video.mjr videoroom-${roomId}-${timestamp}.mp4
```
至此就转为常见格式了，再使用ffmepg把音频和视频结合成一个文件。janus只能支持这种格式，无法支持别的格式。

详情请参考[janus录制](https://janus.conf.meetecho.com/docs/recordings.html)


## 鸣谢
感谢[janus](https://github.com/meetecho/janus-gateway)提供如此好的开源产品。
