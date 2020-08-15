# wf-janus
野火音视频高级版服务，基于[janus](https://github.com/meetecho/janus-gateway)二次开发而来，开发仅限于与野火IM对接，没有做任何功能的修改。使用方法如下：

## 服务器的选用
由于使用的是SFU架构，所有流量都经过媒体服务，对带宽的要求非常高。如果使用固定带宽，价格会非常高昂。建议使用按照流量计费，大部分云服务器都能达到100Mbps，由于是按量计费，费用反而节省很多。

## 导入docker镜像
仅支持docker方式，x64镜像在[这里](https://static.wildfirechat.cn/wildfire_janus_amd64.tar)，下载完之后检查[md5](https://static.wildfirechat.cn/wildfire_janus_amd64.md5)；arm64镜像在[这里](https://static.wildfirechat.cn/wildfire_janus_arm64.tar)，下载完之后检查[md5](https://static.wildfirechat.cn/wildfire_janus_arm64.md5)

镜像下载之后通过下属命令导入镜像:
```
sudo docker import wildfire_janus_amd64.tar wildfire_janus
```
> arm服务器请导入对应的arm镜像

## 修改配置
下载```janus_config```到服务器。修改```janus.transport.mqtt.jcfg```
```
general: {
  enabled = true            # Whether the support must be enabled
  im_host = "imdev.wildfirechat.cn"  # Wildfire IM server host
  client_id = "guest"				# Client identifier
  subscribe_topic = "to-janus"		# Topic for incoming messages
  publish_topic = "from-janus"		# Topic for outgoing messages

```

修改```janus.plugin.videoroom.jcfg```
```
string_ids = true

```

im_host 要使用专业版的授权域名，client_id为了安全，请使用一个随机的uuid，client_id和subscribe_topic和publish_topic要和IM服务配置中的值对应。


## 修改IM服务
IM服务配置文件中修改音视频服务的client_id、subscribe_topic和publish_topic。然后启动IM服务。

## 启动媒体服务
IM服务启动之后才可以启动媒体服务。请使用下面命令启动：
```
sudo docker run -it -e DOCKER_IP=192.168.3.102 --name wf_janus_server --net host -v PATH_TO_janus_config:/var/janus/janus/etc/janus wildfire_janus
```
注意```DOCKER_IP```为服务器的外网IP，```PATH_TO_janus_config```为配置文件的路径。

## 客户端
确保客户端能够正常运行，能够收发消息。替换音视频高级版的SDK，测试音视频通话。

## 水平扩展
客户部署多个媒体服务来水平扩展媒体服务。当用户发起音视频通话时，IM服务会hash分配媒体服务。

## 鸣谢
感谢[janus](https://github.com/meetecho/janus-gateway)提供如此好的开源产品。
