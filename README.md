# GofreeVPN
如何快速搭建一个属于自己的VPN系统科学上网翻墙

# 一、科学上网

通过科学上网，意思就是通过VPN技术，绕过一些网络限制，访问到常用的一些外网网站，比如通过谷歌查询一些技术文献就比较方便了。

# 二、怎么上VPN

使用VPN的方法很多，归纳起来大致有：（1）自建VPN；（2）付费第三方软硬件；（3）免费第三方软件。
| 方式 | 要怎么做 | 花费情况 | 稳定性 |
| ------ | ------ | ------ | ------ |
| 自建VPN | 购买VPS（云服务器）| 按月/季/年付费 | 比较稳定（自建隧道） |
| 付费第三方软件 | 购买软件 | 软件按月/季/年付费 | 经常可能被封，推荐购买大厂产品 |
| 付费第三方硬件 | 购买某些路由器设备 | 硬件费用 + 软件的月/季/年付费 | 投入较大，需要一定技术才能折腾<br>  支持openwrt的路由器，有一定方便性，在同一个wifi下不需要每个终端都去安装软件 |
| 免费第三方软件 | 找免费软件 | 免费 | 一些免费机场 |
<br>

- **这里提供一个免费并简易VPN入口**<br>
![images](https://github.githubassets.com/images/icons/emoji/unicode/2708.png)<br>
[NEECHIGO](https://neechigo.com)<br>
目前只有windows客户端，测试了一段时间，还比较稳定，傻瓜式软件并完全免费。<br>
但软件只开放了指定的一些网站可通过VPN访问。google.com是全域名可用。

# 三、自建VPN

这里主要讨论一下如何自建VPN，下文的方法相对比较简单，通过第三方容器的方式，对技术要求低一些：<br>
## 1.购买VPS（云服务器）<br>
  i. 不需要技术背景，只要会花钱购买就行了
  VPS（云服务器）很多，选择VPS主要从性价比出发考虑。<br>
  如果预算够，最简单的方法就是直接买一台国内云服务商的海外云主机（windows系统），那就啥都解决了。<br>
  <br>
  ii. 要一点英文基础，知道怎么下单购买VPS；要一点技术基础，知道怎么follow下面的方法部署环境。<br>
  如果在乎预算，那就好好甄别一下该怎么选VPS，这里给出一个建议，尽量去选国外服务商提供的VPS，<br>
  流程：在服务商网站注册账户，下单购买VPS，开始部署。<br>
  <br>
  iii. 国外VPS可以去搜索一下，很多很多。价格有高有低（一般几个美金一个月），可以自行上网多搜搜多试试，有些网站30天可以退款。有些VPS可以找到promot code，付款方式一般是支持Paypal，有些支持支付宝，底下列举一些VPS网站，有些访问不受限，有些可能需要先搭梯子才能访问到：<br>
[Bluehost](https://www.bluehost.com/)<br>
[HostGator](https://www.hostgator.com/)<br>
[Hostinger](https://www.hostinger.com/)<br>
[Namecheap](https://www.namecheap.com/)<br>
[BandwagonHost](https://bandwagonhost.com/)<br>
  
## 2.搭建环境
### 2.1安装wireguard客户端<br>
安装并运行客户端（本仓库里面放了一个wireguard的windows客户端安装包，可自行下载），点击“新建空隧道”，<br>
![images](https://iili.io/HdPpmve.png)<br>
弹开“创建新隧道”窗口<br>
![images](https://iili.io/Hdi3jSV.png)<br>
创建一个txt文本文件，拷贝以下内容到文本文件中备用（后续将要在VPS终端中输入这段字符）：<br>
```
docker run \
  -it \
  --rm \
  --cap-add NET_ADMIN \
  --device /dev/net/tun:/dev/net/tun \
  -v wg-access-server-data:/data \
  -e "WG_ADMIN_PASSWORD=admin" \
  -e "WG_WIREGUARD_PRIVATE_KEY=UG87dYLqCuz8ZeGXvkZIWnFwHmJ+X5frhQfjprayk0g=" \
  -p 8000:8000/tcp \
  -p 51820:51820/udp \
  place1/wg-access-server &
```
在文本中需要改变两个字段的内容：<br>
- “WG_ADMIN_PASSWORD=”后面的内容是自定义的（后续wireguard的web登录界面的用户名和口令）<br>
- “WG_WIREGUARD_PRIVATE_KEY=”后面的内容就是前面步骤中wireguard客户端中拷贝出来的私钥<br>
**注意拷贝私钥的时候，前后都不要多出空格，拷贝完成后，即可关闭“创建新隧道”窗口**<br>

### 2.2VPS安装docker<br>
VPS购买**centos 7**镜像, SSH登录(可以使用secureCRT、Xshell各种各样的SSH软件）VPS的root终端：<br>
#### 2.2.1安装docker<br>
在root终端中，直接拷贝以下整一段直接回车执行（安装过程大概两分钟）：<br>
```
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-nightly
yum-config-manager --enable docker-ce-test
yum install -y docker-ce docker-ce-cli containerd.io
yum install -y libseccomp-devel 
systemctl start docker
 
```
#### 2.2.2获取并运行wg-access-server<br>
输入以下整一段直接回车执行（*即前面文本文件编辑的那段字符*）：<br>
```
docker run \
  -it \
  --rm \
  --cap-add NET_ADMIN \
  --device /dev/net/tun:/dev/net/tun \
  -v wg-access-server-data:/data \
  -e "WG_ADMIN_PASSWORD=admin" \
  -e "WG_WIREGUARD_PRIVATE_KEY=UG87dYLqCuz8ZeGXvkZIWnFwHmJ+X5frhQfjprayk0g=" \
  -p 8000:8000/tcp \
  -p 51820:51820/udp \
  place1/wg-access-server &
```
![images](https://iili.io/HdiXAcg.png)<br>
**看到上图中红框内容表明镜像拉取成功，否则重复尝试上面的命令。**<br>

### 2.3登录wireguard后台管理界面<br>
登录wireguard的管理界面，获取隧道配置文件<br>
浏览器输入：http://IP:8000  （IP就是购买的VPS的公网IP地址）<br>
![images](https://iili.io/HdiOQFS.png)<br>
输入用户名、口令，就是前面文本中自定义的字符串<br>
![images](https://iili.io/HdiktTu.png)<br>
登录成功后，随意输入一个网关名字，并点击增加按钮：<br>
![images](https://iili.io/Hdi8zVj.png)<br>
点击“CONNECTION FILE”下载wireguard配置文件到本机计算机，后续需要将这个配置文件导入wireguard客户端中。<br>
![images](https://iili.io/HdiSndN.png)<br>

### 2.4 登录wireguard客户端
在客户端界面上，点击“新建隧道”，选择“从文件导入隧道”，选择刚下载的配置文件wireguard.conf文件。<br>
![images](https://iili.io/HdiDKAJ.png)<br>
最后点击“连接”，然后你就获得了一条属于自己的VPN线路:)<br>
![images](https://iili.io/HdibA0P.png)<br>
