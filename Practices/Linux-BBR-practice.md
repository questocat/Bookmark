### Linux BBR 实践

BBR 的作用就是加速 TCP，即：尽可能的充分利用带宽；降低 buffer 占用率，从而降低延迟。

##### Before
```code
ubuntu@ip-x-x-x-x:/usr/local/bin$ sudo youtube-dl https://www.youtube.com/watch?v=4SdcmxGC3Yc&list=PLo7mBDsRHu11Szh1t-CRqwliRGRA1JOZJ&index=2
[1] 23958
[2] 23959
ubuntu@ip-x-x-x-x:/usr/local/bin$ WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this.
[youtube] 4SdcmxGC3Yc: Downloading webpage
[youtube] 4SdcmxGC3Yc: Downloading video info webpage
[youtube] 4SdcmxGC3Yc: Extracting video information
[youtube] 4SdcmxGC3Yc: Downloading MPD manifest
[youtube] 4SdcmxGC3Yc: Downloading MPD manifest
[download] Destination: Getting_started_webinar_-_February_2017-4SdcmxGC3Yc.mp4
[download] 100% of 171.28MiB in 00:40
[1]-  Done                    sudo youtube-dl https://www.youtube.com/watch?v=4SdcmxGC3Yc
[2]+  Done                    list=PLo7mBDsRHu11Szh1t-CRqwliRGRA1JOZJ
```

##### After
```code
ubuntu@ip-x-x-x-x:/usr/local/bin$ sudo youtube-dl https://www.youtube.com/watch?v=4SdcmxGC3Yc&list=PLo7mBDsRHu11Szh1t-CRqwliRGRA1JOZJ&index=2
[1] 1945
[2] 1946
ubuntu@ip-x-x-x-x:/usr/local/bin$ WARNING: Assuming --restrict-filenames since file system encoding cannot encode all characters. Set the LC_ALL environment variable to fix this.
[youtube] 4SdcmxGC3Yc: Downloading webpage
[youtube] 4SdcmxGC3Yc: Downloading video info webpage
[youtube] 4SdcmxGC3Yc: Extracting video information
[youtube] 4SdcmxGC3Yc: Downloading MPD manifest
[youtube] 4SdcmxGC3Yc: Downloading MPD manifest
[download] Destination: Getting_started_webinar_-_February_2017-4SdcmxGC3Yc.mp4
[download] 100% of 171.28MiB in 00:08
[1]-  Done                    sudo youtube-dl https://www.youtube.com/watch?v=4SdcmxGC3Yc
[2]+  Done                    list=PLo7mBDsRHu11Szh1t-CRqwliRGRA1JOZJ
```

##### 升级前

系统内核要求：Linux kernel 4.9 及以上

检查系统内核版本
```code
uname -r
```
如果系统版本不满足要求，先升级内核：  
在 [内核下载地址](http://kernel.ubuntu.com/~kernel-ppa/mainline) 里找到你要用的内核，即 4.9+ 版本。
另外你要注意：  
如果系统是 64 位，则下载 amd64 的 linux-image 中含有 generic 这个 deb 包；  
如果系统是 32 位，则下载 i386 的 linux-image 中含有 generic 这个 deb 包
这里我使用的是 4.10 版本：
```code
sudo wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/linux-image-4.10.0-041000-generic_4.10.0-041000.201702191831_amd64.deb
sudo dpkg -i linux-image-4.10.0-041000-generic_4.10.0-041000.201702191831_amd64.deb
sudo update-grub
sudo reboot
```

##### 安装（开启） BBR
```code
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf  
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf  
sysctl -p  # 立即生效配置
```
内核是否已开启 BBR
```code
sysctl net.ipv4.tcp_available_congestion_control  
sysctl net.ipv4.tcp_congestion_control  
```
当前内核是否载入 tcp_bbr 模块
```code
lsmod | grep bbr  
```

参考：
 - [内核下载地址](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10)
 - [开启TCP BBR拥塞控制算法](https://github.com/iMeiji/shadowsocks_install/wiki/%E5%BC%80%E5%90%AFTCP-BBR%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%E7%AE%97%E6%B3%95)
 - [Linux Kernel 4.9 中的 BBR 算法与之前的 TCP 拥塞控制相比有什么优势？](https://www.zhihu.com/question/53559433)
