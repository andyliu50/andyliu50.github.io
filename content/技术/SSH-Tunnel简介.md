title: SSH Tunnel简介
date: 2020-05-27 12:06:16

# 什么是SSH Protocol?

**SSH Protocol**（或者称为**Secure Shell**）是一种安全的远程登录计算机的方式。相对于传统的远程登录协议，例如Telnet, rlogin等，SSH采用了对远程连接进行加密的方式，来保证身份认证和数据传输的机密性。  
  
  
	
# 什么是SSH Tunnel?

**SSH tunneling**，又称为**SSH port forwarding**，可以将客户端程序发送的数据通过加密的SSH Tunnel，传送给远端的服务器应用。这种方式比较适用于一些传统的应用， 由于这些应用不支持加密传输，并且进行二次开发的成本较高，因此可以采用SSH Tunneling技术，对这些应用传输的数据进行加密。

SSH Tunnel是由SSH客户端和SSH服务器之间，通过SSH Protocol建立的一条安全通道。应用的客户端程序和服务器程序之间通讯的数据，都会通过SSH Tunnel进行传输。                                           

App Client <-> SSH Client <------------SSH Tunnel------------> SSH Server <-> App Server

 ![]({static}/images/SSH-Tunnel简介/1.png)
  
  

# SSH Tunnel的应用场景
  

- 对于一些传统的应用，由于这些应用不支持加密传输，并且进行二次开发的成本较高，因此可以采用SSH     Tunneling技术，对这些应用传输的数据进行加密。
- 由于防火墙的限制，许多服务器只能在内网访问，无法通过Internet远程访问。通过SSH Tunnel，可以远程访问内网的应用和服务器。
- 通过SSH Tunnel，可以将SSH Client设置为SOCKS代理服务器，所有上网的流量，都将通过SSH Tunnel发送到远端的SSH服务器，并由SSH服务器转发到Internet。

# SSH Tunnel的使用方式

## Local Port Forwarding

**Local Port Forwarding**需要**SSH Client**在本地的IP地址（127.0.0.1或者0.0.0.0）监听一个TCP端口，客户端程序只需要连接到SSH Client的IP地址和监听的端口，客户端程序(App Client)发送的数据都将通过SSH Client和SSH Server之间建立的SSH Tunnel传输，最后，由SSH Server将数据发送给服务端程序(App Server)。

逻辑上来讲，相当于将服务端程序的服务端口映射到SSH Client监听的端口。最终目的是将服务端程序的服务端口映射到App Client可以访问的节点。

这种方式适用于服务器端(App Server)的SSH Server允许被防火墙发布到Internet，这样SSH Client可以远程连接到SSH Server。

App Client <-> **SSH Client(Listen on a TCP port)** ------SSH Tunnel------> SSH Server <-> App Server

配置方式： 

在SSH Client上执行以下命令，该命令执行后，会在SSH Client上监听TCP端口8000：


```bash
ssh -L 127.0.0.1:8000:www.intranet.domain:80 root@sshserver.intranet.domain
```
-L	表示使用Local Port Forwarding

127.0.0.1  	在SSH Client所在计算机的本地127.0.0.1监听TCP服务端口

8000:      SSH Client监听TCP服务端口8000

