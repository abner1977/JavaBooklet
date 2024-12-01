## Docker

### 一、介绍

基于 Google 公司推出的 **Go 语言**实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 [GitHub](https://github.com/docker/docker) 上进行维护。

Docker 项目的目标是**实现轻量级的操作系统虚拟化解决方案**。 Docker 的基础是 Linux 容器（LXC）等技术。

可以实现快速交付和部署、更高效的虚拟化、更轻松的迁移和扩展、更简单的管理。

### 二、对比传统虚拟机总结

| **特性**   | **容器**           | **虚拟机** |
| ---------- | ------------------ | ---------- |
| 启动       | 秒级               | 分钟级     |
| 硬盘使用   | 一般为 MB          | 一般为 GB  |
| 性能       | 接近原生           | 弱于       |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |

### 三、docker架构

<img src="D:\dailySoftWare\typora\md\img\1234.png" alt="1234" style="zoom:50%;" />

host --- 主机载体 == docker安装的地方

| Images    | Docker  镜像，用于创建 Docker  容器的模板。                  |
| --------- | ------------------------------------------------------------ |
| Container | Docker  容器，独立运行的一个或一组应用。                     |
| Client    | Docker  客户端，使用 Docker  Api与 Docker  的守护进程通信。  |
| Host      | Docker  主机，一个物理或者虚拟的机器  用于执行 Docker  守护进程和容器。 |
| Registry  | Docker  仓库，用来保存镜像                                   |
| Machine   | 一个简化Docker安装的命令行工具，  比如VirtualBox、  Digital Ocean、Microsoft Azure。 |



### 四、Cenos7 Docker配置

出现问题 参照 官网 https://docs.docker.com/engine/install/centos/

1、docker是否安装查询

```shell
yum list installed|grep docker 
yum -y remove docker-ce.x86_64 # 如果存在需要先卸载
# 删除存储目录
# rm -rf /etc/docker
# rm -rf /run/docker
# rm -rf /var/lib/dockershim
# rm -rf /var/lib/docker
```

2、centos内核版本查看 

```shell
uname-r  #docker要求centOs的内核版本在3.10 以上

yum -y update #确保yum包最新
```

3、安装必要软件包

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 添加软件源

yum makecache fast  #更新yum 缓存

yum -y install docker-ce #安装docker-ce

systemctl start docker #启动

docker run hello-world  #运行hello-world 镜像： 

```

































### 五、Docker常用命令





### 六、问题解决

1、/var/run/yum.pid 已被锁定，PID 为 3954 的另一个程序正在运行。

```shell
rm -f /var/run/yum.pid
```





### 七、参考文档

[centOs7 安装docker 镜像]: https://blog.csdn.net/weixin_39477597/article/details/87715899

