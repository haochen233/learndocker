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

### 4.5.1 创建Docker Hub镜像
&emsp;&emsp;构建镜像中很重要的一环就是如何共享和发布镜像，可以将镜像发布到Docker Hub或者用户自己的私有Registry中。为了完成这项工作，需要创建一个Docker的账号。  

&emsp;&emsp;要登录到Docker Hub，需要使用`Docker login`命令。个人信息将会保存到$HOME/.dockercfg文件中。  

### 4.5.2 用Docker的commit命令创建镜像  
&emsp;&emsp;在我们新创建的容器中，安装Apache。我们会将这个容器作为一个Web服务器来运行。所以我们想把它的状态保存下来。这样就不用每次都创建一个新容器并再次在里面安装Apache了。为了完成此项工作，先需要从容器中（exit）退出，之后再运行docker commit命令。  
`docker commit gen_container apache2`在docker commit命令中，指定了要提交的修改过的容器的ID（还可以为名字），以及一个目标的镜像仓库和镜像名。docker commit提交的只是修改，类似于Git中的commit，这使得该更新非常轻量。  

还可以使用`docker images`命令来查看新创建的镜像。而且在提交时可以像Git提交一样用`-m`选项来添加提交说明。例如：  
`docker commit -m="only install Apache2" --author=="haochen.zhang" 容器ID haochen233/apache:Webserver` 只不过写法与Git中的写法略有不同。还可以指定镜像的用户名和仓库名，并为该镜像加了一个Webserver标签。  
&emsp;&emsp;然后使用`docker images`命令查看镜像。  

### 4.5.3 用Dockerfile构建镜像  
&emsp;&emsp;并不推荐使用docker commit的方法来构建镜像。相反，推荐使用被称为`Dockerfile`的定义文件和`docker build`命令来构建镜像。`Dockerfile`使用基本的易于DSL语法的指令来构建一个Docker镜像，之后使用`docker build`命令基于该`Dockerfile`中的指令构建一个新的镜像。  
#### 我们的第一个Dockerfile
&emsp;&emsp;现在让我们创建一个目录并在里面创建初始的Dockerfile。我们将创建一个包含简单Web服务器的Docker镜像。  
&emsp;&emsp;我们创建了名为**static_web**的目录用来保存Dockerfile，这个目录就是我们的构建环境（build environment），Docker则将此环境称为上下文（context）或者构建上下文（build context）。Docker会在构建镜像时将构建上下文和该上下文中的文件和目录上传到Docker守护进程。这样Docker守护进程就能直接访问你想在镜像中存储的任何代码、文件或者其他数据。  
&emsp;&emsp;作为开始我们还创建了一个空的Dockerfile，例如：  
```Dockerfile
#Version: 0.0.1
FROM ubuntu:latest
MAINTAINER haochen233 "252087791@qq.com"
RUN apt-get update
RUn apt-get install -y nginx
RUN echo 'Hi, I am in your container' \
    >/usr/share/nginx/html/index.html
EXPOSE 80
```
&emsp;&emsp;需要注意的是，要运行的命令是存放在一个数组结构汇中的。这将告诉Docker按指定的原样来运行该命令。当然也可以不使用数组指定CMD命令，这时候Docker会在指定的命令前加上/bin/sh -c。强烈推荐以数组语法来设置要执行的命令，例如：`RUN ["apt-get", "install", "-y", "nginx"]`

