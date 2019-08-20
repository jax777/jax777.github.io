---
layout: post
title: 开源ids ，snort and Suricata ,简介，端口扫描检测逻辑
categories: 渗透测试
tag: IDS
---

# 开源IDS
> https://www.aldeid.com/wiki/Suricata-vs-snort

知乎发现 raul17 的译文
> https://zhuanlan.zhihu.com/p/34329072

rules下载
> https://rules.emergingthreats.net/OPEN_download_instructions.html
## snort

> https://www.snort.org/documents
> https://www.snort.org/rule-docs
> http://manual-snort-org.s3-website-us-east-1.amazonaws.com/ 
> https://www.cnblogs.com/HacTF/p/7992787.html

- Sniffer mode, which simply reads the packets off of the network and displays them for you in a continuous stream on the console (screen).
- Packet Logger mode, which logs the packets to disk. 
- Network Intrusion Detection System (NIDS) mode, which performs detection and analysis on network traffic. This is the most complex and configurable mode. 

NIDS 模式下，可以分析网络流量，检测出各种不同的攻击方式，对攻击进行报警。

Snort的结构由4大软件模块组成，它们分别是：
1. 数据包嗅探模块
2. 预处理模块——该模块用相应的插件来检查原始数据包，从中发现原始数据的“行为”，如端口扫描，IP碎片等，数据包经过预处理后才传到检测引擎；
3. 检测模块——当数据包从预处理器送过来后，检测引擎依据预先设置的规则检查数据包，一旦发现数据包中的内容和某条规则相匹配，就通知报警模块；
4. 报警/日志模块

### 预处理器
**以下参考 SNORT Users Manual 2.9.13**

- Frag3 IP分片重组和攻击监测
- Session
- Stream
- sfPortscan  检测端口扫描 
- RPC Decode
- Performance Monitor
- HTTP Inspect
- SMTP Preprocessor
- POP Preprocessor
- IMAP Preprocessor
- FTP/Telnet Preprocessor
- SSH
- DNS
- SSL/TLS
- ARP Spoof Preprocessor
- DCE/RPC 2 Preprocessor
- Sensitive Data Preprocessor
- Normalizer
- SIP Preprocessor
- Reputation Preprocessor
- GTP Decoder and Preprocessor
- Modbus Preprocessor
- DNP3 Preprocessor
- AppId Preprocessor

### 规则语法

Snort使用一种简单的规则描述语言，这种描述语言易于扩展，功能也比较强大。Snort规则是基于文本的，规则文件按照不同的组进行分类。

| 类型           | 说明                                                                                          |
| -------------- | --------------------------------------------------------------------------------------------- |
| general        | 这些选项提供有关规则的信息，但在检测期间没有任何影响                                          |
| payload        | These options all look for data inside the packet payload and can be inter-related            |
| non-payload    | These options look for non-payload data      此类规则选项都是对数据包帧结构中特殊字段的匹配。 |
| post-detection | 这些选项是特定于规则的触发器，发生在规则“触发”之后。                                          |

每类规则提供了不同的body 规则选项 关键字

示例:`alert tcp any any -> 192.168.1.1 80 ( msg:"A ha!"; content:"attack"; sid:1; )`

结构: `action proto source dir dest ( body )`

1. action : 
   - alert 生成警报 
   - log 记录
   - pass 忽略
   - drop 阻塞并记录数据包
   - reject 阻塞并记录数据包，如果是 tcp 包则发送一个 TCP reset；如果是 udp 则发送一个 ICMP port unreachable message。
   - sdrop 阻塞数据包，但不记录
   - Activate and Dynamic rules are phased out in favor of a combination of tagging and flowbits 
2. proto
    ip, icmp, tcp, udp
3. source
   源地址
4. dir
   必须是如上所示的单向或<>所示的双向。
5. dest
   目的地址

**body 规则选项**

具体看 Users Manual 
- sid map
  sid这个关键字被用来识别snort规则的唯一性,map文件用来将sid 和 msg对应
  ![](/styles/images/2019-8/sidmap.jpg)
