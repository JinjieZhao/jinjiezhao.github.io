# 开篇 开发环境配置

工欲善其事，必先利其器。不管你是刚入门的初学者，还是经验丰富的开发，一定都被环境配置折磨过。本文介绍如何利用vagrant 搭建统一的开发环境，并介绍一些使用技巧。

## 什么是vagrant

vagrant 是一个虚拟机管理软件，可以快速导入已有的虚拟机、配置环境、打包复用。下面是一个简单的演示。

```shell
# 导入封装好的虚拟机
vagrant init ubuntu1804
vagrant up

# 通过ssh 访问
vagrant ssh

# 封装好的虚拟机有开发需要的各种软件
gcc --version
```

## vagrant 和 docker 区别

docker 是运行环境，里面只有非常少的依赖。vagrant 对应的是虚拟机，我们可以在虚拟机里面安装IDE，配置zsh vim 等

## vagrant 安装

vagrant 是虚拟机管理软件，所以我们还需要安装一个虚拟机，推荐使用virtualbox，原因是virtualbox 是免费的，网上大量封装好的镜像都是基于virtualbox 的，而且virtualbox 跨平台，Windows Linux MacOS 都可以使用。

下载地址：

1. vagrant https://www.vagrantup.com/downloads 
2. virtualbox https://www.virtualbox.org/wiki/Downloads

## vagrant 导入镜像

vagrant 官网https://app.vagrantup.com/ 提供了大量镜像，遗憾的是官网速度很慢。好在国内的镜像源也提供了下载地址。

1. centos7 https://mirrors.ustc.edu.cn/centos-cloud/centos/7/vagrant/x86_64/images/
2. centos8 https://mirrors.ustc.edu.cn/centos-cloud/centos/8/vagrant/x86_64/images/
3. ubuntu1404 https://mirrors.ustc.edu.cn/ubuntu-cloud-images/vagrant/trusty/
4. ubuntu1604 https://mirrors.ustc.edu.cn/ubuntu-cloud-images/xenial/
5. ubuntu1804 https://mirrors.ustc.edu.cn/ubuntu-cloud-images/bionic/
6. ubuntu2004 https://mirrors.ustc.edu.cn/ubuntu-cloud-images/focal

以ubuntu1804 为例，找到vagrant 镜像地址https://mirrors.ustc.edu.cn/ubuntu-cloud-images/bionic/current/bionic-server-cloudimg-amd64-vagrant.box 

```shell
# 导入ubuntu1804 镜像到vagrant
vagrant box add ubuntu1804 https://mirrors.ustc.edu.cn/ubuntu-cloud-images/bionic/current/bionic-server-cloudimg-amd64-vagrant.box
```

## vagrant 启动虚拟机

vagrant 启动的时候根据Vagrantfile 对虚拟机进行一些设置

```shell
# 创建一个新目录
mkdir foo && cd foo

# 导入封装好的虚拟机
# 执行vagrant init ubuntu1804 之后会创建一个Vagrantfile 
vagrant init ubuntu1804

# 对Vagrantfile 进行必要的修改
#
# 修改分配给虚拟机的CPU 和内存
# 给虚拟机分配一个静态ip 地址
# 设置共享目录
# 设置初始化脚本
#
# 详见下面的示例

# 启动虚拟机
vagrant up

# 通过ssh 访问
vagrant ssh
```

下面是我自己的Vagrantfile 示例

```ruby
Vagrant.configure("2") do |config|
  # 设置导入的镜像名
  config.vm.box = "ubuntu1804"

  # 设置虚拟机的主机名  
  config.vm.hostname = 'dev'

  # 给虚拟机分配一个静态ip 地址
  config.vm.network "private_network", ip: "192.168.100.100"

  # 设置共享目录
  config.vm.synced_folder "D:/me", "/me"

  config.vm.provider "virtualbox" do |vb|

	# 修改CPU 和内存大小
    vb.memory = "8096"
    vb.cpus = "4"
  end

  # 虚拟机初始化的时候执行init 脚本
  # init.sh 应该在当前目录
  config.vm.provision "shell", path: "init.sh"
end
```

启动之后，可以使用`vagrant ssh-config` 查看具体ssh 配置，方便我们通过putty xshell mobaxterm 等软件连接到虚拟机

## vagrant 打包现有虚拟机

关机之后执行`vagrant package --output name`，下次要用的时候直接`vagrant box add <box-name> <path>` 就可以啦，快速拥有一个统一的开发环境。

## vagrant 使用技巧

1. 导入镜像到其他磁盘
`vagrant box add <box-name> <path>` 导入镜像会默认导入到`$HOME/.vagrant.d`，Windows 下会占据C 盘的位置，可以通过设置`VAGRANT_HOME` 到其他磁盘解决问题
2. 打包ubuntu 镜像
  需要添加`vb.customize [ "modifyvm", :id, "--uartmode1", "file", File::NULL ]` 到Vagrantfile，不然其他人导入的时候会访问你本地目录下log 文件导致出错
3. centos 镜像共享目录
  centos 官网镜像没有安装vbguest，需要手动安装

