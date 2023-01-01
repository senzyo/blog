---
title: "公共DNS"
date: 2022-12-03T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Network]
featuredImagePreview: ""
summary: 列举几个公共 DNS 的地址, 包括 DoH 与 DoT。
---

## 国内DNS

| 官网                                                    | IPV4(53)                 | IPV6(53)                            | DoH(443)                         | DoH IP(443)                                                      | DoT(853)             | DoT IP(853)                              |
| ------------------------------------------------------- | ------------------------ | ----------------------------------- | -------------------------------- | ---------------------------------------------------------------- | -------------------- | ---------------------------------------- |
| [阿里 DNS](https://www.alidns.com/)                     | 223.5.5.5<br />223.6.6.6 | 2400:3200::1<br />2400:3200:baba::1 | https://dns.alidns.com/dns-query | https://223.5.5.5/dns-query<br />https://223.6.6.6/dns-query     | tls://dns.alidns.com | tls://223.5.5.5<br />tls://223.6.6.6     |
| [腾讯 DNSPod](https://www.dnspod.cn/Products/publicdns) | 119.29.29.29             | 2402:4e00::                         | https://doh.pub/dns-query        | https://1.12.12.12/dns-query<br />https://120.53.53.53/dns-query | tls://dot.pub        | tls://1.12.12.12<br />tls://120.53.53.53 |

## 国外DNS

| 官网                                                                                 | IPV4(53)                         | IPV6(53)                                       | DoH(443)                                           | DoH IP(443)                                                      | DoT(853)                                    | DoT IP(853)                              |
| ------------------------------------------------------------------------------------ | -------------------------------- | ---------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------- | ---------------------------------------- |
| [Google DNS](https://developers.google.com/speed/public-dns/docs/using)              | 8.8.8.8<br />8.8.4.4             | 2001:4860:4860::8888<br />2001:4860:4860::8844 | https://dns.google/dns-query                       | https://8.8.8.8/dns-query<br />https://8.8.4.4/dns-query         | tls://dns.google                            | tls://8.8.8.8<br />tls://8.8.4.4         |
| [Cloudflare DNS](https://developers.cloudflare.com/1.1.1.1/setup/)                   | 1.1.1.1<br />1.0.0.1             | 2606:4700:4700::1111<br />2606:4700:4700::1001 | https://cloudflare-dns.com/dns-query               | https://1.1.1.1/dns-query<br />https://1.0.0.1/dns-query         | tls://cloudflare-dns.com                    | tls://1.1.1.1<br />tls://1.0.0.1         |
| [Quad9](https://www.quad9.net/service/service-addresses-and-features/) Block Malware | 9.9.9.9<br />149.112.112.112     | 2620:fe::fe<br />2620:fe::9                    | https://dns.quad9.net/dns-query                    | https://9.9.9.9/dns-query<br />https://149.112.112.112/dns-query | tls://dns.quad9.net                         | tls://9.9.9.9<br />tls://149.112.112.112 |
| [Dnswarden](https://dnswarden.com/index.html) Uncensored                             |                                  |                                                | https://dns.dnswarden.com/uncensored               |                                                                  | tls://uncensored.dns.dnswarden.com          |                                          |
| [CleanBrowsing](https://cleanbrowsing.org/filters/) Security Filter                  | 185.228.168.9<br />185.228.169.9 | 2a0d:2a00:1::2<br />2a0d:2a00:2::2             | https://doh.cleanbrowsing.org/doh/security-filter/ |                                                                  | tls://security-filter-dns.cleanbrowsing.org |                                          |

{{< admonition type=example title="更多" open=true >}}
更多 DNS 提供商参见 https://adguard-dns.io/kb/zh-CN/general/dns-providers/#cfiec-public-dns
{{< /admonition >}}