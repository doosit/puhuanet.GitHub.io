**参考：**
* [定制AlienVault NIDS规则](https://cybersecurity.att.com/documentation/usm-appliance/ids-configuration/customizing-alienvault-nids-rules.htm)   
* [Suricata 规则](https://suricata.readthedocs.io/en/latest/rules/index.html)   

# 如何定制NIDS规则
1. 通过SSH连接并登入OSSIM，将显示设置菜单。选择Jailbreak System    

2. 编辑/etc/suricata/rules/local.rules文件，写入和修改规则，保存退出    
为确保该规则与现有规则不冲突，应使用5000000到5999999之间的SID    

3. 在/etc/suricata/rule-files.yaml添加对local.rules的引用    

4. 将规则导入数据库(Sensor不需要执行此步骤)    
```
perl /usr/share/ossim/scripts/create_sidmap.pl /etc/suricata/rules
```

5. 重新启动NIDS服务和Agent服务，以使更改生效
```
#service suricata restart
#service ossim-agent restart
```

# 规则之classtype    

```
# classification.config 
# config classification:shortname,short description,priority

config classification: not-suspicious,Not Suspicious Traffic,3 #无可疑，无可疑流量
config classification: unknown,Unknown Traffic,3 #未知，未知流量
config classification: bad-unknown,Potentially Bad Traffic, 2 
config classification: attempted-recon,Attempted Information Leak,2 #信息泄漏
config classification: successful-recon-limited,Information Leak,2 #信息泄漏
config classification: successful-recon-largescale,Large Scale Information Leak,2 #大规模信息泄漏
config classification: attempted-dos,Attempted Denial of Service,2 #尝试拒绝服务
config classification: successful-dos,Denial of Service,2 #拒绝服务
config classification: attempted-user,Attempted User Privilege Gain,1 #获得用户权限
config classification: unsuccessful-user,Unsuccessful User Privilege Gain,1 #获得用户权限
config classification: successful-user,Successful User Privilege Gain,1 #获得用户权限
config classification: attempted-admin,Attempted Administrator Privilege Gain,1 #获得管理员权限
config classification: successful-admin,Successful Administrator Privilege Gain,1 #获得管理员权限


# NEW CLASSIFICATIONS
config classification: rpc-portmap-decode,Decode of an RPC Query,2 #RPC 查询解码
config classification: shellcode-detect,Executable code was detected,1 #可执行代码
config classification: string-detect,A suspicious string was detected,3 #可疑字符串
config classification: suspicious-filename-detect,A suspicious filename was detected,2 #可疑文件名
config classification: suspicious-login,An attempted login using a suspicious username was detected,2 #可疑用户名登录尝试
config classification: system-call-detect,A system call was detected,2 #系统调用
config classification: tcp-connection,A TCP connection was detected,4 #tcp 连接
config classification: trojan-activity,A Network Trojan was detected, 1 #网络特洛伊木马
config classification: unusual-client-port-connection,A client was using an unusual port,2 #客户端使用异常端口
config classification: network-scan,Detection of a Network Scan,3 #网络扫描
config classification: denial-of-service,Detection of a Denial of Service Attack,2 #拒绝服务攻击
config classification: non-standard-protocol,Detection of a non-standard protocol or event,2 #非标准协议或事件
config classification: protocol-command-decode,Generic Protocol Command Decode,3 #通用协议解码
config classification: web-application-activity,access to a potentially vulnerable web application,2 #Web应用活动
config classification: web-application-attack,Web Application Attack,1 #Web应用攻击
config classification: misc-activity,Misc activity,3 #杂项活动
config classification: misc-attack,Misc Attack,2 #杂项攻击
config classification: icmp-event,Generic ICMP event,3 #通用ICMP事件
config classification: kickass-porn,SCORE! Get the lotion!,1 #色情
config classification: policy-violation,Potential Corporate Privacy Violation,1 #违反政策
config classification: default-login-attempt,Attempt to login by a default username and password,2 #尝试使用默认用户名和密码登录
```

## 一些自定义规则    

### [QQPCMgr 腾讯电脑管家](suricata_localrules#5200101)   
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"HFF INFO QQPCMgr"; flow:to_server,established; priority:1; content:"/qqpcmgr/data_update/"; http_uri; content:"QQPCMgr"; http_user_agent; reference:url,puhua.net/suricata_localrules#5200101; classtype:policy-violation; sid:5200101; rev:1;)

### [WeChat Client 微信客户端](suricata_localrules#5200102)   
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"HFF CHAT Wechat Client"; flow:to_server,established; content:"MicroMessenger Client"; http_user_agent; content:"dns.weixin.qq.com"; http_host; reference:url,puhua.net/suricata_localrules#5200102; classtype:policy-violation; sid:5200102; rev:1;)

### [代替 2003492](suricata_localrules#5200103)   
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"ET INFO Suspicious Mozilla User-Agent - Likely Fake (Mozilla/4.0)"; flow:to_server,established; priority:1;  content:!".duba.net"; http_header; content:!"/qqpcmgr/data_update/"; http_uri; content:!".baofeng.com"; http_host; content:"User-Agent|3a| Mozilla/4.0|0d 0a|"; fast_pattern; nocase; http_header; content:!"/CallParrotWebClient/"; http_uri; content:!"Host|3a| www|2e|google|2e|com|0d 0a|"; nocase; http_header; content:!"Cookie|3a| PREF|3d|ID|3d|"; nocase; http_raw_header; content:!"Host|3a 20|secure|2e|logmein|2e|com|0d 0a|"; nocase; http_header; content:!"Host|3a 20|weixin.qq.com"; http_header; nocase; content:!"Host|3a| slickdeals.net"; nocase; http_header; content:!"Host|3a| cloudera.com"; nocase; http_header; content:!"Host|3a 20|secure.digitalalchemy.net.au"; http_header; content:!".ksmobile.com|0d 0a|"; http_header; content:!"gstatic|2e|com|0d 0a|"; http_header; content:!"weixin.qq.com|0d 0a|"; http_header; content:!"|2e|cmcm|2e|com|0d 0a|"; http_header; content:!".deckedbuilder.com"; http_header; content:!".mobolize.com"; http_header; reference:url,puhua.net/suricata_localrules#5200103;url,doc.emergingthreats.net/2003492; classtype:trojan-activity; sid:2003492; rev:99;)

### [Baofeng 暴风影音](suricata_localrules#5200104)   
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"HFF INFO Baofeng"; flow:to_server,established; priority:1; content:".baofeng.com"; http_host; content:"/conf/proxyd.xml";http_uri; reference:url,puhua.net/suricata_localrules#5200104; classtype:trojan-activity; sid:5200104; rev:1;)

### [Duba 毒霸](suricata_localrules#5200105)   
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"HFF INFO Duba"; flow:to_server,established; priority:1; content:".duba.net"; http_header; reference:url,puhua.net/suricata_localrules#5200105; classtype:trojan-activity; sid:5200105; rev:1;)


