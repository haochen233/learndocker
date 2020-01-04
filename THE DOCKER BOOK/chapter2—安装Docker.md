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

### 2.2.2 安装Docker
&emsp;&emsp;我们将使用Docker团队提供的DEB软件包来安装Docker。首先，要添加Docker的**APT**仓库，期间可能会提示我们确认添加仓库并自动将仓库的**GPG**添加到宿主机中。

1. 安装之前需要检查是否安装了**curl**，如果没有需要安装
    - 使用$which curl 查看是否安装。
    - 使用$sudo apt-get -y install curl 安装。
2. 添加 Docker 的**ATP**仓库 $sudo sh -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
3. 添加Docker仓库的GPG密钥 $curl -s https://get.docker.io/gpg | sudo apt-key add 

## Docker守护进程
&emsp;&emsp;安装完Docker后，需要确认Docker的守护进程是否运行。Docker以Root权限运行它的守护进程，来处理普通用户无法完成的操作。**docker**程序是Docker守护进程的客户端程序，同样也需要以root身份运行。

&emsp;&emsp;启动docker$sudo systemctl enable docker $sudo systemctl start docker

