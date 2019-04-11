---
layout: post
title: Linux OOM killer
categories: linux
tag: linux
---

# 说明
我的程序为何惨死街头，君请看oom killer
卡尔萨斯裸死街头究竟何人所为？阿利斯塔半夜惨叫背后隐藏着什么？蒙多究竟要去哪里？艾希到底要来几发？安妮的小熊找到没有？易大师的剑究竟是谁的剑？

# 关于oom killer

**Out Of Memory killer**
看名字就知道只是一个在系统内存耗尽的情况下跳出来杀进程的功能，当然不是随便杀进程的，会有一些优先级。


OOM Killer 会审查所有正在运行的进程，并给他们分配一个成绩。得分最高的进程就会被杀。、基本标准如下：
- 进程及其所有子进程都使用了大量内存
- 尽量少杀死进程，以便释放足够的内存来解决情况
- Root, kernel and important system processes 尽量避开，会给他们一个较低的分数

每个进程的分数 都在 这个文件里 /proc/$(pid)/oom_score

# 查看oom killer 的杀手记录
使用dmesg 查看
`dmesg | grep -i “killed process”`

结果类似这样
`host kernel: Out of Memory: Killed process 1620 (python).`

# 如何防止进程被杀
- 程序少用点内存吧。。。
- 机器太差？ 给必须的进程给个后门，禁止被杀
	给这个值`/proc/$(pid)/oom_adj` 设为-17 
	` echo -17 > /proc/$(pid)/oom_adj`
- 修改内存分配策略（不推荐，有时内存暴涨，会出现无法运行新进程的情况）
	`/proc/sys/vm/overcommit_memory`可用于配置这种策略：
	- `overcommit_memory == 2`，物理内存使用完后，打开任意一个程序均显示内存不足；此时 oom-killer 不会再工作
	- `overcommit_memory == 1`，会从buffer中释放较多物理内存，oom-kill也会继续起作用；
	- `overcommit_memory ==0`，系统默认设置，释放较少物理内存，使得oom-kill机制运作比较明显。
- 直接关闭 oom killer （不推荐，有时内存暴涨，会出现无法运行新进程的情况）
	`sysctl -w vm.panic_on_oom=1`
- **究极办法 ，换电脑，上大内存，远离512m内存vps**

# 参考 
- https://linux-mm.org/OOM_Killer
- http://www.memset.com/docs/additional-information/oom-killer/
- http://blog.csdn.net/fjssharpsword/article/details/9341563