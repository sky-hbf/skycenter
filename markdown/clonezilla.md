#再生龙系统的搭建（环境centos7.0）
------------------
##步骤一:安装tftp
- yum -y install xinetd tftp tftp-server
##步骤二：修改配置tftp文件，并重启xinet.d服务
> service tftp
> { 
>
>        socket_type             = dgram
>        protocol                = udp
>        wait                    = yes
>        user                    = root
>        server                  = /usr/sbin/in.tftpd
>       server_args               = -s /web/tftpboot
>        disable                 = no
>        per_source              = 11
>        cps                     = 100 2
>        flags                   = IPv4
> }
##步骤三：安装samba
- yum -y install samba
##步骤四： 修改samba配置文件,并重启samba服务
>[box]

>   comment = BOX

>   path = /tbcdata/clone

>   valid users = box

>   public = no

>   writable = yes

>   write list = box

>   printable = no

>   create mask = 0775

##步骤五：在dhcp服务器上修改dhcp配置文件，增加内容如下

> next-server 192.168.0.201;

> filename "/default/pxelinux.0";


##步骤六：建立所需要的目录和文件
- mkdir -p /web/clone 
- mkdir -p /web/tftpboot/default
- mkdir -p /web/tftpboot/default/pxelinux.cfg
- cp /usr/share/syslinux/pxelinux.0 /web/tftpboot/default
- touch /web/tftpboot/default/pxelinux.cfg/default
##步骤七： 编辑pxelinux.cfg下面的default文件，内容如下  
> default CloneZilla

> timeout 3000

> prompt 1

> label CloneZilla

> kernel clonezilla/vmlinuz-clonezilla ghes.disable=1

>  append initrd=clonezilla/clonezilla.img boot=live union=aufs noswap noprompt vga=788 live-config live-media-timeout=15 fetch=tftp://192.168.0.201/default/clonezilla/filesystem.squashfs

##步骤八：再生龙源码的配置
- 拷贝再生龙源码包下面的LIVE文件夹拷贝到 /web/tftpboot/default/下面,命名为clonezilla

- 进入到clonezilla文件夹里面，vmlinuz重命名为vmlinuz-clonezilla,initrd.img重名为clonezilla.img


##步骤九：重启各个服务器，关闭selinux和防火墙，并找一台客户端网卡引导测试