&emsp;&emsp;该Dockerfile有一系列指令参数组成。每条指令，如`FROM`，**都必须为大写字母，且后面要跟随一个参数。**Dockerfile中的指令会按照顺序从上到下执行，所以应该根据需要合理安排指令顺序。  
&emsp;&emsp;每条指令都会创建一个新的镜像层并对镜像镜像提交。Docker 大体上按照如下流程执行Dockerfile中的指令。  
- Docker从基础镜像运行一个容器。
- 执行一条指令，对容器做出修改。
- 执行类似`docker commit`的操作，提交一个新的镜像层。
- Docker再基于刚提交的镜像运行一个新容器。
- 执行Dockerfile中的下一条指令，知道所有指令都执行完毕。 
&emsp;&emsp;从上面可以看出，如果你的Dockerfile由于某些原因（如某条指令失败了）没有正常结束，那么你将得到一个可以使用的镜像。这对调试非常有帮助：可以基于该镜像运行一个具备交互功能的容器，使用最后创建的镜像对失败的指令进行调试。  
&emsp;&emsp;每个Dockerfile的第一条指令都应该是`FROM`。`FROM`指令指定一个已经存在的镜像，后续指令都将基于该镜像进行，这个镜像被称为基础镜像（base image）。  
&emsp;&emsp;在上面的例子中，我们指定了ubuntu:latest作为新镜像的基础镜像。接着指定了`MAINTAINER`指令，这条指令会告诉Docker该镜像的作者是谁，一级作者的电子邮箱地址。有助于表示镜像的所有者和联系方式。  
&emsp;&emsp;接下来是`RUN`指令，在上面我们指定了三条RUN指令。RUN指令会在当前镜像中运行指定的命令。每条RUN指令都会创建一个新的镜像层，如果指令执行成功，就会将此镜像层提交，之后继续依次执行Dockerfile中的下一条指令。  
&emsp;&emsp;接着设置了EXPOSE指令，这条指令告诉Docker该容器内的应用程序将会使用容器的指定端口，这并不以为着可以访问任意在容器运行的服务的端口。处于安全的原因，Docker并不会自动打开该端口，而是需要你在使用docker run运行容器时来指定需要打开哪些端口。  
&emsp;&emsp;可以指定多个EXPOSE指令来向外部公开多个端口。  
### 4.5.4 基于Dockerfile构建新镜像
&emsp;&emsp;执行docker build命令时，Dockerfile中的所有指令都会被执行并且提交，并且在该命令成功结束后返回一个新镜像。  
&emsp;&emsp;进入构建上下文目录中，然后使用docker build 命令构建新镜像，使用-t选项为新镜像生孩子了仓库和名称，也可以在构建过程中为镜像设置一个标签。例如：  
&emsp;&emsp;`docker build -t="docker_ubuntu/tree_ubuntu:useless_ubuntu" ./`  
&emsp;&emsp;如果在构建上下文的根目录下存在以`.dockerignore`命名的文件的话，那么该文件内容会被按行进行分割，每一行都是一条文件过滤匹配模式。这非常像`.gitignore`文件用来设置哪些文件不会被上传到构建上下文中去。该文件中模式的匹配规则采用了Go语言中的filepath。  
&emsp;&emsp;最后会输出最终镜像ID。

### 4.5.5指令失败时会怎么样
&emsp;&emsp;因为每执行成功一个指令都会生成**新的镜像ID**，所以假如哪一步指令出错，我们就可以基于最新成功的指令返回的镜像来创建一个容器，然后在容器中进行进一步操作，来确定问题出错在哪里，一旦该问题发现后，就可以退出容器，正确的**修改Dockerfile文件中的错误**，之后再尝试进行构建。 

### 4.5.6 Dockerfile和构建缓存
&emsp;&emsp;由于每一步指令执行成功都会进行提交，所以Docker的构建过程就显得非常聪明。他会将之前的镜像层看做缓存。想要略过缓存功能可以使用`docker build`的`--no-cache`标志。  

### 4.5.7 基于构建缓存的Dockerfile模板
&emsp;&emsp;模板例如：  
```Dockerfile
FROM ubuntu:latest
MAINTAINER haochen233 "252087791@qq.com"
ENV REFRESHED_AT 2020-01-09
RUN apt-get update
```
&emsp;&emsp;其他的不做解释，通过ENV指令来设置一个名为REFRESHED_AT的环境变量，这个环境变量用来表明该模板最后的更新时间。  
&emsp;&emsp;有了这个模板，如果想刷新一个构建（指令），只需修改ENV指令中的日期。这使Docker在命中ENV指令时开始重置这个缓存，并运行后续指令而无需依赖该缓存。

