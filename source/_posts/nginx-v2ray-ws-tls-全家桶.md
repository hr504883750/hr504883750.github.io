---
layout: _post
title: nginx + v2ray + ws + tls 全家桶
date: 2020-08-29 18:53:42
tags:
---



### nginx + v2ray + ws + tls 全家桶

---

**Nginx快速安装**

```shell
vi /etc/yum.repos.d/nginx.repo  # 进行源配置

# nginx.repo 中写入
--------------------
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/6/$basearch/
gpgcheck=0
enabled=1
--------------------
# centos 系统为7将其中6进行替换 
cat /etc/issue  # 查询系统版本

yum install nginx  # 进行安装下载

nginx -v # 查看版本

```

**V2ray安装**

* [一键脚本项目地址]([https://github.com/233boy/v2ray/wiki/V2Ray%E4%B8%80%E9%94%AE%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC](https://github.com/233boy/v2ray/wiki/V2Ray一键安装脚本))
* [官方](https://github.com/v2fly/fhs-install-v2ray)

```shell
# curl依赖
yum install curl

# 要求：Ubuntu 16+ / Debian 8+ / CentOS 7+ 系统
# 一键安装脚本 有好几个自行选择
bash <(curl -s -L https://git.io/v2ray.sh) # 全家桶 支持20多种功能

# 官方
bash <(curl -L -s https://install.direct/go.sh) # 已无法使用
# 下面替代方式
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
bash install-release.sh
# 数据文件
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh
bash install-dat-release.sh
# 移除
bash install-release.sh --remove

```

**TLS证书安装**

```shell
# TLS证书（参考v2官网）
curl  https://get.acme.sh | sh
source ~/.bashrc

# 生成ECC证书：
sudo ~/.acme.sh/acme.sh --issue -d XXX00.ml --standalone -k ec-256 
# XXX00.ml为你自己的域名

# 将ECC证书和密钥安装到 /etc/v2ray 中
sudo ~/.acme.sh/acme.sh --installcert -d XXX007.ml --fullchainpath /etc/v2ray/v2ray.crt --keypath /etc/v2ray/v2ray.key --ecc

# TLS证书相关路径
ssl_certificate       /etc/v2ray/v2ray.crt;
ssl_certificate_key   /etc/v2ray/v2ray.key;

# 注意 80 端口会被已启动的Nginx占用，需要关闭nginx服务
systemctl stop nginx
# 证书生成以后再恢复
systemctl start nginx
# 缺点：脚本生成的定时证书续期任务依赖80端口，所以需要手动关闭nginx
```

**v2ray服务端配置参考**

* [参考配置](https://github.com/mikewubox/V2Ray3)

```
{
	"log": {
		"access": "/var/log/v2ray/access.log",
		"error": "/var/log/v2ray/error.log",
		"loglevel": "warning"
	},
	"inbounds": [
		{
			"port": 12555,
			"protocol": "vmess",
			"settings": {
				"clients": [
					{
						"id": "e6c0e55d-9107-469b-a761-b6016c7ddaa8",
						"level": 1,
						"alterId": 233
					}
				]
			},
			"streamSettings": {
				"network": "ws",
				"wsSettings": {
	  			"path":"/4dd2adbc/"
        }
			},
			"sniffing": {
				"enabled": true,
				"destOverride": [
					"http",
					"tls"
				]
			}
		}
		//include_ss
		//include_socks
		//include_mtproto
		//include_in_config
		//
	],
	"outbounds": [
		{
			"protocol": "freedom",
			"settings": {
				"domainStrategy": "UseIP"
			},
			"tag": "direct"
		},
		{
			"protocol": "blackhole",
			"settings": {},
			"tag": "blocked"
        },
		{
			"protocol": "mtproto",
			"settings": {},
			"tag": "tg-out"
		}
		//include_out_config
		//
	],
	"dns": {
		"servers": [
			"https+local://cloudflare-dns.com/dns-query",
			"1.1.1.1",
			"1.0.0.1",
			"8.8.8.8",
			"8.8.4.4",
			"localhost"
		]
	},
	"routing": {
		"domainStrategy": "IPOnDemand",
		"rules": [
			{
				"type": "field",
				"ip": [
					"0.0.0.0/8",
					"10.0.0.0/8",
					"100.64.0.0/10",
					"127.0.0.0/8",
					"169.254.0.0/16",
					"172.16.0.0/12",
					"192.0.0.0/24",
					"192.0.2.0/24",
					"192.168.0.0/16",
					"198.18.0.0/15",
					"198.51.100.0/24",
					"203.0.113.0/24",
					"::1/128",
					"fc00::/7",
					"fe80::/10"
				],
				"outboundTag": "blocked"
			},
			{
				"type": "field",
				"inboundTag": ["tg-in"],
				"outboundTag": "tg-out"
			}
			,
			{
				"type": "field",
				"domain": [
					"domain:epochtimes.com",
					"domain:epochtimes.com.tw",
					"domain:epochtimes.fr",
					"domain:epochtimes.de",
					"domain:epochtimes.jp",
					"domain:epochtimes.ru",
					"domain:epochtimes.co.il",
					"domain:epochtimes.co.kr",
					"domain:epochtimes-romania.com",
					"domain:erabaru.net",
					"domain:lagranepoca.com",
					"domain:theepochtimes.com",
					"domain:ntdtv.com",
					"domain:ntd.tv",
					"domain:ntdtv-dc.com",
					"domain:ntdtv.com.tw",
					"domain:minghui.org",
					"domain:renminbao.com",
					"domain:dafahao.com",
					"domain:dongtaiwang.com",
					"domain:falundafa.org",
					"domain:wujieliulan.com",
					"domain:ninecommentaries.com",
					"domain:shenyun.com"
				],
				"outboundTag": "blocked"
			}			,
                {
                    "type": "field",
                    "protocol": [
                        "bittorrent"
                    ],
                    "outboundTag": "blocked"
                }
			//include_ban_ad
			//include_rules
			//
		]
	},
	"transport": {
		"kcpSettings": {
            "uplinkCapacity": 100,
            "downlinkCapacity": 100,
            "congestion": true
        }
	}
}
```