- content
  Snort重要的关键词之一。它规定在数据包的负载中搜索指定的样式。它的选项数据可以包含混合的文本和二进制数据。二进制数据一般包含在管道符号中“|”，表示为字节码（bytecode），也就是将二进制数据的十六进制形式。
    ```
    alert tcp any any -> any 139 (content:"|5c 00|P|00|I|00|P|00|E|00 5c|";)
    alert tcp any any -> any 80 (content:!“GET”;)
    ```
  -  Nocase            content字符串大小写不敏感
  -  rawbytes          直接匹配原始数据包
  -  Depth             匹配的深度
  -  Offset            开始匹配的偏移量
  -  Distance          两次content匹配的间距
  -  Within            两次content匹配之间至多的间距  
  -  http_cookie       匹配cookie
  -  http_raw_cookie   匹配未经normalize的cookie
  -  http_header       匹配header
  -  http_raw_header   匹配未经normalize的header
  -  http_method       匹配method
  -  http_url          匹配url
  -  http_raw_url      匹配日在未经normalize的url中
  -  http_stat_code    匹配状态码中匹配
  -  http_stat_msg     匹配状态信息
  -  http_encode       匹配编码格式

- pcre
  允许用户使用与PERL语言相兼容的正则表达式。
  `pcre:[!]"(/<regex>/|m<delim><regex><delim>)[ismxAEGRUBPHMCOIDKY]`
  `alert tcp any any -> any 80 (content:“/foo.php?id="; pcre:"/\/foo.php?id=[0-9]{1,10}/iU";) `

- rawbytes
  忽略解码器及预处理器的操作，直接匹配原始网络包。

- sid  snort id ,这个关键字被用来识别snort规则的唯一性（说的其实严禁，后面会有补充）
- gid 用是为了说明这条规则是snort的哪部分触发的。比如是由解码器、预处理器还是Snort自有规则等
  
- rev 这个关键字是被用来识别规则修改的版本，需要和sid,gid配合使用。 


### 端口扫描检测

#### sfPortscan 预处理器

攻击者预先并没有目标的信息，大多数攻击者发送的请求都会被拒绝（端口关闭）。在正常的网络通讯中，被拒绝的响应是稀少的，并且在一小段时间中出现大量拒绝响应更稀少。我们检测端口扫描的主要目的是检测和跟踪这些被拒绝的响应。

目前最广泛使用的端口扫描器是Nmap，sfPortscan 被设计用来检测Nmap产生的不同类型的扫描。
- Portscan 
  传统端口扫描，一个主机扫描另一主机的大量端口，大多数请求都被拒绝，因为只有部分开发端口
- Decoy(诱骗) Portscan
  攻击者有一个伪造的源地址与真实的扫描地址混杂在一起。这种策略有助于隐藏攻击者的真实身份。
- Distributed(分布式) Portscan 
  Negative queries will be distributed among scanning hosts, so we track this type of scan through the scanned host(通过被扫主机追踪这种行为？)
- Portsweep 
  一台主机扫描多个主机上的一个端口。
  portsweep 扫描的特性可能不会导致许多拒绝响应。例如，如果攻击者扫描 80端口，我们很可能不会看到很多拒绝响应。
- Filtered Portscan 报文无法到达指定的端口

配置选项
1. sense_level
   - low
     仅通过错误packets 生成，误报极少。设置的时间窗口为60秒，之后重新从统计、
   - medium 
     通过跟踪连接数产生警报，但在NAT、代理、dns缓存等地方可能产生误报。时间窗 90 秒
   - high
     持续监控，可以捕获一些慢速扫描。时间窗600秒
2. detect_ack_scans
   默认关闭
   This option will include sessions picked up in midstream by the stream module, which is necessary to detect ACK scans. However, this can lead to false alerts, especially under heavy load with dropped packets; which is why the option is off by default. 

src\preprocessors\portscan.h
src\preprocessors\portscan.c
src\preprocessors\spp_sfportscan.c
src\preprocessors\spp_sfportscan.h

根据 sense_level 查找相关代码
![](/styles/images/2019-8/snortport.jpg)

- 设置不同级别时间窗
  
  ```c
  static int ps_proto_update_window(PS_PROTO *proto, time_t pkt_time)
  {
      time_t interval;

      switch(portscan_eval_config->sense_level)
      {
          case PS_SENSE_LOW:
              //interval = 15;
              interval = 60;
              break;

          case PS_SENSE_MEDIUM:
              //interval = 15;
              interval = 90;
              break;

          case PS_SENSE_HIGH:
              interval = 600;
              break;

          default:
              return -1;
      }

  ```

