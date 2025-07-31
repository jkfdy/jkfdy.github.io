---
title: 面向新手的Git教程：打不开GitHub（三）？
date: 2025-07-31 11:00:00 +0800
categories: [Git]
tags: [Git, GitHub]
---

> 温馨提示：本文包含众多非微信公众平台超链接，为达到更好的阅读体验，请点击文字底部“阅读原文”按钮阅读本文网页版本。

前面已经写了[《面向新手的Git教程：打不开GitHub（一）》](https://jkfdy.scfhao.cn/posts/access-github/)和[《面向新手的Git教程：打不开GitHub（二）》](https://jkfdy.scfhao.cn/posts/access-github-2/)。其中在“一”中介绍了通过修改系统的`/etc/hosts`文件来达到可以访问 GitHub.com 的原理，在“二”中分析了一下大家评论中提到的“GitHub520”项目的致命缺陷，如果没有看过前两篇的建议先读前两篇。

我们知道，GitHub.com 在全球有很多 IP，在指出“GitHub520”项目的不足后，我一直在问自己，那如何获取 GitHub.com 最适合我们的 IP 呢？所以今天我们就一起探讨一下这个问题。

比较`ping`不同 IP 的延时，找到延时最小的那个 IP 的思路是没有问题的。但这个思路有两个前提：

* `ping`命令需要在本地电脑上执行，在自己的网络环境中才能找到对自己延时最小的那个IP，不同网络运营商/不同地区的延时都不一样。
* 遍历的 IP 列表需要尽可能全面，要不然就可能会有延时更小的 IP 被漏掉。

## 获取最全的 GitHub IP 列表

GitHub 提供了一个[API](https://api.github.com/meta)返回 GitHub 用到的 IP 和域名（对于浏览器访问，我们着重关注其中的 web 服务 IP 列表）。

需要注意的是，这个 API 返回的是 CIDR 格式的 IP 网段，如`185.199.108.0/22`，一个网段代表很多个 IP（对 CIDR 网段不了解的可以找 AI 现学一下，很简单的）。

## 如何判断一个 IP 是否可用

并不是这个 API 返回的任何一个 IP 都能用，在拿到一个 IP 后，我们最好是先访问一下，看一下它的响应是否是 GitHub.com 的网页。

想做到这一点很简单，在 HTTP 协议中，HTTP 请求头中会有一个`Host`请求头用于向服务器表明想要访问的服务器域名，与之对应 HTTP 响应头中有一个`Server`响应头用于向客户端表明自己是哪个服务器。

如果使用 https 访问此 IP 地址，并且服务器返回的 Server 响应头的内容是`github.com`，那么这个 IP 就是可用的了。

找到可用的 IP 列表后，接下来就可以 ping 这些 IP 了，找到延时最小的那一个，然后设置到`/etc/hosts`文件中即可。

## 自动化

程序员怎么能手动执行这些操作呢？当然要写一个脚本出来，不会写脚本也无所谓，让 AI 来帮忙写。

把思路给了 AI 后，让 AI 帮忙写一个 Shell 脚本，比如下面这个（由 Trae 和 DeepSeek 生成）。

```bash
#!/bin/bash

# 从 https://api.github.com/meta 获取web ip列表
web_ips=$(curl -s https://api.github.com/meta | jq -r '.web[]')

server_name="github.com"

best_ip=""
best_ping_time=999

# 遍历IP列表
for ip in $web_ips; do
    # 判断IP是否为ipv6格式，如果是则跳过本次循环
    if [[ $ip == *":"* ]]; then
        continue
    fi
    # 检查是否为CIDR格式（网段）
    if [[ $ip == *"/"* ]]; then
        # 记录该网段连续失败 IP 数量
        fail_count=0
        # 使用进程替换避免子shell问题
        while read -r single_ip; do
            echo "处理网段($ip)内IP: $single_ip"
            # 这里可以添加对网段内单个IP的处理逻辑
            # 使用 curl 和 https 协议访问此 IP，忽略证书错误，并打印响应的 location 响应头
            server_header=$(curl -sI -k -H "Host: $server_name" --max-time 2 "https://$single_ip" | grep -i '^Server:' | tr -d '\r' | cut -d' ' -f2-)
            # 判断 server_header 是否不为空且第一个元素是否等于 server_name
            if [[ -n "$server_header" && "$server_header" != "null" ]]; then
                # 重置失败计数
                fail_count=0
                if [[ "$server_header" == "$server_name" ]]; then
                    # 对单个IP执行ping操作，发送4个ICMP包，获取平均延时(macOS兼容)
                    ping_result=$(ping -c 4 "$single_ip" | awk '/round-trip/ {print $4}' | cut -d '/' -f 2)
                    if [[ -z "$ping_result" ]]; then
                        # 备用提取方式
                        ping_result=$(ping -c 4 "$single_ip" | tail -1 | awk '{print $4}' | cut -d '/' -f 2)
                    fi
                    # 判断 ping_result 是否为有效数值
                    if [[ -n "$ping_result" && "$ping_result" =~ ^[0-9]+(\.[0-9]+)?$ ]]; then
                        # 将 ping_result 转换为浮点数进行比较
                        if (( $(echo "$ping_result < $best_ping_time" | bc -l) )); then
                            best_ping_time=$ping_result
                            best_ip=$single_ip
                            echo "发现更好的IP: $best_ip, 平均延时: $best_ping_time ms"
                        else
                            echo "IP $single_ip 平均延时: $ping_result ms, 高于当前最佳IP: $best_ip, 平均延时: $best_ping_time ms"
                        fi
                    else
                        echo "ping $single_ip 平均延时不是数值 $ping_result"
                    fi
                else
                    echo "IP $single_ip 返回的 server 响应头是$server_header"
                fi
            else
                echo "IP $single_ip 没有返回 server 响应头"
                # 增加失败计数
                ((fail_count++))
                # 检查失败计数是否超过阈值
                if [[ $fail_count -gt 3 ]]; then
                    echo "网段 $ip 内连续失败 IP 数量超过3个,跳过"
                    break
                fi
            fi
        done < <(nmap -sL "$ip" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+')
    fi
done

echo "本次扫描最佳IP: $best_ip, 平均延时: $best_ping_time ms"
```

执行结果如下：

```bash
...省略
处理网段(140.82.112.0/20)内IP: 140.82.112.3
248.348 < 999
发现更好的IP: 140.82.112.3, 平均延时: 248.348 ms
...省略
处理网段(140.82.112.0/20)内IP: 140.82.112.0
IP 140.82.112.0 没有返回 server 响应头
处理网段(140.82.112.0/20)内IP: 140.82.112.1
IP 140.82.112.1 没有返回 server 响应头
处理网段(140.82.112.0/20)内IP: 140.82.112.2
IP 140.82.112.2 返回的 server 响应头是nginx
处理网段(140.82.112.0/20)内IP: 140.82.112.3
发现更好的IP: 140.82.112.3, 平均延时: 248.348 ms
处理网段(140.82.112.0/20)内IP: 140.82.112.4
IP 140.82.112.4 平均延时: 274.510 ms, 高于当前最佳IP: 140.82.112.3, 平均延时: 248.348 ms
处理网段(140.82.112.0/20)内IP: 140.82.112.5
IP 140.82.112.5 没有返回 server 响应头
处理网段(140.82.112.0/20)内IP: 140.82.112.6
IP 140.82.112.6 没有返回 server 响应头
处理网段(140.82.112.0/20)内IP: 140.82.112.7
IP 140.82.112.7 没有返回 server 响应头
处理网段(140.82.112.0/20)内IP: 140.82.112.8
IP 140.82.112.8 没有返回 server 响应头
网段 140.82.112.0/20 内连续失败 IP 数量超过3个,跳过
...省略
处理网段(20.26.156.215/32)内IP: 20.26.156.215
IP 20.26.156.215 平均延时: 287.452 ms, 高于当前最佳IP: 20.27.177.113, 平均延时: 83.977 ms
本次扫描最佳IP: 20.27.177.113, 平均延时: 83.977 ms
```

我也把这个脚本上传到[GitHub](https://github.com/scfhao/GitHubIPChecker)了，大家可以试用一下：）
