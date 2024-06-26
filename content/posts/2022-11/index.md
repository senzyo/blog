---
title: "Linux date命令的参数格式"
date: 2022-04-06T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Help,Linux]
featuredImagePreview: ""
summary: Linux date 命令的参数格式, 比如 `date + %F`。
toc:
  enable: false
---

以下是`date`相应参数在**2022年4月10日14点40分50秒**的输出示例: 

  | 参数  | 解释                                                    | 输出示例                        |
  | ----- | ------------------------------------------------------- | ------------------------------- |
  | %%    | 输出`%`                                                 | %                               |
  | %a    | 区域设置的工作日缩写                                    | Sun                             |
  | %A    | 区域设置的工作日全称                                    | Sunday                          |
  | %b    | 区域设置的月份缩写                                      | Apr                             |
  | %B    | 区域设置的月份全称                                      | April                           |
  | %c    | 区域设置的工作日+日月年+时间                            | Sun 10 Apr 2022 02:40:50 PM CST |
  | %C    | 比完整年份`%Y`少了最后两个数字                          | 20                              |
  | %d    | 日                                                      | 10                              |
  | %D    | 月日年, 格式相当于`%m/%d/%y`                            | 04/10/22                        |
  | %e    | 日, 相当于`%d`或`%_d`                                   | 10                              |
  | %F    | 年月日, 格式相当于`%Y-%m-%d`                            | 2022-04-10                      |
  | %g    | ISO 8601标准, 年份的最后两个数字                        | 22                              |
  | %G    | ISO 8601标准, 年份全称                                  | 2022                            |
  | %h    | 月份缩写, 相当于`%b`                                    | Apr                             |
  | %H    | 小时, 24小时制 (00~23)                                  | 14                              |
  | %I    | 小时, 12小时制 (01~12)                                  | 02                              |
  | %j    | 一年的第几天 (001~366)                                  | 100                             |
  | %k    | 小时, 24小时制 (00~23)                                  | 14                              |
  | %l    | 小时, 12小时制, 注意数字前面有个空格 ( 1~12)            | 2                               |
  | %m    | 月份 (01~12)                                            | 04                              |
  | %M    | 分钟 (00~59)                                            | 40                              |
  | %n    | 相当于换行符                                            |                                 |
  | %N    | 纳秒 (000000000~999999999)                              | 888682953                       |
  | %p    | 区域设置的上午还是下午                                  | PM                              |
  | %P    | 相当于`%p`, 但是小写                                    | pm                              |
  | %r    | 区域设置的时分秒, 12小时制                              | 02:40:50 PM                     |
  | %R    | 时分, 相当于`%H:%M`                                     | 14:40                           |
  | %s    | Unix时间戳, 自`1970-01-01 00:00:00 UTC`起的第几秒       | 1649572850                      |
  | %S    | 秒 (00~60)                                              | 50                              |
  | %t    | 相当于制表符                                            |                                 |
  | %T    | 时分秒, 24小时制, 相当于`%H:%M:%S`                      | 14:40:50                        |
  | %u    | 一周的第几天 (1~7), 周一是第一天                       | 7                               |
  | %U    | 一年的第几周 (00~53), 周日是每周的开始日               | 15                              |
  | %V    | ISO 8601标准, 一年的第几周 (01~53), 周一是每周的开始日 | 14                              |
  | %w    | 一周的第几天 (0~6), 周日是第0天                        | 0                               |
  | %W    | 一年的第几周 (00~53), 周一是每周的开始日               | 14                              |
  | %x    | 区域设置的日期表示                                      | 04/10/2022                      |
  | %X    | 区域设置的时间表示                                      | 02:40:50 PM                     |
  | %y    | 年份的最后两个数字 (00~99)                              | 22                              |
  | %Y    | 完整年份                                                | 2022                            |
  | %z    | 时区                                                    | +0800                           |
  | %:z   | 时区, 时`:`分                                           | +08:00                          |
  | %::z  | 时区, 时`:`分`:`秒                                      | +08:00:00                       |
  | %:::z | 使用`:`表示时区的必要精度                               | +08                             |
  | %Z    | 时区的字母缩写                                          | CST                             |

