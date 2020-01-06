# 第二章 安装Docker
&emsp;&emsp;目前，Docker已经支持非常多的Linux平台。  
&emsp;&emsp;目前来讲，Docker团队推荐使用Ubuntu或RHEL宿主机中部署Docker，这两个发行版中直接提供了可安装的软件包。

## 2.1 安装Docker的先决条件
&emsp;&emsp;Docker要求的条件具体如下。
* 运行64位CPU架构的计算机（目前只能是x86_64和amd64），请注意，Docker目前不支持32位CPU。
* 运行Linux3.8或更高版本内核。
* 内核必须支持一种适合的存储驱动（storage driver）例如：  
    - Device Manager
    - AUFS
    - vfs
    - btrfs
    - 默认存储驱动通常是Device Mapper
* 内核必须支持并开启cgroup和命名空间功能。

## 2.2 在Ubuntu中安装Docker
### 2.2.1 检查前提条件
#### 1. 内核
&emsp;&emsp;$uname -r 查看内核版本要在Linux3.8及以上

#### 2. 检查Device Mapper
&emsp;&emsp;可以使用$sudo grep device-mapper /proc/devices 检查是否有device-mapper。

&emsp;&emsp;cgroup和命名空间自2.6版本开始已经集成在Linux内核中了。

### 2.2.2 远程安装Docker
&emsp;&emsp;我们将使用Docker团队提供的DEB软件包来安装Docker。首先，要添加Docker的**APT**仓库，期间可能会提示我们确认添加仓库并自动将仓库的**GPG**添加到宿主机中。

1. 安装之前需要检查是否安装了**curl**，如果没有需要安装
    - 使用$which curl 查看是否安装。
    - 使用$sudo apt-get -y install curl 安装。
2. 添加 Docker 的**ATP**仓库 $sudo sh -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
3. 添加Docker仓库的GPG密钥 $curl -s https://get.docker.io/gpg | sudo apt-key add 

### 离线安装
1. 下载containerd.io_1.2.4-1_amd64.deb、docker-ce-cli_19.03.1_3-0_ubuntu-bionic_amd64.deb、docker-ce_19.03.1_3-0_ubuntu-xenial_amd64.deb三个文件。
2. 下载好上面三个文件后，通过**dpkg -i** 命令安装这三个软件。
3. 然后使用dpkg -l 命令与grep搭配查看是否已经安装（ii）例如：$dpkg -l | grep container  
4. 然后使用$sudo docker info 查看docker的信息
### 2.2.3 Docker与UFW
&emsp;&emsp;在Ubuntu中，如果使用UFW（Unconplicateed Firewall 不复杂的防火墙）还需做点改动才能让Docker工作。Docker使用网桥管理容器中的网络。默认情况下，UFW会丢弃所有数据包。因此需要在UFW中启用数据包的转发。需要对/etc/default/ufw文件做一些改动。将 DEFAULT_FORWARD_POLICY="DROP"改为： DEFAULT_FORWARD_POLICY="ACCEPT"  

## 2.9 Docker守护进程
&emsp;&emsp;安装完Docker后，需要确认Docker的守护进程是否运行。Docker以Root权限运行它的守护进程，来处理普通用户无法完成的操作。**docker**程序是Docker守护进程的客户端程序，同样也需要以root身份运行。

&emsp;&emsp;开机自启动docker$sudo systemctl enable docker $sudo systemctl start docker

### 2.9.1 配置Docker守护进程
&emsp;&emsp;我们可以使用-H 标志指定不同的网络接口和端口配置例如:  $sudo docker -d -H tcp://0.0.0.0:9190 这条命令将**Docker守护进程**绑定到宿主机上的所欲网络地址。Docker客户端不会自动检测到网络的变化，我们需要通过-H选项来指定服务器的地址。 还可以通过设置DOCKER_HOST唤环境变量来省略此步骤。例如：export DOCKER_HOST="tcp://0.0.0.0:9190"  

&emsp;&emsp;当然，也可以同时指定多个绑定地址。

&emsp;&emsp;在启动守护进程时，我们还可以通过在命令前指定DEBUG=1 参数来输出更详细的信息。例如：$DEBUG=1 /usr/bin/docker -d  
&emsp;&emsp;要想让改动永久生效，需要编辑启动配置项，在Ubuntu中需要编辑/etc/defalut/docker 文件，并修改DOCKER_OPTS 变量。

### 2.9.2 检查Docker守护进程是否正在运行
&emsp;&emsp;使用ps命令即可，例如：$ps -ef | grep dockerd | grep -v grep