- 规则阈值配置
  1. 结构体
      ```c
      typedef struct s_PS_ALERT_CONF
      {
          short connection_count;
          short priority_count;
          short u_ip_count;
          short u_port_count;

      } PS_ALERT_CONF;
      
      ```
      - connection_count
        onnection_count指明了当前时间段内在主机(src or dst)上有多少活跃的连接。该字段对于基于连接的协议(TCP)很准确，对于其它协议（UDP等），它是一个估计值。portscan是否被过滤可以用该字段进行辨别，如果connection_count较大，而priority_count较小，则表明portscan被过滤了。

      - priority_count
       记录”bad responses”（无效响应，如TCP RST, ICMP unreachable）. priority_count越大，说明捕获的无效响应包越多. 在判断扫描时 priority_count 是先于 connection_count进行判断的，它们俩是并列的，但是priority_count优先和阈值比较。
      ![优先比较](/styles/images/2019-8/priority-count.jpg)

      - u_ip_count
        u_ip_count记录着和主机最后进行通信的IP地址(last_ip)，如果新来一个数据包，其源IP地址src_ip，如果src_ip 不等于last_ip，就对u_ip_count字段加1。对于Portscan类型扫描，该值比较小；对于活跃的主机（和外界通信频繁），这个值会比较大，这样有可能导致portscan被检测成Distributed scan.
      - u_port_count
        u_port_count记录着和主机最后进行通信的端口（last_port），当新来的数据包的目的端口(dst_port)不等于last_port，那么对u_port_count加1.
      
  
  2. 配置
      ```c
      static int ps_alert_tcp(PS_PROTO *scanner, PS_PROTO *scanned)
      {
          static PS_ALERT_CONF *one_to_one;
          static PS_ALERT_CONF *one_to_one_decoy;
          static PS_ALERT_CONF *one_to_many;
          static PS_ALERT_CONF *many_to_one;

          /*
          ** Set the configurations depending on the sensitivity
          ** level.
          */
          switch(portscan_eval_config->sense_level)
          {
              case PS_SENSE_HIGH:
                  one_to_one       = &g_tcp_hi_ps;
                  one_to_one_decoy = &g_tcp_hi_decoy_ps;
                  one_to_many      = &g_tcp_hi_sweep;
                  many_to_one      = &g_tcp_hi_dist_ps;
      ......
      ```
      ```c
      /*
      **  Scanning configurations.  This is where we configure what the thresholds
      **  are for the different types of scans, protocols, and sense levels.  If
      **  you want to tweak the sense levels, change the values here.
      */
      /*
      **  TCP alert configurations
      */
      static PS_ALERT_CONF g_tcp_low_ps =       {0,5,25,5};
      static PS_ALERT_CONF g_tcp_low_decoy_ps = {0,15,50,30};
      static PS_ALERT_CONF g_tcp_low_sweep =    {0,5,5,15};
      static PS_ALERT_CONF g_tcp_low_dist_ps =  {0,15,50,15};

      static PS_ALERT_CONF g_tcp_med_ps =       {200,10,60,15};
      static PS_ALERT_CONF g_tcp_med_decoy_ps = {200,30,120,60};
      static PS_ALERT_CONF g_tcp_med_sweep =    {30,7,7,10};
      static PS_ALERT_CONF g_tcp_med_dist_ps =  {200,30,120,30};

      static PS_ALERT_CONF g_tcp_hi_ps =        {200,5,100,10};
      static PS_ALERT_CONF g_tcp_hi_decoy_ps =  {200,7,200,60};
      static PS_ALERT_CONF g_tcp_hi_sweep =     {30,3,3,10};
      static PS_ALERT_CONF g_tcp_hi_dist_ps =   {200,5,200,10};

      /*
      **  UDP alert configurations
      */
      static PS_ALERT_CONF g_udp_low_ps =       {0,5,25,5};
      static PS_ALERT_CONF g_udp_low_decoy_ps = {0,15,50,30};
      static PS_ALERT_CONF g_udp_low_sweep =    {0,5,5,15};
      static PS_ALERT_CONF g_udp_low_dist_ps =  {0,15,50,15};

      static PS_ALERT_CONF g_udp_med_ps =       {200,10,60,15};
      static PS_ALERT_CONF g_udp_med_decoy_ps = {200,30,120,60};
      static PS_ALERT_CONF g_udp_med_sweep =    {30,5,5,20};
      static PS_ALERT_CONF g_udp_med_dist_ps =  {200,30,120,30};

      static PS_ALERT_CONF g_udp_hi_ps =        {200,3,100,10};
      static PS_ALERT_CONF g_udp_hi_decoy_ps =  {200,7,200,60};
      static PS_ALERT_CONF g_udp_hi_sweep =     {30,3,3,10};
      static PS_ALERT_CONF g_udp_hi_dist_ps =   {200,3,200,10};

      /*
      **  IP Protocol alert configurations
      */
      static PS_ALERT_CONF g_ip_low_ps =        {0,10,10,50};
      static PS_ALERT_CONF g_ip_low_decoy_ps =  {0,40,50,25};
      static PS_ALERT_CONF g_ip_low_sweep =     {0,10,10,10};
      static PS_ALERT_CONF g_ip_low_dist_ps =   {0,15,25,50};

      static PS_ALERT_CONF g_ip_med_ps =        {200,10,10,50};
      static PS_ALERT_CONF g_ip_med_decoy_ps =  {200,40,50,25};
      static PS_ALERT_CONF g_ip_med_sweep =     {30,10,10,10};
      static PS_ALERT_CONF g_ip_med_dist_ps =   {200,15,25,50};

      static PS_ALERT_CONF g_ip_hi_ps =         {200,3,3,10};
      static PS_ALERT_CONF g_ip_hi_decoy_ps =   {200,7,15,5};
      static PS_ALERT_CONF g_ip_hi_sweep =      {30,3,3,7};
      static PS_ALERT_CONF g_ip_hi_dist_ps =    {200,3,11,10};

      /*
      **  ICMP alert configurations
      */
      static PS_ALERT_CONF g_icmp_low_sweep =   {0,5,5,5};
      static PS_ALERT_CONF g_icmp_med_sweep =   {20,5,5,5};
      static PS_ALERT_CONF g_icmp_hi_sweep =    {10,3,3,5};

      static int ps_get_proto(PS_PKT *, int *);
      ```
