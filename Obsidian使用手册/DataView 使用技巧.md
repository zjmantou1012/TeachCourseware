
[原文连接](https://forum-zh.obsidian.md/t/topic/195) 

三种生成目录的方式：
1. 从名字
2. 从标签
3. 从作者

## 从名字 

`dataview list from "" where contains(file.name,"习惯")` 


![基础语法](https://forum-zh.obsidian.md/uploads/default/original/1X/d61dd1fe9074e883ed0b7d6f87adc75e86270597.jpeg) 


![](https://forum-zh.obsidian.md/uploads/default/original/1X/44029b25b4fbdffc22aaf866f40c73dae8cf0250.jpeg)


![file中的属性](https://forum-zh.obsidian.md/uploads/default/original/1X/732f60ee4b6ef0fc87acba54337666ba2f6deabc.jpeg)

## 从标签 

`dataview list from #时间管理` 

### 从作者 

借助yaml语法： 
1. 只要写在6个横杠符号之间，yaml就可以被识别。够简单吧？
2. 必须写在文件最上方

```yaml
---
author: zjmantou
title: adb常用命令笔记
time: 2024-08-05 周一
tags:
  - 笔记
  - Android
  - adb
---
```

搜索： 

```dataview
list 
from ""
where contains(title,"adb常用命令笔记")
```
## table 

![](https://forum-zh.obsidian.md/uploads/default/original/1X/5b23983552a126798a11960902264f4ea74eeea3.jpeg)


# dataview官网

[dataview](https://blacksmithgu.github.io/obsidian-dataview/)

