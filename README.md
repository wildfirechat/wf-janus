# 野火音视频高级版媒体服务
野火音视频高级版媒体服务，基于[janus](https://github.com/meetecho/janus-gateway)二次开发而来，开发仅限于与野火IM对接，修改开源代码在[这里](https://github.com/heavyrain2012/janus-gateway)，没有做任何功能的修改。使用方法如下：

## 高级版的特点
高级版是通过云服务器中转，通信质量更有保障，详情请参考[链接](https://docs.wildfirechat.cn/blogs/野火音视频简介.html)，另外支持超级会议模式，支持无限人数参与，请参考[链接](https://docs.wildfirechat.cn/blogs/野火音视频之超级会议.html)。

## 服务器的选用
由于使用的是SFU架构，所有流量都经过媒体服务，对带宽的要求非常高。如果使用固定带宽，价格会非常高昂。建议使用按照流量计费，大部分云服务器都能达到200Mbps，可以支持较大的通话容量。按量计费相比按带宽计费，费用会节省更多。

客户端会跟Janus服务进行直连，所以就需要Janus服务部署在公网，或者有足够多的UDP端口映射到公网。

## 导入docker镜像
服务在2021.8.4日、2022.7.14和2024.10.4有重大升级，最新的版本从这里下载：x86_64镜像在[这里](http://static.wildfirechat.net/wildfire_janus_amd64_ff1f4f4e.tar)，下载完之后检查[md5](http://static.wildfirechat.net/wildfire_janus_amd64_ff1f4f4e.md5)；arm64镜像在[这里](http://static.wildfirechat.net/wildfire_janus_arm64_ff1f4f4e.tar)，下载完之后检查[md5](http://static.wildfirechat.net/wildfire_janus_arm64_ff1f4f4e.md5)

如果IM服务是2024.10.4日之前的版本，从这里下载：x86_64镜像在[这里](http://static.wildfirechat.net/wildfire_janus_amd64_before_20241004.tar)，下载完之后检查[md5](http://static.wildfirechat.net/wildfire_janus_amd64_before_20241004.md5)；arm64镜像在[这里](http://static.wildfirechat.net/wildfire_janus_arm64_before_20241004.tar)，下载完之后检查[md5](http://static.wildfirechat.net/wildfire_janus_arm64_before_20241004.md5)

如果IM服务是2022.7.14日之前的版本，从这里下载：x86_64镜像在[这里](http://static.wildfirechat.cn/wildfire_janus_amd64_before_20220714.tar)，下载完之后检查[md5](http://static.wildfirechat.cn/wildfire_janus_amd64_before_20220714.md5)；arm64镜像在[这里](http://static.wildfirechat.cn/wildfire_janus_arm64_before_20220714.tar)，下载完之后检查[md5](http://static.wildfirechat.cn/wildfire_janus_arm64_before_20220714.md5)

如果IM服务是2021.8.4日之前的版本，从这里下载：x86_64镜像在[这里](http://static.wildfirechat.net/wildfire_janus_amd64_legacy.tar)，下载完之后检查[md5](http://static.wildfirechat.net/wildfire_janus_amd64_legacy.md5)；arm64镜像在[这里](http://static.wildfirechat.net/wildfire_janus_arm64_legacy.tar)，下载完之后检查[md5](http://static.wildfirechat.net/wildfire_janus_arm64_legacy.md5)

镜像下载之后通过下属命令导入镜像:
```
sudo docker load -i wildfire_janus_amd64.tar
```
> arm服务器请导入对应的arm镜像

## 修改配置
下载[janus_config](./janus_config)目录到服务器。
1. 修改目录下的```janus.transport.mqtt.jcfg```
    ```
    general: {
      enabled = true            # Whether the support must be enabled
      im_host = "im_server_host"  # Wildfire IM server host
      im_port = 80  # Wildfire IM server http port。请保持不变，仅当改动过客户端端口时修改
      client_id = "conference_server_1"				# 必须是IM服务配置conference.client_list的id之一，如果有多个Janus服务，不能重复。
      ...
    }
    ```
    >im_host和im_port是janus访问IM服务短连接的地址和端口，如果janus与im服务在同一个内网中，建议可以使用内网地址，如果使用域名，需要确保janus能够访问这个地址，client_id为了安全，请使用一个随机的uuid，如果部署多台，需要确保每台都的client_id都是唯一的，client_id需要配置到IM服务配置文件中的conference.client_list列表中去。
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
      ice_enforce_list = "eth0"
      ...
    }

    ```
    > rtp_port_range 为媒体流使用的UDP端口范围，端口至少5000个。UDP端口范围默认是20000-40000，如果修改端口范围建议最小端口大于10000，最大不能超过65535。需要确保服务器防火墙和安全组放开权限，需要确保客户端网络防火墙放开权限。

    > 配置文件中的```ice_enforce_list```配置需要设置上外网IP所对应的网卡。查找网卡名称可以用```ifconfig```命令来查看。

## 防火墙和安全组设置
服务器需要开放UDP指定端口范围的入访权限（默认是20000-40000）。服务器需要去连接IM，需要开通到IM服务的80/1883端口。

## 修改IM服务
IM服务配置文件中修改音视频服务的client_id列表(conference.client_list)、signal_server_address(conference.signal_server_address)。其中：
1. client_id列表要包含所有janus服务的clientId，以逗号分割，为了以后扩展方便，可以预先写入多个id；
2. signal_server_address填写***当前IM服务的内网IP，不能是域名或者别名***，Janus服务只认IP地址，注意这个配置为***当前IM服务的内网IP地址，不是音视频服务的内网IP地址***；

修改配置后重新启动IM服务。

## 启动媒体服务
IM服务启动之后才可以启动媒体服务。请使用下面命令启动：
```
sudo docker run -it --privileged=true -e DOCKER_IP=YOUR_PUBLIC_IP(服务器公网IP) --name wf_janus_server --net host -v PATH_TO_janus_config(配置文件目录的绝对路径):/var/janus/janus/etc/janus -v PATH_TO_RECORDS_FOLDER(用来保存会议录制的目录的绝对路径):/opt/janus/share/janus/recordings wildfire_janus
```
注意```YOUR_PUBLIC_IP(服务器公网IP)```为Janus服务所在服务器的外网IP，```PATH_TO_janus_config(配置文件目录的绝对路径)```为配置文件的路径，```PATH_TO_RECORDS_FOLDER(用来保存会议录制的目录的绝对路径)```为录制文件保存目录，**这三个占位符本身需要修改为具体的值，而不是修改```:```后面那部分，其他的不用修改**, 下面是修改后的命令示例:

```
sudo docker run -it --privileged=true -e DOCKER_IP=46.8.147.195 --name wfc_janus_server --net host -v /Users/userName/wf-janus/config:/var/janus/janus/etc/janus -v /Users/userName/records_folder:/opt/janus/share/janus/recordings wildfire_janus
```

上线时需要deamon方式运行，且设置为自动重启。命令如下：
```
sudo docker run -d --privileged=true -e DOCKER_IP=YOUR_PUBLIC_IP --name wf_janus_server --net host -v PATH_TO_janus_config:/var/janus/janus/etc/janus -v PATH_TO_RECORDS_FOLDER:/opt/janus/share/janus/recordings --restart=always wildfire_janus
```

> --restart=always 一定要设置上，因为当出现不可逆错误时，janus服务会退出，这样自动重启就能恢复。
> --net host 参数不能遗忘，janus服务需要跟外网通讯，要么host网络，要么做端口映射。建议host网络性能更好更容易。
> docker 挂载主机目录，请参考 [docker run -v](https://www.cnblogs.com/starfish29/p/10653960.html), [docker volums](https://docs.docker.com/storage/volumes/)

## 客户端
确保客户端能够正常运行，能够收发消息；替换音视频高级版的SDK；注意高级版是不需要stun/turn服务，去除掉turn服务的配置。

## 双流模式
双流模式下，客户端会发出去两路视频流，一路是标准流，另外一路是微流。标准流的参数可以通过设置profile接口来更改，微流的流量大小固定为10KB/s。客户端SDK会自动根据view的大小，设置其中最大的一个窗口为标准流，其他都为微流。这样就节省大量的带宽资源。如果不需要双流模式，可以关闭双流模式，客户端只发布和拉取标准流。

## 手动控制视频流
如果主播视频超过一屏，没有显示的主播拉取视频流就是浪费资源的，可以手动控制每个主播的视频流。SDK有个属性```autoSwitchVideoType```，当值为false时，可以通过```setParticipantVideoType```方法来控制主播```不拉取视频流```、```拉取视频大流```和```拉取视频小流```。

## 视频处理
美颜、虚化背景等功能需要处理视频流，SDK有回调，应用实现回调，可以处理视频Frame。

## 通话容量计算
野火普通模式和双流模式的容量计算是不同的。

普通模式下，对于N个参与者来说，每个人都会发送一路上行数据，所以上行路数就是参与者的个数，每个人都接受其他每个人的下行数据，这样下行路数就是N x (N-1)路。

双流模式下，对于N个参与者来说，每个人都会发送一路标准流和一路微，所以上行就是N路标准流，N路微流，每个人都接受一路标准流和N-2路微流，这样下行流量就是N路标准流 + N*(N-2)路微流。

默认情况下一路标准流的带宽需要100KB/s（可以配置），微流是10KB/s。所以普通模式下每个人上行是100KB/s，服务器的上行总流量为N * 100KB/s；双流模式下每个人是110KB/s，服务器的上行总流量为N * 110KB/s。下行时标准模式，每个人的流量为（N - 1） * 100KB/s，服务器总的下行流量为 N * （N - 1） * 100KB/s；双流模式下每个人的流量为 100KB/s +  (N - 2) * 10KB/s，服务器总的下行流量为 N * 100KB/s + N * (N - 2) * 10KB/s。

以9人通话为例子，标准模式下，每人上行100KB/s，下行 8 * 100 = 800KB/s，服务器总上行 9 * 100 = 900KB/s，服务器总下行为 7200KB/s。在双流模式下，每人上行110KB/s，下行 100 + 7 * 10 = 170KB/s，服务器的总上行 9 * 110 = 990KB/s，服务器的总下行为 1530KB/s。可以看出9人会议，双流模式流量上行比普通模式多消耗10%，但下行双流模式比普通模式少接近80%的流量。

野火音视频高级版可以支持会议模式，也就是有N个主播（发布自己的数据和接收其它主播的数据）和M个听众（只接收主播的数据)，同理可以计算对应流量。

如果更改过视频profile，可以实际测量每一路流量大小，从而计算整体容量大小（双流模式下，微流固定是10KB/s，上行数减去微流大小就是标准流的带宽)。

## 水平扩展
通过容量计算方法可以看出，受限于服务器的带宽，单台只能支持有限的通话容量。可以部署多台媒体服务来水平扩展媒体服务。当用户发起音视频通话时，IM服务会Hash分配会议到某台媒体服务。因此可以看出总的容量可以随着媒体服务器的增加而增加，但单个会议的最大容量还是受限于单台服务器的最大带宽。使用超级会议模式时，会议可以扩展到整个系统中，因此当发起大用户数量的视频会议直播时可以选择超级会议模式。

扩展时需要确保每台服务的```client_id```不能重复，请同步修改janus服务的配置文件和IM服务的配置文件，确保janus配置中```client_id```是唯一的，且在IM配置文件中janus ```client_id```列表中。

## 录制文件的处理
Janus支持服务器端录制，在IM服务的配置文件关于janus配置部分有如下配置：
```
#音视频服务录制策略。0 不允许录制，1 总是录制，2 客户端选择是否录制。
conference.record_strategy 1
```
当策略为0时，所有实时音视频都不录制；当策略为1时，所有实时音视频都录制；当策略为2时，客户端发起会议有个是否录制参数，决定是否录制。不支持实时音视频发起后再更改是否录制。

当录制完成后，会保存到janus启动的docker命令指定的目录。录制文件的格式为```mjr```，这种格式是直接把每个流的RTP内容写入文件，这样就不会对服务器造成任何的计算压力。录制的文件是每个用户的每个流独立存储，文件名格式如下：
```
videoroom-${roomId}-user-${userId}-${timestamp}-[audio/video]-[0-2].mjr
```
roomId也就是会议id，如果是客户端生成格式是发起者的ID加上```|```加上随机数字，也可以应用层生产会议id发起时填入；userId为此媒体流所属的用户id；timestamp为此媒体流开始的时间；```[audio/video]``` 此媒体类是语音流还是视频流；```[0-2]```媒体类的id，一般音频是```0```，视频大流是```1```，视频小流是```2```。

以9人视频会议为例，每人会录制3个文件，分别是音频文件、视频大流文件和视频小流文件，一共录制27个文件。

janus只能支持这种处理方式，不能录制为常见格式也不能自动合流。因此需要进行后期处理才能够回放，下载[janus-pp-rec](./janus-pp-rec)，然后执行：
```
./janus-pp-rec  videoroom-${roomId}-user-${userId}-${timestamp}-audio-0.mjr 1.opus
./janus-pp-rec  videoroom-${roomId}-user-${userId}-${timestamp}-video-1.mjr 1.mp4
```
再使用ffmepg把音频和视频结合成一个文件:
```
ffmpeg -i 1.mp4 -i 1.opus -c:v copy -c:a aac call.mp4
```
至此已经为一个用户合成了多媒体文件。回放还需要把多个用户的多媒体合流成一个多媒体文件（ffmpeg可以做到，具体命令请网上搜索，注意起始时间同步），或者播放时播放多个文件。

详情请参考[janus录制](https://janus.conf.meetecho.com/docs/recordings.html)

## 媒体流转发（RTP_FORWARD）
RTP_FORWARD是[janus](https://janus.conf.meetecho.com/docs/videoroom.html)一个有意思的功能，可以把音视频的RTP流实时的转发到指定的UDP端口上去。这样当需要把会议内容实时推流到直播平台时，就可以使用这种方法。请注意这里janus只是把RTP流原封不动的转发出来，还需要不小的工作量做来把音视频转换后再推送到直播平台。下面讲一下如果使用媒体流转发功能：

### 开启转发
在server sdk中的ConferenceAdmin对象中，有个listParticipants方法，可以查询到用户的发布媒体流情况，可以得到媒体类型（音频还是视频）、媒体ID、编码（音频为opus，视频为H264/VP8）。当知道这些信息后，就可以调用rtpForward方法把音视频流转到指定地址去
```
static IMResult<Void> rtpForward(String roomId, String userId, String rtpHost, int audioPort, int audioPt, long audioSSRC, int videoPort, int videoPt, long videoSSRC)
```
方法中，roomId为房间ID，也就是客户端看到的callId或者会议ID；userId为您要转发用户的ID；rtpHost只接收地址。音频和视频要分开转发，需要不同的端口，如果不需要某个流，可以把对应的port赋为0就行。audioPt一般为111，videoPt在H264编码时为98，VP8时为100（实际上其他数字也可以，在接收端需要把PT对应起来）。SSRC默认可以为0.

当开启成功后，指定用户的RTP流就会持续的转发过去。可以使用gstreamer来处理转发过来的流，比如下面会播放收到的视频流
```
gst-launch-1.0 -v udpsrc port=10005 caps = "application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)98" ! rtph264depay ! decodebin ! videoconvert ! autovideosink sync=false
```

### 查询转发
在server sdk中的ConferenceAdmin对象中，有个listRtpForwarders方法，可以查询到已经开启的媒体流转发状态。查询的每个流中都有个streamId，停止媒体转发需要使用到此参数，所以一般停止之前都需要先查询一下。

### 停止转发
当不需要时，可以停止转发。在server sdk中的ConferenceAdmin对象中，有个stopForwarders方法，可以查询到已经开启的媒体流转发状态。注意此参数需要对每个流调用一次，比如某个用户同时转发了音频流和视频流，需要调用2次来停止。

## 问题排查
1. 音视频依赖IM作为信令通道的，首先确保通话双方能够互相发送文本消息。
2. Janus服务器公网IP配置不对，也就是启动命令中的```DOCKER_IP```参数。一定要使用***Janus服务器的公网IP地址***，不能是域名也不能是内网IP，也不能配置为IM服务的地址（分开部署的情况下）。
3. 没有配置```ice_enforce_list```或者配置错误。请指定为绑定公网IP的网卡。
4. Janus服务挂掉。当出现严重错误或者IM服务节点变动时，Janus会自动退出，这时需要在docker启动命令中添加```--restart=always```参数，让janus服务自动重启。
5. Janus没有配置好网络。janus服务需要跟外网通讯，需要启动命令中添加参数```--net host```。
5. 客户端到服务器的连通问题，这时需要检查云服务器的安全组和防火墙是否开放了对应的UDP端口。确认过安全组和防火墙后如果还是无法正常使用，请再按照下面说明检查端口是否是通的。
6. 移动客户端音视频SDK授权域名不对，请检查客户端配置文件中的***IM host***是不是跟音视频SDK绑定的地址一致。
7. 如果上述检查还是无法解决问题，请抓取IM服务日志，janus服务日志和客户端日志发送给support@wildfirechat.cn，其中IM服务日志需要改成同步写，请参考[IM服务常见问题](https://docs.wildfirechat.cn/faq/server.html)中的问题45。

### UDP端口连通性检查
Janus服务处于公网，客户端无论处于任何NAT之内都应该可以连接。当出现连接超时的错误时，很有可能是Janus与客户端之间UDP端口无法互通。可以用[netcat](https://www.baidu.com/s?wd=netcat)来检查他们之间的连通性。
1. 环境准备: 需要在客户端网络之内准备一台linux或者mac作为测试客户端(不能用同一个机房的服务器，如果本地没有linux或者mac，可以去外网外地购买一台按小时计费的最低配云服务器，一天不到2元，测试完释放花不了多少钱)；在测试客户端和janus服务器上分别安装```netcat```，已知Ubuntu和mac已经预安装了，centos可以用命令```yum install -y nc```来安装。其它系统可以百度怎么安装。
2. 在Janus服务上，执行命令 ```nc -ulvp 30000```。30000为监听UDP端口，需要注意Janus服务配置的端口范围之内。
3. 在客户端上执行命令 ```nc -u YOUR_PUBLIC_IP 30000```。```YOUR_PUBLIC_IP```是Janus服务的公网IP，也是启动命令内的参数。
4. 在客户端输入内容，检查服务器端是否收到对应内容。
5. 服务器端收到后，在服务器端窗口输入内容，检查客户端是否收到对应内容。

如果4、5失败说明网络之间不通，需要运维检查网络环境。如果成功，客户端需要断开，再重试几次。再更换服务器端端口，重复测试几次。UDP检查方案来源于[这里](https://docs.azure.cn/zh-cn/articles/azure-operations-guide/virtual-network/aog-virtual-network-using-netcat-check-the-connectivity#测试-udp-端口连通性)。

## ffmpeg安装
录制音视频转换工具[janus-pp-rec](./janus-pp-rec)需要依赖ffmpeg库3.X，一般通过命令直接安装即可。如果无法直接安装，也可以下载源码编译安装：
```
wget https://github.com/FFmpeg/FFmpeg/releases/download/n3.0/ffmpeg-3.0.tar.gz
tar -xzvf ffmpeg-3.0.tar.gz
cd ffmpeg-3.0
./configure --prefix=/usr --enable-shared
make
sudo make install
```

## 服务兼容
服务在2021.8.4日和2022.7.14日有重大升级，需要确保janus服务、IM服务和客户端SDK同时使用这个日期之前的版本或者之后的版本。如果是新的janus服务和IM服务，旧的客户端SDK，可能会有部分旧型号的手机无法接通，已知影响手机为iphone8及之前型号，Android还没有发现有不能接通问题。请升级时注意janus服务和IM服务同时升级，客户端也要尽量升级。

## 关于野火音视频高级版的一些基础知识
如果需要更好地二次开发和使用，请详细阅读[野火音视频高级版的一些基础知识](https://docs.wildfirechat.cn/blogs/野火音视频高级版的一些知识.html)，确保您对野火音视频高级版有一定的了解。

## 鸣谢
本产品基于[janus](https://github.com/meetecho/janus-gateway)二次开发而来，十分感谢他们的奉献！