### 4.5.8 查看新镜像
&emsp;&emsp;可以使用`docker images`命令来完成，若想深入探求镜像是如何构建出来的，可以使用docker history命令。  

### 4.5.9 从新镜像启动容器
&emsp;&emsp;例如：  
`docker run -d -p 80 --name static_web docker_ubuntu/static_web \nginx -g "daemon off;"`
&emsp;&emsp;启动了一个名为`static_web`的新容器，同时指定了`-d`选项，告诉Docker以分离的方式在后台运行。这种方式非常适合运行类似`Nginx守护进程`这样需要长时间运行的进程。我们也指定了需要在容器中运行的命令：`nginx -g "daemon off"`,这将以前台运行的方式启动Nginx，来作为我们的Web服务器。  
&emsp;&emsp;其中的`-p`标志，该标志用来控制Docker在运行时应该公开哪些网络端口给外部（宿主机），运行一个容器时，Docker可以通过两种办法来在宿主机上分配端口。
- Docker可以在宿主机上随机选择一个为与49153~65535的一个比较大的端口号来映射到容器中的80端口上。  
- 可以在Docker宿主机中指定一个具体的端口号来映射到容器中的80端口上。  
- 使用`docker ps -l`命令来查看一下容器内部的端口分布情况。
- 也可以通过docker port来查看容器的端口映射情况。例如：  
`docker port 容器ID 80`该命令会返回宿主机中映射的端口。
- docker run命令的 -p选项还可以让我们灵活的管理容器和宿主机之间的端口映射关系。可以指定将容器中的端口映射到Docker宿主机的某一特定端口上。例如 ：  
`docker run -d -p 80:80 --name static_web2 docker_ubuntu/static_web \nginx -g "daemon off;"`这行命令会将容器内的80端口绑定到本地宿主机的80端口上。很明显，我们必须非常小心地使用这种直接绑定的做法；如果要运行多个容器，只有一个容器能够成功地将端口绑定到本地宿主机上，这回限制Docker的灵活性。
- 为了解决避免这种问题，可以将容器内的端口绑定到宿主机的不同端口上去，例如：`docker run -d -p 8080:80 --name static_web3 docker_ubuntu/static_web \nginx -g "daemon off;"`将容器中的80端口绑定到宿主机的8080端口上。
- 我们也可以将端口绑定在特定的网络接口（即**IP**地址）上。 
例如：`docker run -d -p 127.0.0.1 9190:80 --name static_web4 docker_ubuntu/static_web \nginx -g "daemon off;"`我们将容器内的80端口绑定到了本地宿主机的127.0.0.1这个IP的9190端口上去了，也可以使用类似的方式将容器内的80端口绑定到一个宿主机的随机端口上。例如：`docker run -d -p 127.0.0.1 9190:80 --name static_web5 docker_ubuntu/static_web \nginx -g "daemon off;"`。
- Docker还提供了一个更简单的方式，即`-P`参数（P大写）该参数会将容器内的 80 端口对本地宿主机公开，并且绑定到一个随机端口上，该命令会将用来构建该镜像的**Dockerfile**中的**EXPOSE**指令指定的其他端口也一并公开。