- 扫描检测逻辑

    **以sense_level high 的 one_to_one 扫描即传统端口扫描为例**
    配置`static PS_ALERT_CONF g_tcp_hi_ps =        {200,5,100,10}`

    这里scanned 都是被扫描主机的统计。scanner 为攻击主机的信息。 

  1. 比较`scanned->priority_count >= 5// conf->priority_count`
     1. `scanned->u_ip_count < 100 //conf->u_ip_count `
         `&& scanned->u_port_count >= 10 //conf->u_port_count`
         600 秒时间窗内 错误包>=5，不同连接ip数<100，不同端口数>=10 即判断为扫描
  2. 比较`scanned->connection_count >= 200 //conf->connection_count`
     1. `scanned->u_ip_count < 100//conf->u_ip_count`
        ` && scanned->u_port_count >= 10//conf->u_port_count`
      
        600 秒时间窗内 活跃连接数>=200，不同连接ip数<100，不同端口数>=10 即判断为扫描=
  
  ```c

  static int ps_alert_one_to_one(PS_PROTO *scanner, PS_PROTO *scanned,
          PS_ALERT_CONF *conf, int proto)
  {
      int action;
      if(!conf)
          return -1;

      /*
      **  Let's evaluate the scanned host.
      */

      if(scanned)
      {
          if(scanned->priority_count >= conf->priority_count)
          {
              action = ps_get_rule_action(proto, PS_ALERT_ONE_TO_ONE);
              if ((action == RULE_TYPE__DROP) ||
                  (action == RULE_TYPE__SDROP) ||
                  (action == RULE_TYPE__REJECT) ||
                  (!scanned->alerts))
              {
                  if(scanned->u_ip_count < conf->u_ip_count &&
                      scanned->u_port_count >= conf->u_port_count)
                  {
                      if(scanner)
                      {
                          if(scanner->priority_count >= conf->priority_count)
                          {
                              /*
                              **  Now let's check to make sure this is one
                              **  to one
                              */
                              scanned->alerts = PS_ALERT_ONE_TO_ONE;
                              return 0;
                          }
                      }
                      else
                      {
                          /*
                          **  If there is no scanner, then we do the best we can.
                          */
                          scanned->alerts = PS_ALERT_ONE_TO_ONE;
                          return 0;
                      }
                  }
              }
          }
          if(scanned->connection_count >= conf->connection_count)
          {
              action = ps_get_rule_action(proto, PS_ALERT_ONE_TO_ONE_FILTERED);
              if ((action == RULE_TYPE__DROP) ||
                  (action == RULE_TYPE__SDROP) ||
                  (action == RULE_TYPE__REJECT) ||
                  (!scanned->alerts))
              {
                  if(conf->connection_count == 0)
                      return 0;

                  if(scanned->u_ip_count < conf->u_ip_count &&
                    scanned->u_port_count >= conf->u_port_count)
                  {
                      scanned->alerts = PS_ALERT_ONE_TO_ONE_FILTERED;
                      return 0;
                  }
              }
          }
      }

      return 0;

  }
  ```

  **sense_level high 的 many_to_one 扫描即 distributed 分布式扫描**
  配置 `static PS_ALERT_CONF g_tcp_hi_dist_ps =   {200,5,200,10};`
  1. 比较`scanned->priority_count >= 5// conf->priority_count`
     1. `scanned->u_ip_count >= 200 //conf->u_ip_count `
         `&& scanned->u_port_count <= 10 //conf->u_port_count`
         600 秒时间窗内 错误包>=5，不同连接ip数>=200，不同端口数<=10 即判断为分布式扫描
  2. 比较`scanned->connection_count >= 200 //conf->connection_count`
     1. `scanned->u_ip_count >= 200//conf->u_ip_count`
        ` && scanned->u_port_count <= 10//conf->u_port_count`
      
        600 秒时间窗内 活跃连接数>=200，不同连接ip数>=200，不同端口数<=10 即判断为分布式扫描

  **sense_level high 的 one_to_many 扫描即portsweep**
  配置 `static PS_ALERT_CONF g_tcp_hi_sweep =     {30,3,3,10};`
  必须有scanne的信息才能判断出portsweep
  1. 比较`scanner->priority_count >= 3// conf->priority_count`
     1. `scanner->u_ip_count >= 3 //conf->u_ip_count `
         `&& scanner->u_port_count <= 10 //conf->u_port_count`
         600 秒时间窗内 扫描源 错误包>=3，不同连接ip数>=3，不同端口数<=10 即判断为portsweep
  2. 比较`scanner->connection_count >= 30 //conf->connection_count`
     1. `scanner->u_ip_count >= 3//conf->u_ip_count`
        ` && scanner->u_port_count <= 10//conf->u_port_count`
        600 秒时间窗内 扫描源 活跃连接数>=30，不同连接ip数>=3，不同端口数<=10 即判断为portsweep


