---
title: YAML 日志格式
tags:
  - 随笔
date: 2024-09-24 15:55:15
---
# YAML
YAML (YAML Ain't Markup Language)是一种人类可读的数据序列化格式,常用来表示配置文件。

在Go中,使用'gopgk.in/yaml.v3'包来解析YAML文件。


## YAML格式
可见 [菜鸟教程-YAML入门教程](https://www.runoob.com/w3cnote/yaml-intro.html)

基本语法为
1. 大小写敏感
2. 使用缩进表示层级关系, 缩进不允许使用 tab ,只允许空格,缩进的空格数目不重要,只要同层左对齐。
3. 注释用`#`表示


YAML 支持对象、数组 、纯量三种数据类型， 分别以键值对、`-` 开头的行、和具体值表示。


可见下面基本格式样例
```yml
languages:
  - Ruby
  - Perl
  - Python 
websites:
  YAML: yaml.org 
  Ruby: ruby-lang.org 
  Python: python.org 
  Perl: use.perl.org
```

转换成json为
```json
{ 
  languages: [ 'Ruby', 'Perl', 'Python'],
  websites: {
    YAML: 'yaml.org',
    Ruby: 'ruby-lang.org',
    Python: 'python.org',
    Perl: 'use.perl.org' 
  } 
}
```