### 4.5.10 Dockerfile指令
&emsp;&emsp;我们已经看过了一些Dockerfile中可用的指令，如RUN和EXPOSE。但是，实际上还可以在Docker中放入很多其他指令。这些命令可以在[Docker官方网站](http://doc.docker.com/reference/builder/)查看Dockerfile中可以使用的全部指令的清单。

&emsp;&emsp;下面先简单介绍一些指令
1. CMD  
CMD用于指定一个容器启动时要运行的命令。这有点类似`RUN`指令，只是`RUN`指令是**指定镜像被构建时要运行的命令**，而CMD是指**容器被启动时要运行的命令**，这与docker run命令启动容器时指定要运行的命令非常类似。例如：  
`docker run -i -t docker_ubuntu/static_web /bin/true`该命令与如下CMD指令是等效的，`CMD ["bin/true"]` ,当然也可以为要运行的命令指定参数。例如：`CMD ["bin/true", "-l"]`

&emsp;&emsp;需要注意的是，要运行的命令是存放在一个数组结构汇中的。这将告诉Docker按指定的原样来运行该命令。当然也可以不使用数组指定CMD命令，这时候Docker会在指定的命令前加上/bin/sh -c。强烈推荐以数组语法来设置要执行的命令。

&emsp;&emsp;最后还需牢记，使用docker run命令可以覆盖CMD指令，如果docker run 与CMD指令都指定了命令，命令行中指定的命令会覆盖CMD指令。

&emsp;&emsp;例如：假如我们的Dockerfile文件中有`CMD ["/bin/bash"]`,然后使用docker build命令构建一个新镜像`test`。并基于此镜像启动一个新容器，`docker run -i -t test`，然后就会进入shell界面。这是因为Docker使用了CMD指令中指定的命令。  

&emsp;&emsp;例如：Dockerfile文件中定义与上面一样，只是用镜像启动容器的命令不同`docker run -i -t test /bin/ps`,启动容器会仅仅是执行了ps命令就退出了。这是因为docker run命令指定的命令覆盖了CMD指令指定的命令。

&emsp;&emsp;注意，在Dockerfile中只能指定一条CMD指令。如果指定了多条CMD指令，也只有最后一条CMD会被使用（以最后一条为准）。

2. ENTRYPOINT  
ENTRYPOINT指令与CMD指令非常相似。CMD指定的命令在启动容器时容易被覆盖，而ENTRYPOINT指定的命令不容易被覆盖，实际上，docker run 命令行中指定的任何参数都会被当做参数再次传递给`ENTRYPOINT`指令中指定的命令。


&emsp;&emsp;例如：`ENTRYPOINT ["/usr/sbin/nginx"]`，也可以通过输出的方式传递参数 `ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]`,现在重新构建我们的镜像设置为：`ENTRYPOINT ["/usr/sbin/nginx"]`，然后使用docker build 重新构建，然后基于新镜像启动容器：`docker run -i -t test -g "daemon off;"`，我们指定了`-g "daemon off;"`参数，该参数会传递给ENTRYPOINT指定的命令。然后相当于执行了`/usr/sbin/nginx -g "daemon off;"`。这就是ENTRYPOINT的用法。

&emsp;&emsp;我们可以同时使用**CMD**和**ENTRYPOINT**，这使得我们可以构建一个镜像，该镜像既可以运行一个默认的命令，还支持**docker run**命令行为该命令指定可覆盖的选项或者标志。例如：`ENTRYPOINT ["/usr/sbin/nginx"]`和`CMD ["-h"]`。这样就达成了我们的目的。  
&emsp;&emsp;还可以在运行时通过**docker run**的`--entrypoint`标志覆盖**ENTRYPOINT**指令。  

3. WORKDIR  
WORKDIR指令用来在从镜像中创建一个新容器时，在容器内部设置一个工作目录，ENTRYPOINT 和/或 CMD指定的程序会在这个目录下执行。

&emsp;&emsp;可以为特定的指令设置不同的工作目录。例如：

    WORKDIR /opt/webapp/db
    RUN bundle install
    WORKDIR /opt/webapp
    ENTRYPOINT ["rackup"]

先将工作目录切换到`/opt/webapp/db`然后运行了`RUN bundle install`命令，接着将工作目录切换到`/opt/webapp`,然后设置了ENTERPOINT指令来启动rackup命令。

&emsp;&emsp;可以通过docker run的`-w`在启动容器时覆盖工作目录。例如：`docker run -it -w /var/log test pwd`，该命令会将工作命令设置为`/var/log`。