## 3.1 确保Docker已经就绪
&emsp;&emsp;$sudo docker info 该命令会返回所有容器的和镜像的数量、一级一些其他信息。
## 3.2 运行我们的第一个容器
&emsp;&emsp;让我们尝试启动第一个Docker容器，使用**docker run**命令创建容器，例如：  
$sudo docker run -i -t ubuntu /bin/bash  
&emsp;&emsp;下面来解读这行命令，首先是我们告诉Docker执行docker run命令，并指定了-**i**和-**t**两个命令行参数。**-i**标志保证容器中STDIN是开启的，尽管我们并没有附着到容器中，持久的标准输入时交互式的“半边天”。**-t**标志则是另外“半边天”，它告诉Docker为要创建的容器分配一个**伪tty终端**，这样新建的容器才能提供一个**交互式shell**。若要在命令行下创建一个我们能与之**进行交互的容器**，而不是一个运行在后台服务的容器，这两个参数已经是最基本的参数了。  
&emsp;&emsp;然后是ubuntu这个参数，这是我们告诉Docker基于什么镜像来创建容器，则使用ubuntu镜像。在这背后发生了什么呢？，首先Docker会检查本地是否存在ubuntu镜像，本地没有的话，就会连接官方维护的Docker Hub Registry。在Docker Hub中查找，一旦找到就会下载该镜像到并保存到本地宿主机中。 
&emsp;&emsp;随后Docker用这个镜像创建了一个新容器。该容器拥有自己的网络、IP地址，以及一个用来和宿主机进行通信的桥接网络接口。最后的/bin/bash告诉Docker在新容器中运行什么命令，即我们在容器中运行/bin/bash 命令启动了一个Bash shell。  
&emsp;&emsp;当容器创建完毕后，Docker执行/bin/bash命令，然后就可以看到容器内的shell了。 

## 3.3 使用第一个容器
&emsp;&emsp;登录到了新容器里面显示效果为：**root@0248c3a4ed57:/#**容器的ID为**0248c3a4ed57**  
&emsp;&emsp;首先，可以获取该容器的主机名，使用**hostname**命令,容器的主机名就是ID。再来看看/etc/hosts文件。  
&emsp;&emsp;试着在容器中安装软件包,例如安装一个vim软件**apt-get install vim**。  
&emsp;&emsp;然后可以继续在容器做其他事情，当所有工作都结束时，输入exit就可以返回到Ubuntu宿主机的命令行提示符了。  
&emsp;&emsp;那个容器怎么样了？容器已经停止运行了！只有在指定的/bin/bash 命令处于运行状态时，容器也才会处于运行状态。一旦退出容器，/bin/bash命令也就结束了，容器也就停止了运行。  
&emsp;&emsp;但容器仍然是存在的，使用$docker ps -a命令查看系统中容器的列表。默认情况下，当执行**docker ps**命令时，只能看到正在运行的容器。如果指定-a选项，那么就会列出所有容器，不论停止的还是正在运行的。

## 3.4容器命名
&emsp;&emsp;Docker会为我们创建的每一个容器自动生成一个随机的名称可以用。可以使用 --name选项指定容器的名称。  
例如：$docker run --name test1 -i -t ubuntu /bin/bash 创建了一个名为test1的容器。容器的名称是唯一的。使用好的容器名称能够更好的管理容器。  
&emsp;&emsp;用docker rm命令删除已有的容器。

## 3.5 重新启动已经停止的容器
&emsp;&emsp;可以用**docker start**启动一个已经停止的容器。 例如：$sudo docker start testctr 注：testctr是容器名。  
除了使用容器名，还可以使用容器ID来重新启动已停止的容器。还能使用**docker restart**命令来启动一个已经停止的容器。

## 3.6 附着到容器上
&emsp;&emsp;Docker容器重新启动的时候，会沿用docker run 命令时指定的参数来运行。需要使用docker attach命令，重新附着到该容器的会话上（回到Bash提示符）。同样可以使用容器ID。

## 3.7 创建守护式容器
&emsp;&emsp;除了这些交互式运行的容器，也可以创建长期运行的容器。守护式容器，没有交互式会话，非常适合运行应用程序和服务。大多数时候我们都需要以守护式来运行我们的容器。

例如： 

    $sudo docker run --name daemon_hello -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
&emsp;&emsp;-d 参数Docker会让这个容器放到后台运行。上面的命令只会返回一个容器ID。使用docker ps 只能看到一个运行中的程序。  

## 3.8 容器内部都在干什么
&emsp;&emsp;现在我们已经在后台运行了一个while循环的守护型容器。为了知道该容器内部都干了些什么，可以使用**docker logs**命令来获取容器的日志。  
例如：
    $sudo docker logs daemon_hello
    可以使用-t选项为每条日志加上时间戳
    可以使用-f选项持续监视

## 3.9 查看容器内的进程   
&emsp;&emsp;使用**docker top**命令查看容器内部运行的进程

## 3.10 在容器内部运行进程
&emsp;&emsp;可以使用**docker exec**命令在容器内部额外启动新进程。可以在容器内运行的进程有两种类型：后台任务和交互式任务，后台任务在容器内运行且没有交互需求，而交互式任务则保持在前台运行。  
例如：
    
    在容器daemon内部运行一个后台进程需要使用-d选项
    $sudo docker exec -d daemon_hello touch /root/f1.txt
    还可以在daemon_hello容器中启动一个诸如打开shell的交互式任务
    $sudo docker exec -i -t daemon_hello /bin/bash
&emsp;&emsp;和运行交互式容器一样，这里的-i和-t选项为我们执行的进程创建了tty并捕捉STDIN。接着指定了容器和要执行的命令

## 3.11 停止守护式容器
&emsp;&emsp;要停止守护式容器，只需要执行**docker stop**命令。同样可以使用容器ID。  
例如： 
```shell
$sudo docker stop daemon_hello
```
实质上docker stop命令会向Docker容器发送**SIGTERM**信号，如果想快速终止某个容器，还可以使用docker kill命令来向容器进程发送**SIGKILL**信号

## 3.12 自动重启容器
&emsp;&emsp;如果由于某种错误而导致容器停止运行，我们还可以通过--restart选项，让Docker自动重启该容器。  
例如:  
```shell
$sudo docker run --restart=always --name test ubuntu /bin/bash
```
&emsp;&emsp;上例中--restart 标志被设置为always。无论容器的退出码是什么，Docker都会自动重启该容器。除了always，我们还可以设置为**on-failure**，这样只有当容器的退出码为非零时，才会自动重启。另外**on-failure**还能接受一个可选的重启次数参数。  
例如：  
```shell
--restart=on-failure:5
```
&emsp;&emsp;在上例中当容器的退出码不为0时，最多会自动重启5次。

## 3.13 深入容器
&emsp;&emsp;除了使用docker ps命令获取容器的信息，还可以使用**docker inspect**获取更详细的容器信息。

## 3.14删除容器
&emsp;&emsp;如果容器不在使用，可以使用**docker rm**命令来删除它们。需要注意的是：运行中的容器是不能删除的，必须停止容器，然后才能删除。

&emsp;&emsp;删除所有容器，首先可以使用**docker ps -a -q**命令列出所有容器的ID，然后使用docker rm删除就可以了。
例如 ：  
```shell
$docker rm `docker ps -a -q`
```
