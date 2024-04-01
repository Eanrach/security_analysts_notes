
- [第一步IOC监控](#%E7%AC%AC%E4%B8%80%E6%AD%A5ioc%E7%9B%91%E6%8E%A7)
- [第二步寻找DinodasRAT运行痕迹](#%E7%AC%AC%E4%BA%8C%E6%AD%A5%E5%AF%BB%E6%89%BEdinodasrat%E8%BF%90%E8%A1%8C%E7%97%95%E8%BF%B9)
- [第三步阻断链接](#%E7%AC%AC%E4%B8%89%E6%AD%A5%E9%98%BB%E6%96%AD%E9%93%BE%E6%8E%A5)
- [第四步通知人员处理/结束流程](#%E7%AC%AC%E5%9B%9B%E6%AD%A5%E9%80%9A%E7%9F%A5%E4%BA%BA%E5%91%98%E5%A4%84%E7%90%86%E7%BB%93%E6%9D%9F%E6%B5%81%E7%A8%8B)

## 第一步IOC监控

对情报IP通信或发现哈希直接进入下一步，只发现对域名通信则判断ip是否为情报ip，是则进入下一步，否(判断为已被sinkhole)则通知相关人员后结束流程。

```
DOMAINs:
	update.centos-yum.com 
	update.microsoft-setting.com 
IPs:
	199.231.211.19
HASHs:
	6302acdfce30cec5e9167ff7905800a6220c7dda495c0aae1f4594c7263a29b2
	57f64f170dfeaa1150493ed3f63ea6f1df3ca71ad1722e12ac0f77744fb1a829
	8138f1af1dc51cde924aa2360f12d650
	decd6b94792a22119e1b5a1ed99e8961
```


## 第二步寻找DinodasRAT运行痕迹

通过提前部署的agent判断24小时内是否存在以下行为或指令

```
创建以下文件
	/etc/rc.local
	/tmp/.netc.ini
	/usr/lib/systemd/system/rc.local.service
命令
	ln -s /lib/systemd/system/rc.local.service /etc/systemd/system/
	chmod 777 /etc/rc.local
```

DinodasRAT会创建或写入`/etc/.netc.conf`储存受害者信息，DinodasRAT会通过`/etc/rc.local`将自己附加到初始化流程中实现自启动，上述两个文件操作行为均存在则判断为该系统已被成功植入并可能被该恶意程序远程控制，若只存在对`/etc/rc.local`的操作则判断为可能被DinodasRAT植入，进入下一步流程。

## 第三步阻断链接

以防被控主机进一步对资产造成损害，联动其他设备阻断其与IOC地址的通信，若其非重要业务主机则可对其进行断网处理。

## 第四步通知人员处理/结束流程

通知相关人员进行应急响应并反馈处理结果



reference:
[Analysis of DinodasRAT Linux implant](https://securelist.com/dinodasrat-linux-implant/112284/)
[VirusTotal](https://www.virustotal.com/gui/file/15412d1a6b7f79fad45bcd32cf82f9d651d9ccca082f98a0cca3ad5335284e45/behavior)
[VirusShare](https://virusshare.com/file?15412d1a6b7f79fad45bcd32cf82f9d651d9ccca082f98a0cca3ad5335284e45)