[www.intranet.domain](http://www.intranet.domain)  		远端内网服务器的主机域名

80      远端内网服务器的服务端口80

root@sshserver.intranet.domain   远端SSH服务器的登录账号(root)和地址(sshserver.intranet.domain)

 

注意：由于SSH Client只是在localhost(127.0.0.1)监听服务端口8000，所以App Client必须和SSH Client在同一台计算机上，才能访问远程应用www.intranet.domain。

 

如果App Client运行在另外一台计算机上，则需要使用以下命令，让SSH Client在IP地址0.0.0.0上监听服务端口8000，这样，SSH Client就可以接收来自于任何计算机的客户端程序访问。

 

```bash
ssh -L 0.0.0.0:8000:www.intranet.domain:80 root@sshserver.intranet.domain
```
  
  
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------  
  
  

## Remote Port Forwarding

在有些场景中，服务器端(App Server)的SSH Server无法被发布到Internet，这时候就只能使用Remote Port Forwadingd的方式，将服务端应用程序的服务端口，映射到客户端应用程式(App Client)这一侧的SSH Server。

逻辑上来讲，相当于将服务端程序的服务端口映射到SSH Server监听的端口。最终目的是将服务端程序的服务端口映射到App Client可以访问的节点。


App Client <-> **SSH Server(Listen on a TCP port)** <------SSH Tunnel------- SSH Client <-> App Server

配置方式：

在SSH Client上执行以下命令，该命令执行后，会在SSH Server上监听TCP端口8000，绑定的IP地址是127.0.0.1：

```bash
ssh -R :8000:www.intranet.domain:80 root@sshserver.intranet.domain
```

-R         表示使用Remote Port Forwarding

8000:      SSH Servert监听TCP服务端口8000

[www.intranet.domain](http://www.intranet.domain)  远端内网服务器的主机域名

80         远端内网服务器的服务端口80

root@sshserver.intranet.domain  远端SSH服务器的登录账号(root)和地址(sshserver.intranet.domain)


注意：默认情况下，SSH Server只会绑定IP地址127.0.0.1，如果需要绑定IP地址0.0.0.0，需要修改sshd_config配置文件中的参数**GatewayPorts**。


以下是关于参数**GatewayPorts**的三种方式：

```
GatewayPorts no
```


该设置只允许SSH Server自己连接监听的服务端口8000，拒绝来自于外部计算机的连接请求。

 

```
GatewayPorts yes`
```
 

该设置允许任何计算机的连接请求，如果SSH Server在Internet上，任何计算机都可以访问该服务端口8000。

 

通过执行以下命令，可以在SSH Server上绑定IP地址0.0.0.0，允许任何计算机访问服务端口8000。

 

```bash
ssh -R 0.0.0.0:8000:www.intranet.domain:80 root@sshserver.intranet.domain`
```
 

```
GatewayPorts clientspecified
```

 

该设置只允许来自于指定IP地址的连接请求。格式如下：


通过执行以下命令，可以限制只允许来自于IP地址2.2.2.2的计算机访问服务端口8000。

 

```bash
ssh -R 2.2.2.2:8000:www.intranet.domain:80 root@sshserver.intranet.domain`
```  
  
  
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------  
  
  

## Dynamic Port Forwarding

 

Dynamic Port Forwarding可以将SSH Client配置为SOCKS代理服务器，所有的浏览通过SSH Tunnel发送到SSH Server，并由SSH Server转发至内网应用，或者Internet。

 

```bash
ssh -C -D 1080 root@sshserver.intranet.domain  //绑定IP地址127.0.0.1

ssh -C -D 0.0.0.0:1080 root@sshserver.intranet.domain  //绑定IP地址0.0.0.0
```

 

-C 启用数据压缩

-D 使用Dynamic Port Forwarding

 

App Client -> **SSH Client(Listen on a TCP port)** ---------SSH Tunnel---------> SSH Server -> Internet/Corporate App

 

 

## 启用和关闭Port Forwarding

 

通过配置SSH Server的配置文件**sshd_config**中的参数**AllowTcpForwarding**，可以启用和关闭Port Forwarding。

 

> **AllowTcpForwarding**
> 
> Specifies whether TCP forwarding is permitted. 
> 
> The available options are:
> 
> 
> **yes** (the default) or all to allow TCP forwarding, 
> 
> **no** to prevent all TCP forwarding, 
> 
> **local** to allow local (from the perspective of ssh(1)) forwarding only
>  
> **remote** to allow remote forwarding only. 
> 
> Note that disabling TCP forwarding does not improve security unless users are also denied shell access, as they can always install their own forwarders.