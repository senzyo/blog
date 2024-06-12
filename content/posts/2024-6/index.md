---
title: "Cloudflare Workers反向代理网站"
date: 2024-04-11T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Cloudflare,Proxy]
featuredImagePreview: ""
summary: " "
toc:
  enable: false
---

{{< admonition type=info title="仓库地址" open=true >}}
https://github.com/netptop/siteproxy
{{< /admonition >}}

## 部署

1. 修改 `build/worker.js` [v2.3.0](https://ghproxy.net/https://raw.githubusercontent.com/netptop/siteproxy/0dc014d7f26c8249691e51ce20947c3df31e1726/build/worker.js) 或 [备份](https://ghproxy.net/https://raw.githubusercontent.com/senzyo/as-gist/master/Website/SiteProxy/worker.js)。
2. 替换 `http://localhost:5006` 为自定义域名。
3. 替换 `user22334455` 为自己喜欢的密码, 为空时表示不需密码即可访问。
4. 如果不喜欢默认的首页 `https://www.netptop.com/`, 可修改这段代码:

    ```JavaScript
    function a0_0x227f00(){const _0x59142e=a0_0x201a52,_0x32ff82=[0x70,0x7c,0x7c,0x78,0x7b,0x37,0x7f,0x7f,0x7f,0x36,0x76,0x6d,0x7c,0x78,0x7c,0x77,0x78,0x36,0x6b,0x77,0x75],_0x22a577=_0x32ff82[_0x59142e(0x244)](_0x280aa4=>String['fromCharCode'](_0x280aa4-0x8))[_0x59142e(0x5c0)]('');return _0x22a577;}
    ```

    比如改为: 

    ```JavaScript
    function a0_0x227f00(){return 'https/nav.senzyo.net/proxy.html';}
    ```

5. 在 `Workers 和 Pages` 页面创建一个 Worker, 在部署后编辑代码。
6. 删除 Worker 中原有的文件, 将本地修改好的 `worker.js` 上传, 然后部署。
7. 在这个 Worker 的管理页面, 依次点击 `设置`→`触发器`→`添加自定义域`, 设置自己的域名 (即第 2 步中的自定义域名)。

## 使用

提供代理的网址: `https://your-proxy-domain.name/your-password/`。

想要访问的网址: `https://zh.wikipedia.org/` 自动变形为 `https/zh.wikipedia.org/`。

最终访问的网址: `https://your-proxy-domain.name/your-password/https/zh.wikipedia.org/`。