参考
> https://onestraw.github.io/snort/sfportscan-addon-detect-portscan/


## Suricata

suricata是一款开源高性能的入侵检测系统，并支持ips（入侵防御）与nsm（网络安全监控）模式，用来替代原有的snort入侵检测系统，完全兼容snort规则语法和支持lua脚本。

> https://redmine.openinfosecfoundation.org/projects/suricata/wiki

### 规则语法

兼容snort 规则
![示例](/styles/images/2019-8/suricatarule.png)

[开源规则库https://github.com/ptreesearch/AttackDetection](https://github.com/ptresearch/AttackDetection)

### 端口扫描检测

Suricata 中没有类似sfPortscan的预处理器，检测端口扫描依靠规则实现。
![](/styles/images/2019-8/suricata-nmap.jpg)

emergingthreats 中关于nmap的rules

```
suricata emerging.rules\emerging-deleted.rules
  337,163: #alert tcp any any -> $HOME_NET any (msg:"ET DELETED Pitbull IRCbotnet Commands"; flow:from_server,established; content:"PRIVMSG|20|"; pcre:"/PRIVMSG.*@(portscan|nmap|back|udpflood|tcpflood|httpflood|linuxhelp|rfi|system|milw0rm|logcleaner|sendmail|join|part|help)/i";  reference:url,en.wikipedia.org/wiki/IRC_bot; reference:url,doc.emergingthreats.net/2007625; classtype:trojan-activity; sid:2007625; rev:6; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  789,93: #alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET DELETED EXE Using Suspicious IAT NtUnmapViewOfSection Possible Malware Process Hollowing"; flowbits:isset,ET.http.binary; flow:established,to_client; content:"NtUnmapViewOfSection"; nocase; fast_pattern:only; reference:url,blog.spiderlabs.com/2011/05/analyzing-malware-hollow-processes.html; reference:url,sans.org/reading_room/whitepapers/malicious/rss/_33649; classtype:bad-unknown; sid:2012817; rev:4; metadata:created_at 2011_05_18, updated_at 2011_05_18;)
  789,219: #alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET DELETED EXE Using Suspicious IAT NtUnmapViewOfSection Possible Malware Process Hollowing"; flowbits:isset,ET.http.binary; flow:established,to_client; content:"NtUnmapViewOfSection"; nocase; fast_pattern:only; reference:url,blog.spiderlabs.com/2011/05/analyzing-malware-hollow-processes.html; reference:url,sans.org/reading_room/whitepapers/malicious/rss/_33649; classtype:bad-unknown; sid:2012817; rev:4; metadata:created_at 2011_05_18, updated_at 2011_05_18;)

suricata emerging.rules\emerging-scan.rules
  65,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sS window 2048"; fragbits:!D; dsize:0; flags:S,12; ack:0; window:2048; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000537; classtype:attempted-recon; sid:2000537; rev:8; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  67,60: #alert ip $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sO"; dsize:0; ip_proto:21; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000536; classtype:attempted-recon; sid:2000536; rev:7; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  69,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sA (1)"; fragbits:!D; dsize:0; flags:A,12; window:1024; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000538; classtype:attempted-recon; sid:2000538; rev:8; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  71,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sA (2)"; fragbits:!D; dsize:0; flags:A,12; window:3072; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000540; classtype:attempted-recon; sid:2000540; rev:8; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  73,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -f -sF"; fragbits:!M; dsize:0; flags:F,12; ack:0; window:2048; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000543; classtype:attempted-recon; sid:2000543; rev:7; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  75,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -f -sN"; fragbits:!M; dsize:0; flags:0,12; ack:0; window:2048; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000544; classtype:attempted-recon; sid:2000544; rev:7; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  77,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -f -sX"; fragbits:!M; dsize:0; flags:FPU,12; ack:0; window:2048; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000546; classtype:attempted-recon; sid:2000546; rev:7; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  213,68: #alert icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"GPL SCAN PING NMAP"; dsize:0; itype:8; reference:arachnids,162; classtype:attempted-recon; sid:2100469; rev:4; metadata:created_at 2010_09_23, updated_at 2010_09_23;)
  247,65: alert http $EXTERNAL_NET any -> $HTTP_SERVERS any (msg:"ET SCAN NMAP SQL Spider Scan"; flow:established,to_server; content:"GET"; http_method; content:" OR sqlspider"; http_uri; reference:url,nmap.org/nsedoc/scripts/sql-injection.html; classtype:web-application-attack; sid:2013778; rev:2; metadata:created_at 2011_10_19, updated_at 2011_10_19;)
  247,193: alert http $EXTERNAL_NET any -> $HTTP_SERVERS any (msg:"ET SCAN NMAP SQL Spider Scan"; flow:established,to_server; content:"GET"; http_method; content:" OR sqlspider"; http_uri; reference:url,nmap.org/nsedoc/scripts/sql-injection.html; classtype:web-application-attack; sid:2013778; rev:2; metadata:created_at 2011_10_19, updated_at 2011_10_19;)
  323,62: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"GPL SCAN nmap TCP"; ack:0; flags:A,12; flow:stateless; reference:arachnids,28; classtype:attempted-recon; sid:2100628; rev:8; metadata:created_at 2010_09_23, updated_at 2010_09_23;)
  325,62: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"GPL SCAN nmap XMAS"; flow:stateless; flags:FPU,12; reference:arachnids,30; classtype:attempted-recon; sid:2101228; rev:8; metadata:created_at 2010_09_23, updated_at 2010_09_23;)
  327,62: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"GPL SCAN nmap fingerprint attempt"; flags:SFPU; flow:stateless; reference:arachnids,05; classtype:attempted-recon; sid:2100629; rev:7; metadata:created_at 2010_09_23, updated_at 2010_09_23;)
  361,61: alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine)"; flow:to_server,established; content:"Mozilla/5.0 (compatible|3b| Nmap Scripting Engine"; nocase; http_user_agent; depth:46; reference:url,doc.emergingthreats.net/2009358; classtype:web-application-attack; sid:2009358; rev:5; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  361,104: alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine)"; flow:to_server,established; content:"Mozilla/5.0 (compatible|3b| Nmap Scripting Engine"; nocase; http_user_agent; depth:46; reference:url,doc.emergingthreats.net/2009358; classtype:web-application-attack; sid:2009358; rev:5; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  361,194: alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine)"; flow:to_server,established; content:"Mozilla/5.0 (compatible|3b| Nmap Scripting Engine"; nocase; http_user_agent; depth:46; reference:url,doc.emergingthreats.net/2009358; classtype:web-application-attack; sid:2009358; rev:5; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  415,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sS window 1024"; fragbits:!D; dsize:0; flags:S,12; ack:0; window:1024; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2009582; classtype:attempted-recon; sid:2009582; rev:3; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  417,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sS window 3072"; fragbits:!D; dsize:0; flags:S,12; ack:0; window:3072; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2009583; classtype:attempted-recon; sid:2009583; rev:3; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  419,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sS window 4096"; fragbits:!D; dsize:0; flags:S,12; ack:0; window:4096; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2009584; classtype:attempted-recon; sid:2009584; rev:2; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  421,68: alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (msg:"ET SCAN NMAP SIP Version Detect OPTIONS Scan"; flow:established,to_server; content:"OPTIONS sip|3A|nm SIP/"; depth:19; classtype:attempted-recon; sid:2018317; rev:1; metadata:created_at 2014_03_25, updated_at 2014_03_25;)
  423,66: alert tcp $EXTERNAL_NET any -> $HOME_NET 5060:5061 (msg:"ET SCAN NMAP SIP Version Detection Script Activity"; content:"Via|3A| SIP/2.0/TCP nm"; content:"From|3A| <sip|3A|nm@nm"; within:150; fast_pattern; classtype:attempted-recon; sid:2018318; rev:1; metadata:created_at 2014_03_25, updated_at 2014_03_25;)
  427,61: #alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -f -sV"; fragbits:!M; dsize:0; flags:S,12; ack:0; window:2048; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000545; classtype:attempted-recon; sid:2000545; rev:8; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  429,66: alert udp $EXTERNAL_NET 10000: -> $HOME_NET 10000: (msg:"ET SCAN NMAP OS Detection Probe"; dsize:300; content:"CCCCCCCCCCCCCCCCCCCC"; fast_pattern:only; content:"CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC"; depth:255; content:"CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC"; within:45; classtype:attempted-recon; sid:2018489; rev:3; metadata:created_at 2014_05_20, updated_at 2014_05_20;)
  449,61: alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap NSE)"; flow:to_server,established; content:"Nmap NSE"; http_user_agent; reference:url,doc.emergingthreats.net/2009359; classtype:web-application-attack; sid:2009359; rev:4; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  449,104: alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap NSE)"; flow:to_server,established; content:"Nmap NSE"; http_user_agent; reference:url,doc.emergingthreats.net/2009359; classtype:web-application-attack; sid:2009359; rev:4; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  449,153: alert http $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap NSE)"; flow:to_server,established; content:"Nmap NSE"; http_user_agent; reference:url,doc.emergingthreats.net/2009359; classtype:web-application-attack; sid:2009359; rev:4; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
  507,50: alert tcp any any -> $HOME_NET any (msg:"ET SCAN Nmap NSE Heartbleed Request"; flow:established,to_server; content:"|18 03|"; depth:2; byte_test:1,<,4,2; content:"|01|"; offset:5; depth:1; byte_test:2,>,2,3; byte_test:2,>,200,6; content:"|40 00|Nmap ssl-heartbleed"; fast_pattern:2,19; classtype:attempted-recon; sid:2021023; rev:1; metadata:created_at 2015_04_28, updated_at 2015_04_28;)
  507,246: alert tcp any any -> $HOME_NET any (msg:"ET SCAN Nmap NSE Heartbleed Request"; flow:established,to_server; content:"|18 03|"; depth:2; byte_test:1,<,4,2; content:"|01|"; offset:5; depth:1; byte_test:2,>,2,3; byte_test:2,>,200,6; content:"|40 00|Nmap ssl-heartbleed"; fast_pattern:2,19; classtype:attempted-recon; sid:2021023; rev:1; metadata:created_at 2015_04_28, updated_at 2015_04_28;)
  509,50: alert tcp $HOME_NET any -> any any (msg:"ET SCAN Nmap NSE Heartbleed Response"; flow:established,from_server; content:"|18 03|"; depth:2; byte_test:1,<,4,2; byte_test:2,>,200,3; content:"|40 00|Nmap ssl-heartbleed"; fast_pattern:2,19; classtype:attempted-recon; sid:2021024; rev:1; metadata:created_at 2015_04_28, updated_at 2015_04_28;)
  509,195: alert tcp $HOME_NET any -> any any (msg:"ET SCAN Nmap NSE Heartbleed Response"; flow:established,from_server; content:"|18 03|"; depth:2; byte_test:1,<,4,2; byte_test:2,>,200,3; content:"|40 00|Nmap ssl-heartbleed"; fast_pattern:2,19; classtype:attempted-recon; sid:2021024; rev:1; metadata:created_at 2015_04_28, updated_at 2015_04_28;)
  587,60: alert http $HOME_NET any -> any any (msg:"ET SCAN Possible Nmap User-Agent Observed"; flow:to_server,established; content:"|20|Nmap"; http_user_agent; fast_pattern; metadata: former_category SCAN; classtype:web-application-attack; sid:2024364; rev:3; metadata:affected_product Any, attack_target Client_and_Server, deployment Perimeter, signature_severity Audit, created_at 2017_06_08, performance_impact Low, updated_at 2017_06_13;)
  587,128: alert http $HOME_NET any -> any any (msg:"ET SCAN Possible Nmap User-Agent Observed"; flow:to_server,established; content:"|20|Nmap"; http_user_agent; fast_pattern; metadata: former_category SCAN; classtype:web-application-attack; sid:2024364; rev:3; metadata:affected_product Any, attack_target Client_and_Server, deployment Perimeter, signature_severity Audit, created_at 2017_06_08, performance_impact Low, updated_at 2017_06_13;)
```

解释
`alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"ET SCAN NMAP -sS window 2048"; fragbits:!D; dsize:0; flags:S,12; ack:0; window:2048; threshold: type both, track by_dst, count 1, seconds 60; reference:url,doc.emergingthreats.net/2000537; classtype:attempted-recon; sid:2000537; rev:8; metadata:created_at 2010_07_30, updated_at 2010_07_30;)`

- fragbits 检查ip头的分片标志位 !D
- dsize 检查包的数据部分大小 0
- flags 检查tcp flags的值 S,12
  保留位“1”和“2”分别用“C”和“E”代替，以匹配RFC 3168，“向IP添加显式拥塞通知（ECN）”。 “1”和“2”的旧值对于flag关键字仍然有效，但现在已弃用。

| flag | 说明                                                                             |
| ---- | -------------------------------------------------------------------------------- |
| F    | FIN - Finish (LSB in TCP Flags byte)                                             |
| S    | SYN - Synchronize sequence numbers                                               |
| R    | RST - Reset                                                                      |
| P    | PSH - Push                                                                       |
| A    | ACK - Acknowledgment                                                             |
| U    | URG - Urgent                                                                     |
| C    | CWR - Congestion Window Reduced (MSB in TCP Flags byte)                          |
| E    | ECE - ECN-Echo (If SYN, then ECN capable. Else, CE flag in IP header is set)     |
| 0    | No TCP Flags Set The ollowing modifiers can be set to change the match criteria: |
| 2    | Reserved bit 2                                                                   |
| 1    | Reserved bit 1 (MSB in TCP Flags byte)                                           |
| +    | match on the specified bits, plus any others                                     |
| *    | match if any of the specified bits are set                                       |
| !    | match if the specified bits are not set                                          |
- ack 检查tcp应答（acknowledgement）的值 0
- window 关键字用于检查特定的TCP窗口大小。 2048
- threshold: type both, track by_dst, count 1, seconds 60 
  60 秒记录一次

**可以尝试修改下nmap中的特征，可以过部分ids检测。snort 的sfPortscan 尝试下时间对抗，提高间隔，防止触发阈值。600秒时间窗口，10个端口**
