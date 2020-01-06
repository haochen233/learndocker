## 4.1 什么是Docker镜像
&emsp;&emsp;Docker镜像是由文件系统叠加而成。最底端是一个引导文件系统，即**bootfs**，Docker镜像的第二层是root文件系统**rootfs**，并且Docker利用联合加载（union mount）技术又在root文件系统层上加载更多的只读文件系统。联合加载指的是同时加载多个文件系统，但在外面看起来只能看到一个文件系统。联合加载会将各层文件系统叠加到一起，这样最终的文件系统会包含所有底层的文件和目录。  

&emsp;&emsp;Docker将这样的文件系统称为镜像。一个镜像可以放到另一个镜像的顶部，位于下面的镜像称为父镜像（parent image），可以以此类推，直到镜像栈的最底部，最底部的镜像称为基础镜像（base image）。最后，当从一个镜像启动容器时，Docker会在给镜像的最顶层加载一个读写文件系统。我们想在Docker中运行的程序就是在这个读写层中执行的。

&emsp;&emsp;当Docker第一次启动一个容器时，初始的读写层是空的。当文件系统发生变化时，这些变化都会应用到这一层上。比如，向修改一个文件，这个文件首先会从读写层下面的只读层被复制到读写层。该文件的只读版本依然存在，但是已经被读写层中的该文件副本（刚复制过来的）所隐藏。这种机制被称为写时复制。（因为需要对一个文件进行写操作即改动时，就会从只读层拷贝过来一份进行修改，实际只读层的并未改动且被隐藏了）。
&emsp;&emsp;这个技术是使Docker如果强大的技术之一。每个只读层都只能读，并且永远不会变化。
![alt 镜像理解](https://github.com/haochen233/picture/blob/master/Docker%E9%95%9C%E5%83%8F%E7%90%86%E8%A7%A3.png "理解")
&emsp;&emsp;容器是可以修改的，它们都有自己的状态。结合上图，容器的这种特点加上镜像分层框架（image layering framework），可以使我们快速构建镜像并且运行包含我们自己的应用程序和服务的容器。

## 4.2 列出镜像
&emsp;&emsp;可以使用`docker images`命令来列出Docker主机上可用的镜像。  
&emsp;&emsp;镜像从仓库（repository）下载下来。镜像保存在仓库中，而仓库保存在Registry中。默认的Registry是由Docker公司运营的公共Registry服务，即Docker Hub。  
&emsp;&emsp;每个仓库都可以存放很多镜像。  
&emsp;&emsp;使用`docker pull`命令来拉取某仓库中的所有内容  
&emsp;&emsp;例如：  
`$sudo docker pull ubuntu`  
&emsp;&emsp;该命令会拉取ubuntu仓库中的所有内容。  
&emsp;&emsp;为了区分同一个仓库的不同镜像，Docker提供了一种称为标签（tag）的功能。每个镜像在列出来时都带有一个标签，例如ubuntu仓库中的镜像有：12.04、12.10等等，每个标签对组成特定镜像的一些镜像层进行标记（例如12.04就是对所有Ubuntu 12.04进行的层的标记）。这种机制使得同一个仓库可以存储多个镜像。  
&emsp;&emsp;我们当然也可以根据仓库名加上冒号:和标签名，来指定该仓库中的某一镜像。  
&emsp;&emsp;例如：  
`$sudo docker run -i -t --name test ubuntu:12.04 /bin/bash`  
&emsp;&emsp;这个例子会从镜像ubuntu:12.04启动一个容器。  
&emsp;&emsp;Docker Hub中有两种仓库：用户仓库（user repository）和顶层仓库（top-level repository）。用户仓库是由Docker用户创建的，而顶层仓库则是由Docker内部人来管理的。  
&emsp;&emsp;用户仓库的命名有用户名和仓库名组成：如haochen/testrepo  
+ 用户名：haochen
+ 仓库名：testrepo
&emsp;&emsp;顶级仓库中的镜像是架构良好、安全且最新的。  

## 4.3 拉取镜像
&emsp;&emsp;用`docker run`命令启动一个容器时，如果镜像不在本地，Docker会从Docker Hub上下载该镜像。如果没有指定具体镜像标签，则会自动下latest标签的镜像。  
&emsp;&emsp;可以使用`docker pull`拉取镜像，还可以用该命令拉取指定标签的镜像  
&emsp;&emsp;例如：`docker pull fedora:20` 那么该命令只会拉取fedora:20镜像  

## 4.4 查找镜像
&emsp;&emsp;`docker search`命令可以查找所有Docker Hub上公共的可用镜像。然后就能够使用`docker pull`来进行拉取。

## 4.5 构建镜像
&emsp;&emsp;构建Docker镜像有两种办法：  
+ 使用`docker commit`命令
+ 使用`docker build`命令和`Dockerfile`文件  
&emsp;&emsp;现在并不推荐使用`docker commit`命令，应该使用更灵活、强大的`Dockerfile`来构建Docker镜像。  
