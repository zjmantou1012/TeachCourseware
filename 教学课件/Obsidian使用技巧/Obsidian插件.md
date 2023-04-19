### 推荐插件
- File Explorer Note Count：显示笔记的数量
![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/setting_note_count.png)
- Recent File：显示最近访问的文件
- Pandoc：转换成word格式（通过命令面板搜索doc）
- Minimal Theme Setting：主题

### 插件清单
- Outliner
- Mindmap
- Calendar
- Tasks
- Obsidian memos
- Excalidraw
- cMenu
- Quick Explorer
- DataView

### 插件类型
- 个性化功能
- 界面增强
- 展现增强
- 数据格式增强

### 特别推荐

#### 特别推荐：Proxy github

无需梯子访问插件库

#### ~~Outerline~~

新版本核心插件支持

#### Mindmap

以思维导图形式展示大纲

打开命令面板搜 Mindmap 

#### Calender

日记插件，帮助我们方便的回顾写的日记

创建周记
- 显示周数
- 设计周记模板

#### Tasks

管理任务，例如提醒事项（不太推荐）

#### Obsidian memos
![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/memos.png)

![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/Memos2.png)
类似Flomo功能插件

#### Excalidraw

在Obsidian中绘制流程图

![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/Excalidraw.png)

#### Quick Explorer
快速导航栏

## 搜索VS查询（DataView）

搜索：针对内容查找

查询：针对属性的展示

![[搜索技巧]]
##### DataView

![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/dataview.png)

- 定义：Obsidian资料库的查询工具/插件
	- 查询对象：Obsidian数据库
	- 查询依据：YAML数据库/Meatainfo
- 场景
	- 什么时候使用**搜素**
		- 条件单一
		- 无需保存结果
	- 什么时候使用**查询**
		- 条件复杂
		- 需要保存结果
	- YAML
		- 位于Markdown文件开头
		- 首尾三个 ‘-’
		- ![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/YAML.png)
		- Obsidian支持的YAML字段（可以用模板来创建Yaml语句）![[](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/Yaml_template.png)
			- tags
			- publish
			- cssclass
			- aliases
		- 自定义字段
			- category
			- date
			- time
			- title
			- rating
		- 行内标记
			- One Field:: Value
			- 这份文档，可以打[rating:: 5]分
	- Obsidian文件属性
		- file.name：文件标题（字符串）
		- file.folder：文件所属文件夹路径
		- file.path：文件路径
		- file.size：（in bytes）文件大小
		- file.ctime：文件创建的时间（包含日期和时间）
		- file.mtime：文件的修改时间
		- file.cday：文件创建的日期
		- file.mday：文件修改的日期
		- file.tags：笔记中所有标签组
		- file.etags：除去子标签的数组
		- file.inlinks：指向此文件的所有入链数组
		- file.outlinks：指向此文件的所有出链数组
		- file.aliases：文件别名数组
		- file.day：如果文件名中有日期，那么就会以这个字段显示。比如文件名中包含yyyy-mm-dd(年-月-日，例如2021-03-21)，那么就存在这个metadata。
	- 任务属性
		- Task会继承所在文件的所有字段，比如Task所在的页面中已经包含了rating信息来，那么task也会有。
		- completed：任务是否完成
		- fullyCompeted：任务以及所有的子任务是否完成
		- text：任务名
		- line：task所在行
		- path：task所在路径
		- section：连接到任务所在的区块
		- link：连接到距离任务最近的可连接区块
		- subtasks：子任务
		- real：true，是一个真正的任务；false，一个任务之前或者之后的元素列表
		- completion：任务完成的日期
		- due：任务到期时间
		- created：创建时间
		- annotated：true，任务有自定义标记；否则为false
	- 展示方式：
		- List
		- Table
		- Task
	-  完整语法![](https://zjmantou-drawingbed.oss-cn-hangzhou.aliyuncs.com/picture/YAML_yufa.png)
		- dataview
		- list|table|task
		- from
		- where
		- sort
			- asc，升序
			- desc，降序
- 使用建议
	- 保存常用查询
	- 生成文件夹索引
	- ......
- 问题
	- DataView生成的链接是否会改变关系图？
