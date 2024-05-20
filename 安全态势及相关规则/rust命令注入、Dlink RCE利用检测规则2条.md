
- [CVE-2024-24576利用检测方式一则](#cve-2024-24576%E5%88%A9%E7%94%A8%E6%A3%80%E6%B5%8B%E6%96%B9%E5%BC%8F%E4%B8%80%E5%88%99)
- [CVE-2024-3273利用检测规则一则](#cve-2024-3273%E5%88%A9%E7%94%A8%E6%A3%80%E6%B5%8B%E8%A7%84%E5%88%99%E4%B8%80%E5%88%99)


## CVE-2024-24576利用检测方式一则

任意rust、Python、PHP、node.js进程触发CreateProcess、 CreateProcessA 或 CreateProcessW时命令行参数符合如下格式

```
.*(?i)cmd\.exe\W.*\/c\W.*(:\\.*|.*)\.(bat|cmd).*\W".*\\"\W&\W.*"
```

reference:

[CVE-2024-24576 Windows 下多语言命令注入漏洞分析 | 程序人生](https://programlife.net/2024/04/14/cve-2024-24576-rust-command-injection-vulnerability/)


## CVE-2024-3273利用检测规则一则


```
title: CVE-2024-3273
id: d8e3702d-35d5-4adc-9724-2a1a9a75b64d
status: test
description: CVE-2024-3273
references:
    - https://www.freebuf.com/news/397297.html
    - https://github.com/adhikara13/CVE-2024-3273
date: 2024/05/13
tags:
    - CVE-2024-3273
    - D-Link
    - RCE
logsource:
    category: webserver
detection:
    selection:
        url|contains: /cgi-bin/nas_sharing.cgi?user=messagebus&passwd=&cmd=15&system=
    selection1:
        response.status: 200
    selection2:
        response.body|contains: root
    condition: selection and selection1 and selection2
level: critical
```


reference:

[D-Link NAS 设备存在严重 RCE 漏洞，数万用户受到影响 - FreeBuf网络安全行业门户](https://www.freebuf.com/news/397297.html)

[GitHub - adhikara13/CVE-2024-3273: Exploit for CVE-2024-3273, supports single and multiple hosts](https://github.com/adhikara13/CVE-2024-3273)