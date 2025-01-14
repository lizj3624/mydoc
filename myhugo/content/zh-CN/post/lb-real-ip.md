---
title: "负载均衡获得真实源IP的6种方法"
date: 2022-09-23T14:01:33+08:00
tags:
   - LB 
   - nginx
categories:
   - LB 
   - nginx
toc: true
---

> 使用负载均衡后，获取真实源IP方法汇总, 本文转载[负载均衡获得真实源IP的6种方法](https://cloud.tencent.com/developer/article/1468108)

当数据包从负载均衡器往后端转发时候，真实源IP可在L3、L4、L7实现，并且分别有2种方法可以获得真实IP，因此共有6种方法:

**保持L3层源IP不变，根据连接次数可以分为**

- 一次连接模式，如lvs
- 二次连接模式，如haproxy的透明模式

**在L4层数据里，添加源IP信息，有2种模式**

- 在4层的option字段里增加源IP信息，比如tcp option、udp option

- 在4层末尾和7层开头之间，增加proxy protocol信息

**在L7层数据里，增加源IP信息，有2种模式**

- 协议自带，例如HTTP的X-FORWARD-FOR
- 业务程序自行实现

**一次连接与二次连接**

- 一次连接：负载均衡器对数据包仅做转发，而不对后端重新发起三次握手

- 二次连接：和一次连接相对应，在tcp转发时候，对后端重新进行了三次握手。上面所讲的L4和L7方法的负载均衡，都是二次连接

可以通过对比源端口是否有改变来简单判断是一次连接还是二次连接，端口没改变，可以理解为一次连接，有改变就是二次连接

**1. 方法1: L3的一次连接模式**

介绍：是指在网络层不对源IP做修改，直接将数据包转发给后端，当后端接收到数据的时候，源IP就是真实IP。

实现：LVS-DR、LVS-NAT、LVS-TUNNEL模式。其中LVS-TUNNEL比较特别，是在原有数据包的开头封装了IP头，当后端收到数据的时候，将封装的IP头进行解封装，获得的就是原有数据包。

优点：逻辑简单，当负载均衡器故障切换的时候，从客户端到后端的tcp连接不会中断

缺点：对网络架构有要求，比如DR模式，要求后端配VIP，并且回包要能直接回到客户机；NAT模式，要求回包经过负载均衡器；TUNNEL模式要求后端支持IPIP隧道

**2. 方法2: L3的二次连接模式**

介绍：是指负载均衡器和后端重新进行三次握手，但保持数据包的源IP为真实IP。

实现：haproxy（开启tproxy透明代理模式）+ iptables（fwmark打标记）+ 策略路由这3者组合才能实现

优点：可以实现L7层（如HTTP/HTTPS）的负载均衡，而一次连接主要实现L4层的负载均衡

缺点：配置最复杂，同时要求回包经过负载均衡器

**3. 方法3: L4的toa模式**

介绍：在4层的option字段里增加源IP信息，比如在tcp option里增加源IP信息（称为toa）、udp option里增加源IP信息（称为uoa）

实现：负载均衡器配置lvs-fullnat（只支持toa），iqiyi/dpvs（支持toa和uoa）；后端要加载toa、uoa模块，这个模块的作用是替换了后端应用程序获取IP的系统接口的钩子

优点：对网络架构要求低

缺点：lvs-fullnat需要编译内核，且只支持rhel/centos 6的内核，另外多年未更新；iqiyi/dpvs功能强大，但需要dpdk的支持；后端都需要加载toa/uoa模块，且只支持linux系统。

**4. 方法4: L4的proxy protocol模式**

介绍：在L7层开头增加proxy protocol数据（该协议是haproxy发明），目前有v1（明文）和v2（二进制）版本

实现：负载均衡器和后端同时开启proxy protocol，haproxy和nginx均支持，不过haproxy的配置颗粒度更小，另外nginx转发数据时候只支持v1

优点：对网络架构要求低，只要程序支持即可，因此理论上没有操作系统限制

缺点：目前支持proxy protocol的程序较少，且两端只能都开启或者都不开启该协议，不能一端开一端不开

**5. 方法5: L7协议自带，例如HTTP的X-Forwarded-For

介绍：最常见的方法，协议自带源IP信息，或者可定制插入

实现：例如HTTP协议有X-FORWARD-FOR，也可以自己将源IP插入到HTTP头部信息里

优点：对网络架构要求低，配置简便

缺点：容易被伪造

**6. 方法6: L7层业务程序自行实现**

介绍：最好的方法，就是业务方自行实现

实现：业务自行实现，例如在client端插入源IP信息，带到server端

优点：对网络架构无要求，只要网络可通即可，只要安全做好，不容易被伪造

缺点：业务方要有一定开发能力

**总结**

如果能做到无状态，不需要真实源IP，是最好的。因为这样对底层架构没有要求，底层架构就可以做的更高级更弹性。

如果一定要获得真实源IP，推荐方案的顺序就是倒推，从L7到L3，即首选方法6，如果不行再选用方法5，以此类推到方法1
