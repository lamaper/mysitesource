+++
date = '2022-09-05T21:17:02+08:00'
draft = true
title = 'WinServer19+CTFd+Docker部署动态靶机'
categories = ['ctf']
+++

lamaper@qq.com



[安装|CTFd 文档](https://docs.ctfd.io/docs/deployment/installation)

## 一、部署服务端Docker

正确安装Windows Server 2019；

若要在 Windows Server 上安装 Docker，可以使用由 Microsoft 发布的 [OneGet 提供程序 PowerShell 模块](https://github.com/oneget/oneget)（称为 [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)）。

注：在购买的服务器中不可直接安装docker desktop，因为购买的服务器已经进行过一次虚拟化，安装

此提供程序启用 Windows 中的容器功能，并安装 Docker 引擎和客户端。 以下是操作方法：

### 0、安装FastGithub

[Releases · dotnetcore/FastGithub](https://github.com/dotnetcore/fastgithub/releases)

[fastgithub国内镜像(gitee.com)](https://gitee.com/mirrors/fastgithub?_from=gitee_search)

运行fastgithub

### **1、**安装docker

打开提升的 PowerShell 会话，从 [PowerShell 库](https://www.powershellgallery.com/packages/DockerMsftProvider)安装 Docker-Microsoft PackageManagement 提供程序。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

如果系统提示安装 NuGet 提供程序，键入 `Y` 进行安装。

**如果出错，应该关闭PowerShell窗口，用admin权限重新打开操作，因为实践中发现下载中断后无法继续的情况。**

如果在打开 PowerShell 库时遇到错误，则可能需要将 PowerShell 客户端使用的 TLS 版本设置为 TLS 1.2。 为此，请运行以下命令：

```powershell
# Set the TLS version used by the PowerShell client to TLS 1.2.


[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;
```

### **2、**使用 PackageManagement PowerShell 模块安装最新版本的 Docker

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell 询问是否信任包源“DockerDefault”时，键入 `A` 以继续进行安装。

在安装完成后，请重启计算机。

```powershell
Restart-Computer -Force
```

### **3、**如果希望稍后更新 Docker，请执行以下操作：

使用以下命令检查安装的版本：

```powershell
Get-Package -Name Docker -ProviderName DockerMsftProvider
```

使用以下命令查找当前版本：

```powershell
Find-Package -Name Docker -ProviderName DockerMsftProvider
```

准备好升级后，运行以下命令：

```powershell
Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force
```

### 4、完善部署、安装GUI、安装docker-compose

运行以下命令以启动 Docker：

```powershell
Start-Service Docker
```

在Powershell输入命令查看是否正常运行：

```powershell
docker
```

安装完成的docker EE 默认内核为windows，通常情况下需要切换到linux内核，可通过如下代码进行切换：

```powershell
[Environment]::SetEnvironmentVariable("LCOW_SUPPORTED", "1", "Machine")

Restart-Service Docker
```

Windows server 的 docker 没有可视化UI，可安装第三方的工具，比如 portainer：

```powershell
docker run -d --name portainer --restart always -p 9000:9000 -v \\.\pipe\docker_engine:\\.\pipe\docker_engine portainer/portainer
```

安装docker-compose：

```powershell
Invoke-WebRequest "https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-Windows-x86_64.exe" -UseBasicParsing -OutFile $Env:ProgramFiles\Docker\docker-compose.exe
```

注意，在安装docker-compose后请输入该命令以确保docker-compose正确安装：

```powershell
docker-compose --version
```

如果报错，则代表docker-compose没有被正确安装，解决方法是，直接使用github下载最新版[Release v2.15.1 · docker/compose · GitHub](https://github.com/docker/compose/releases/tag/v2.15.1)，之后将下载后的文件放入/docker目录下，删除原来的``docker-compose.exe``，替换为新下载的文件，并改名为``docker-compose.exe``，即可。

## 二、下载安装CTF-d

### 1、下载CTF-d

[Release 3.5.0 · CTFd/CTFd · GitHub](https://github.com/CTFd/CTFd/releases/tag/3.5.0)

### 2、部署CTF-d

修改ctfd目录下docker-compose.yml的SECRET_KEY；

在ctfd目录下打开powershell，运行

```powershell
docker-compose up
```

在本地浏览器http://localhost:8000进行初始化

### 问题解决：no matching manifest for windows/amd64 10.0.17763 in the manifest list entries

当出现此类问题时，代表docker启动的参数有问题，此时先停止docker服务；

```powershell
net stop docker
```

之后win+R启动运行，打开注册表（regedit），进入到``计算机\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\docker``下，修改``ImagePath``，将其键值改为：``"<Docker安装位置>\Docker\dockerd.exe" --run-service --experimental=true``;

然后重新启动docker服务

```powershell
net start docker
```



