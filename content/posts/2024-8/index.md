---
title: "GitHub原账号被标记为spam的前后"
date: 2024-11-15T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [GitHub]
featuredImagePreview: ""
summary: "2024-10-28, 我的 GitHub 原账号 [senzyo](https://github.com/senzyo) 被标记为 `spam`, 觉得申诉效果渺茫, 所以改用新的 GitHub 账号 [senzyo-desu](https://github.com/senzyo-desu) 和 GitLab 账号 [senzyo_sama](https://gitlab.com/senzyo_sama)。<br />虽然在提交工单的 43 天后, [senzyo](https://github.com/senzyo) 恢复正常, 但同时 [senzyo-desu](https://github.com/senzyo-desu) 被暂停使用, 我还得再改回来, 难绷。"
---

## 账号被标记

{{< admonition type=warning title="账号迁移" open=true >}}
<del>GitHub 原账号 [senzyo](https://github.com/senzyo) 被标记为 `spam`, 所以改用 GitHub 新账号 [senzyo-desu](https://github.com/senzyo-desu) 和 GitLab 账号 [senzyo_sama](https://gitlab.com/senzyo_sama)。</del>

<del>`senzyo` 相关的 GitHub 链接, 请替换为 `senzyo-desu` 相关。</del>
{{< /admonition >}}

由于想尝试 sing-box [v1.10.0](https://sing-box.sagernet.org/zh/changelog/#1100) 支持 AdGuard DNS 过滤的新特性, 于是把 [AdguardTeam/AdguardFilters](https://github.com/AdguardTeam/AdguardFilters) 仓库中的规则通过 GitHub Actions 转换为 srs 格式, 没想到账号立刻就被 GitHub 官方标记为 spam 了。

百思不得其解啊, 这也算滥用嘛。

查了查账号被标记这种情况的工单处理速度, 看到了这篇帖子 [Github 账号被标记, 提交工单超过一个月都没有回应](https://v2ex.com/t/1078590), 发现就算新注册一个付费账号来发起工单, 也不能保证一定可以得到回复。

好吧, 我估计 [senzyo](https://github.com/senzyo) 这个账号大概是没戏了。

不过账号没有被彻底封禁, 还可以被登录, 但是不能被删除, 也不能被更改用户名 (防止转生是吧, 很好, 把我防住了)

## 好消息与坏消息

### 好消息

在提交申诉工单的 37 天后, 终于得到回复。在简短的两次交流后 (虽然耗时一周), GitHub 终于表示已清除对于账号 [senzyo](https://github.com/senzyo) 的所有限制, **[senzyo](https://github.com/senzyo) 堂堂复活**, 可喜可贺？

### 坏消息

当初好不容易从 [senzyo](https://github.com/senzyo) 迁移到 [senzyo-desu](https://github.com/senzyo-desu), 结果现在原账号恢复正常, 新账号却被暂停了 (Suspend)。

所以我还得把一些项目数据中, `senzyo-desu` 相关的 GitHub 链接再改回 `senzyo`, 麻烦。

我倒是知道, 根据 GitHub 的 ToS, 一个人只能拥有一个 GitHub 免费账号, 但是一般管的也不严啊, 好家伙这下把我整老实了, 乖乖用 GitLab 得了。