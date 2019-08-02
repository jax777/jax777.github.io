---
layout: post
title: snort and Suricata 规则语法
categories: 渗透测试
tag: IDS
---

# 了解一些开源IDS

## snort

> https://www.snort.org/documents
> https://www.snort.org/rule-docs

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
>http://manual-snort-org.s3-website-us-east-1.amazonaws.com/ 

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


### 端口扫描检测\规则

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
  
  one_to_one 扫描即传统端口扫描为例；
  1. 比较`scanned->priority_count >= conf->priority_count`
     1. `scanned->u_ip_count < conf->u_ip_count && scanned->u_port_count >= conf->u_port_count`
     2. 
  2. 比较`scanned->connection_count >= conf->connection_count`
     1. `scanned->u_ip_count < conf->u_ip_count && scanned->u_port_count >= conf->u_port_count`
      报警 `PS_ALERT_ONE_TO_ONE_FILTERED`

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

参考
> https://onestraw.github.io/snort/sfportscan-addon-detect-portscan/


## Suricata

suricata是一款开源高性能的入侵检测系统，并支持ips（入侵防御）与nsm（网络安全监控）模式，用来替代原有的snort入侵检测系统，完全兼容snort规则语法和支持lua脚本。

### 规则语法

### 端口扫描检测规则
