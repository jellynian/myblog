---
layout: post
title: "Ucloud配置haproxy+keepalived(利用ucloud-API实现外网浮动IP切换)"
description: ""
key: 1
category: sysadmin
tags: [sysadmin]
last_updated: 2018年 03月 13日 星期二 19:05:52 CST
---

# 背景

先交代下事情发生的背景，公司在使用ucloud的负载均衡产品的过程中由于某种特殊的需求必须要自建外网负载均衡器，由于ucloud的云主机使用nat地址转化技术，外网IP的数据包被转发到了内网IP，而且外网IP是与云主机绑定的，所以就不能用传统的方式实现外网haproxy服务的高可用了，但是ucloud实现了切换公网IP的api，我们让keepalived在另一台机器挂掉的时候调用api来将该机器的外网浮动IP解除绑定，然后绑定到自己身上，api切换的时间极短保证了线上负载均衡器故障能够迅速切换。


# 安装keepalived 

    sudo apt update 
    sudo apt install keepalived -y

# 准备脚本

## 下载ucloud sdk

    git clone https://github.com/ucloud/ucloud-sdk-python.git /etc/keepalived/script #下载ucloud python sdk 到 /etc/keepalived/script目录
    cd /etc/keepalived/script  # 进入 script 目录 
    cp config.simple.py config.py   # 复制config.simple.py模板文件

## 编写脚本
修改 config.py 文件，



    #配置公私钥"""
    public_key  = "" #你的公钥
    private_key = "" #你的私钥
    project_id = "" # 项目ID 请在Dashbord 上获取

    """
    #添加以下内容
    EIPID="eip-lb011e"  #你需要浮动的EIP的ID
    MYUHOSTID="uhost-33eitwa"  #脚本部署在这台机器的ID，可在控制台上查看 注意，你脚本部署在哪台机器这个ID就是哪台机器的ID
    PEERUHOSTID="uhost-333xvs" # 另一台机器的ID 
    REGION="hk"  #地区 Region 可在ucloud官网查阅

接下来修改 bind_eip.py 将Parameters段参照以下内容修改

    Parameters={
        "Action":"BindEIP",
        "EIPId":EIPID,
        "ResourceId":MYUHOSTID,
        "ResourceType":"uhost",
        "Region":REGION
    }

修改 unbind_eip.py 将Parameters段参照以下内容修改

    Parameters={
        "Action":"UnBindEIP",
        "EIPId":EIPID,
        "ResourceId":PEERUHOSTID,
         "ResourceType":"uhost",
        "Region":REGION
    }

新建脚本  slave2master.sh

    #!/bin/bash
    python  /etc/keepalived/script/unbind_eip.py  #先把IP从故障的那台机器解绑
    python /etc/keepalived/script/bind_eip.py     # 再绑定到本机上


编写一个故障检查脚本 check_haproxy.sh


    #!/bin/bash
    GREP_OPTIONS=""
    Count1=`netstat -antp |grep -v grep |grep haproxy |wc -l`

    if [ $Count1 -eq 0  ]; then
    exit 1  #keepalived 调用这个脚本，当退出值不为0就认为本机故障
    fi
    exit 0



# 编写 keepalived的配置文件

## slave的配置文件参考 


    ! Configuration File for keepalived

    global_defs {
              router_id xxx-lb-test
    }

    vrrp_script chk_haproxy {
            script "/etc/keepalived/check_haproxy.sh" 
            interval 2 

    }

    vrrp_instance VI_1 {

        unicast_peer {
            10.8.100.13    !另一台机器(master)的IP                  
        }
            state BACKUP   !默认进入backup模式
            interface eth0
            virtual_router_id 194
            priority 10
            weight 5
            !    nopreempt
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass e8qvAwPTWgCEepUKCQ4tN
            }
            track_script {
                       chk_haproxy                   
            }
                notify_master /etc/keepalived/script/slave2master.sh   #一旦master出现故障该脚本就会被执行

    }


## master的配置文件 参考


    ! Configuration File for keepalived

    global_defs {
        router_id xxx-lb-test
    }

    vrrp_script chk_haproxy {
        script "/etc/keepalived/check_haproxy.sh" 
        interval 2 
    }



    vrrp_instance VI_1 {

        unicast_peer {
            10.8.126.62  # Slave 的IP            
        }
        state MASTER #默认为 MASTER
        interface eth0
        virtual_router_id 194
        priority 100
        !    nopreempt
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass e8qvAwPTWgCEepUKCQ4tN
            }
        track_script {
               chk_haproxy
        }
        notify_master /etc/keepalived/script/slave2master.sh #一旦slave出现故障该脚本就会被执行
    }

