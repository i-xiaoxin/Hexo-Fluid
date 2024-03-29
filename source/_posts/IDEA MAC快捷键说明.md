---
title: IDEA Mac快捷键说明
date: 2023-1-8 20:29:02
tags: 
- IDEA
- Mac
index_img: 
banner_img: /img/article3.webp
categories:
- IDEA
comment: waline
---

# Mac键盘符号和修饰键说明

- `⌘` Command
- `⇧` Shift
- `⌥` Option
- `⌃` Control
- `↩︎` Return/Enter
- `⌫` Delete
- `⌦` 向前删除键（Fn+Delete）
- `↑` 上箭头
- `↓` 下箭头
- `←` 左箭头
- `→` 右箭头
- `⇞` Page Up（Fn+↑）
- `⇟` Page Down（Fn+↓）
- `Home` Fn + ←
- `End` Fn + →
- `⇥` 右制表符（Tab键）
- `⇤` 左制表符（Shift+Tab）
- `⎋` Escape (Esc)

### **一、Editing（编辑）**

- `⌃Space` 基本的代码补全（补全任何类、方法、变量）
- `⌃⇧Space` 智能代码补全（过滤器方法列表和变量的预期类型）
- `⌘⇧↩` 自动结束代码，行末自动添加分号
- `⌘P` 显示方法的参数信息
- `⌃J, Mid. button click` 快速查看文档
- `⇧F1` 查看外部文档（在某些代码上会触发打开浏览器显示相关文档）
- `⌘+鼠标放在代码上` 显示代码简要信息
- `⌘F1` 在错误或警告处显示具体描述信息
- `⌘N, ⌃↩, ⌃N` 生成代码（getter、setter、构造函数、hashCode/equals,toString）
- `⌃O` 覆盖方法（重写父类方法）
- `⌃I` 实现方法（实现接口中的方法）
- `⌘⌥T` 包围代码（使用if..else, try..catch, for, synchronized等包围选中的代码）
- `⌘/` 注释/取消注释与行注释
- `⌘⌥/` 注释/取消注释与块注释
- `⌥↑` 连续选中代码块
- `⌥↓` 减少当前选中的代码块
- `⌃⇧Q` 显示上下文信息
- `⌥↩` 显示意向动作和快速修复代码
- `⌘⌥L` 格式化代码
- `⌃⌥O` 优化import
- `⌃⌥I` 自动缩进线
- `⇥ / ⇧⇥` 缩进代码 / 反缩进代码
- `⌘X` 剪切当前行或选定的块到剪贴板
- `⌘C` 复制当前行或选定的块到剪贴板
- `⌘V` 从剪贴板粘贴
- `⌘⇧V` 从最近的缓冲区粘贴
- `⌘D` 复制当前行或选定的块
- `⌘⌫` 删除当前行或选定的块的行
- `⌃⇧J` 智能的将代码拼接成一行
- `⌘↩` 智能的拆分拼接的行
- `⇧↩` 开始新的一行
- `⌘⇧U` 大小写切换
- `⌘⇧] / ⌘⇧[` 选择直到代码块结束/开始
- `⌥⌦` 删除到单词的末尾（⌦键为Fn+Delete）
- `⌥⌫` 删除到单词的开头
- `⌘+ / ⌘-` 展开 / 折叠代码块
- `⌘⇧+` 展开所以代码块
- `⌘⇧-` 折叠所有代码块
- `⌘W` 关闭活动的编辑器选项卡

### **二、Search/Replace（查询/替换）**

- `Double ⇧` 查询任何东西
- `⌘F` 文件内查找
- `⌘G` 查找模式下，向下查找
- `⌘⇧G` 查找模式下，向上查找
- `⌘R` 文件内替换
- `⌘⇧F` 全局查找（根据路径）
- `⌘⇧R` 全局替换（根据路径）
- `⌘⇧S` 查询结构（Ultimate Edition 版专用，需要在Keymap中设置）
- `⌘⇧M` 替换结构（Ultimate Edition 版专用，需要在Keymap中设置）

### **三、Usage Search（使用查询）**

- `⌥F7 / ⌘F7` 在文件中查找用法 / 在类中查找用法
- `⌘⇧F7` 在文件中突出显示的用法
- `⌘⌥F7` 显示用法

### **四、Compile and Run（编译和运行）**

- `⌘F9` 编译Project
- `⌘⇧F9` 编译选择的文件、包或模块
- `⌃⌥R` 弹出 Run 的可选择菜单
- `⌃⌥D` 弹出 Debug 的可选择菜单
- `⌃R` 运行
- `⌃D` 调试
- `⌃⇧R, ⌃⇧D` 从编辑器运行上下文环境配置

### **五、Debugging（调试）**

- `F8` 进入下一步，如果当前行断点是一个方法，则不进入当前方法体内
- `F7` 进入下一步，如果当前行断点是一个方法，则进入当前方法体内，如果该方法体还有方法，则不会进入该内嵌的方法中
- `⇧F7` 智能步入，断点所在行上有多个方法调用，会弹出进入哪个方法
- `⇧F8` 跳出
- `⌥F9` 运行到光标处，如果光标前有其他断点会进入到该断点
- `⌥F8` 计算表达式（可以更改变量值使其生效）
- `⌘⌥R` 恢复程序运行，如果该断点下面代码还有断点则停在下一个断点上
- `⌘F8` 切换断点（若光标当前行有断点则取消断点，没有则加上断点）
- `⌘⇧F8` 查看断点信息

### **六、Navigation（导航）**

- `⌘O` 查找类文件
- `⌘⇧O` 查找所有类型文件、打开文件、打开目录，打开目录需要在输入的内容前面或后面加一个反斜杠`/`
- `⌘⌥O` 前往指定的变量 / 方法
- `⌃← / ⌃→` 左右切换打开的编辑tab页
- `F12` 返回到前一个工具窗口
- `⎋` 从工具窗口进入代码文件窗口
- `⇧⎋` 隐藏当前或最后一个活动的窗口，且光标进入代码文件窗口
- `⌘⇧F4` 关闭活动run/messages/find/... tab
- `⌘L` 在当前文件跳转到某一行的指定处
- `⌘E` 显示最近打开的文件记录列表
- `⌘⌥← / ⌘⌥→` 退回 / 前进到上一个操作的地方
- `⌘⇧⌫` 跳转到最后一个编辑的地方
- `⌥F1` 显示当前文件选择目标弹出层，弹出层中有很多目标可以进行选择(如在代码编辑窗口可以选择显示该文件的Finder)
- `⌘B / ⌘ 鼠标点击` 进入光标所在的方法/变量的接口或是定义处
- `⌘⌥B` 跳转到实现处，在某个调用的方法名上使用会跳到具体的实现处，可以跳过接口
- `⌥ Space, ⌘Y` 快速打开光标所在方法、类的定义
- `⌃⇧B` 跳转到类型声明处
- `⌘U` 前往当前光标所在方法的父类的方法 / 接口定义
- `⌃↓ / ⌃↑` 当前光标跳转到当前文件的前一个/后一个方法名位置
- `⌘] / ⌘[` 移动光标到当前所在代码的花括号开始/结束位置
- `⌘F12` 弹出当前文件结构层，可以在弹出的层上直接输入进行筛选（可用于搜索类中的方法）
- `⌃H` 显示当前类的层次结构
- `⌘⇧H` 显示方法层次结构
- `⌃⌥H` 显示调用层次结构
- `F2 / ⇧F2` 跳转到下一个/上一个突出错误或警告的位置
- `F4 / ⌘↓` 编辑/查看代码源
- `⌥ Home` 显示到当前文件的导航条
- `F3`选中文件/文件夹/代码行，添加/取消书签
- `⌥F3` 选中文件/文件夹/代码行，使用助记符添加/取消书签
- `⌃0...⌃9` 定位到对应数值的书签位置
- `⌘F3` 显示所有书签

### **七、Refactoring（重构）**

- `F5` 复制文件到指定目录
- `F6` 移动文件到指定目录
- `⌘⌫` 在文件上为安全删除文件，弹出确认框
- `⇧F6` 重命名文件
- `⌘F6` 更改签名
- `⌘⌥N` 一致性
- `⌘⌥M` 将选中的代码提取为方法
- `⌘⌥V` 提取变量
- `⌘⌥F` 提取字段
- `⌘⌥C` 提取常量
- `⌘⌥P` 提取参数

### **八、VCS/Local History（版本控制/本地历史记录）**

- `⌘K` 提交代码到版本控制器
- `⌘T` 从版本控制器更新代码
- `⌥⇧C` 查看最近的变更记录
- `⌃C` 快速弹出版本控制器操作面板

### **九、Live Templates（动态代码模板）**

- `⌘⌥J` 弹出模板选择窗口，将选定的代码使用动态模板包住
- `⌘J` 插入自定义动态代码模板

### **十、General（通用）**

- `⌘1...⌘9` 打开相应编号的工具窗口
- `⌘S` 保存所有
- `⌘⌥Y` 同步、刷新
- `⌃⌘F` 切换全屏模式
- `⌘⇧F12` 切换最大化编辑器
- `⌥⇧F` 添加到收藏夹
- `⌥⇧I` 检查当前文件与当前的配置文件
- `§⌃, ⌃`` 快速切换当前的scheme（切换主题、代码样式等）
- `⌘,` 打开IDEA系统设置
- `⌘;` 打开项目结构对话框
- `⇧⌘A` 查找动作（可设置相关选项）
- `⌃⇥` 编辑窗口标签和工具窗口之间切换（如果在切换的过程加按上delete，则是关闭对应选中的窗口）

### **十一、Other（一些官方文档上没有体现的快捷键）**

- `⌘⇧8` 竖编辑模式

## **导航**

- ⌘O 查找类文件 Ctrl + N
- ⌘⌥O 前往指定的变量 / 方法 Ctrl + Shift + Alt + N
- ⌃← / ⌃→ 左右切换打开的编辑tab页 Alt← / Alt→
- ⎋ 从工具窗口进入代码文件窗口 ESC
- ⌘L 在当前文件跳转到某一行的指定处 Ctrl + G
- ⌘E 显示最近打开的文件记录列表 Ctrl + E
- ⌘⌥← / ⌘⌥→ 退回 / 前进到上一个操作的地方 Ctrl + Alt + ← Ctrl + Alt + →
- ⌘⇧⌫ 跳转到最后一个编辑的地方
- ⌃H 显示当前类的层次结构 Ctrl + H
- ⌘⇧H 显示方法层次结构
- ⌃⌥H 显示调用层次结构
- F4 / ⌘↓ 编辑/查看代码源
- ⌘⌥U 显示类UML图
- ⌃J 查看注释

## **编辑**

- ⌥⌦ 删除到单词的末尾（⌦键为Fn+Delete）
- ⌥⌫ 删除到单词的开头
- ⌘+ / ⌘- 展开 / 折叠代码块
- ⌘F1 在错误或警告处显示具体描述信息
- ⌘⌥L 格式化代码
- ⌃⌥O 优化import
- ⇧↩ 开始新的一行
- ⌘⇧↩ 自动结束代码，行末自动添加分号
- ⌃I 实现方法（实现接口中的方法）
- ⇧F6 重命名文件或者变量
- ⌘N, ⌃↩, ⌃N 生成代码（getter、setter、构造函数、hashCode/equals,toString）
- ⌘P 显示方法的参数信息

## **查找**

- Double⇧ 查找任何东西
- ⌘⇧F 全局查找（根据路径）
- ⌘F 文件内查找
- ⌘G 查找模式下，向下查找
- ⌘⇧G 查找模式下，向上查找

## **导航**

- ⌘⌥B 跳转到接口的实现
- ⌘U 查看接口定义
- ⌘⌥← / ⌘⌥→ 退回 / 前进到上一个操作的地方
- ⌘B / ⌘ 鼠标点击 进入光标所在的方法/变量的接口或是定义处
- ⌃⇧B 跳转到类型声明处
- ⌥ Space, ⌘Y 快速打开光标所在方法、类的定义
- ⌘O 查找类文件
- ⌘⇧O 查找所有类型文件、打开文件、打开目录，打开目录需要在输入的内容前面或后面加一个反斜杠/
- F12 返回到前一个工具窗口
- ⎋ 从工具窗口进入代码文件窗口
- ⇧⎋ 隐藏当前或最后一个活动的窗口，且光标进入代码文件窗口
- F3选中文件/文件夹/代码行，添加/取消书签
- ⌥F3 选中文件/文件夹/代码行，使用助记符添加/取消书签
- ⌃0…⌃9 定位到对应数值的书签位置
- ⌘F3 显示所有书签
- ⌥F1 显示当前文件选择目标弹出层，弹出层中有很多目标可以进行选择(如在代码编辑窗口可以选择显示该文件的Finder)
- ⌘F12 弹出当前文件结构层，可以在弹出的层上直接输入进行筛选（可用于搜索类中的方法）

## **通用**

- ⌃⌘F 切换全屏模式

# Part 1：Editing（编辑）

快捷键	作用
Control + Space	基本的代码补全（补全任何类、方法、变量）
Control + Shift + Space	智能代码补全（过滤器方法列表和变量的预期类型）
Command + Shift + Enter	自动结束代码，行末自动添加分号
Command + P	显示方法的参数信息
Control + J	快速查看文档
Shift + F1	查看外部文档（在某些代码上会触发打开浏览器显示相关文档）
Command + 鼠标放在代码上	显示代码简要信息
Command + F1	在错误或警告处显示具体描述信息
Command + N, Control + Enter, Control + N	生成代码（getter、setter、hashCode、equals、toString、构造函数等）
Control + O	覆盖方法（重写父类方法）
Control + I	实现方法（实现接口中的方法）
Command + Option + T	包围代码（使用if...else、try...catch、for、synchronized等包围选中的代码）
Command + /	注释 / 取消注释与行注释
Command + Option + /	注释 / 取消注释与块注释
Option + 方向键上	连续选中代码块
Option + 方向键下	减少当前选中的代码块
Control + Shift + Q	显示上下文信息
Option + Enter	显示意向动作和快速修复代码
Command + Option + L	格式化代码
Control + Option + O	优化 import
Control + Option + I	自动缩进线
Tab / Shift + Tab	缩进代码 / 反缩进代码
Command + X	剪切当前行或选定的块到剪贴板
Command + C	复制当前行或选定的块到剪贴板
Command + V	从剪贴板粘贴
Command + Shift + V	从最近的缓冲区粘贴
Command + D	复制当前行或选定的块
Command + Delete	删除当前行或选定的块的行
Control + Shift + J	智能的将代码拼接成一行
Command + Enter	智能的拆分拼接的行
Shift + Enter	开始新的一行
Command + Shift + U	大小写切换
Command + Shift + ] / Command + Shift + [	选择直到代码块结束 / 开始
Option + Fn + Delete	删除到单词的末尾
Option + Delete	删除到单词的开头
Command + 加号 / Command + 减号	展开 / 折叠代码块
Command + Shift + 加号	展开所以代码块
Command + Shift + 减号	折叠所有代码块
Command + W	关闭活动的编辑器选项卡

# Part 2：Search / Replace（查询/替换）

快捷键	作用
Double Shift	查询任何东西
Command + F	文件内查找
Command + G	查找模式下，向下查找
Command + Shift + G	查找模式下，向上查找
Command + R	文件内替换
Command + Shift + F	全局查找（根据路径）
Command + Shift + R	全局替换（根据路径）
Command + Shift + S	查询结构（Ultimate Edition 版专用，需要在 Keymap 中设置）
Command + Shift + M	替换结构（Ultimate Edition 版专用，需要在 Keymap 中设置）

# Part 3：Usage Search（使用查询）

快捷键	作用
Option + F7 / Command + F7	在文件中查找用法 / 在类中查找用法
Command + Shift + F7	在文件中突出显示的用法
Command + Option + F7	显示用法

# Part 4：Compile and Run（编译和运行）

快捷键	作用
Command + F9	编译 Project
Command + Shift + F9	编译选择的文件、包或模块
Control + Option + R	弹出 Run 的可选择菜单
Control + Option + D	弹出 Debug 的可选择菜单
Control + R	运行
Control + D	调试
Control + Shift + R, Control + Shift + D	从编辑器运行上下文环境配置

# Part 5：Debugging（调试）

快捷键	作用
F8	进入下一步，如果当前行断点是一个方法，则不进入当前方法体内
F7	进入下一步，如果当前行断点是一个方法，则进入当前方法体内，如果该方法体还有方法，则不会进入该内嵌的方法中
Shift + F7	智能步入，断点所在行上有多个方法调用，会弹出进入哪个方法
Shift + F8	跳出
Option + F9	运行到光标处，如果光标前有其他断点会进入到该断点
Option + F8	计算表达式（可以更改变量值使其生效）
Command + Option + R	恢复程序运行，如果该断点下面代码还有断点则停在下一个断点上
Command + F8	切换断点（若光标当前行有断点则取消断点，没有则加上断点）
Command + Shift + F8	查看断点信息

# Part 6：Navigation（导航）

快捷键	作用
Command + O	查找类文件
Command + Shift + O	查找所有类型文件、打开文件、打开目录，打开目录需要在输入的内容前面或后面加一个反斜杠/
Command + Option + O	前往指定的变量 / 方法
Control + 方向键左 / Control + 方向键右	左右切换打开的编辑 tab 页
F12	返回到前一个工具窗口
Esc	从工具窗口进入代码文件窗口
Shift + Esc	隐藏当前或最后一个活动的窗口，且光标进入代码文件窗口
Command + Shift + F4	关闭活动 run/messages/find/... tab
Command + L	在当前文件跳转到某一行的指定处
Command + E	显示最近打开的文件记录列表
Option + 方向键左 / Option + 方向键右	光标跳转到当前单词 / 中文句的左 / 右侧开头位置
Command + Option + 方向键左 / Command + Option + 方向键右	退回 / 前进到上一个操作的地方
Command + Shift + Delete	跳转到最后一个编辑的地方
Option + F1	显示当前文件选择目标弹出层，弹出层中有很多目标可以进行选择(如在代码编辑窗口可以选择显示该文件的 Finder)
Command + B / Command + 鼠标点击	进入光标所在的方法/变量的接口或是定义处
Command + Option + B	跳转到实现处，在某个调用的方法名上使用会跳到具体的实现处，可以跳过接口
Option + Space, Command + Y	快速打开光标所在方法、类的定义
Control + Shift + B	跳转到类型声明处
Command + U	前往当前光标所在方法的父类的方法 / 接口定义
Control + 方向键下 / Control + 方向键上	当前光标跳转到当前文件的前一个 / 后一个方法名位置
Command + ] / Command + [	移动光标到当前所在代码的花括号开始 / 结束位置
Command + F12	弹出当前文件结构层，可以在弹出的层上直接输入进行筛选（可用于搜索类中的方法）
Control + H	显示当前类的层次结构
Command + Shift + H	显示方法层次结构
Control + Option + H	显示调用层次结构
F2 / Shift + F2	跳转到下一个 / 上一个突出错误或警告的位置
F4 / Command + 方向键下	编辑 / 查看代码源
Option + Home	显示到当前文件的导航条
F3	选中文件 / 文件夹 / 代码行，添加 / 取消书签
Option + F3	选中文件 / 文件夹/代码行，使用助记符添加 / 取消书签
Control + 0…Control + 9	定位到对应数值的书签位置
Command + F3	显示所有书签

# Part 7：Refactoring（重构）

快捷键	作用
F5	复制文件到指定目录
F6	移动文件到指定目录
Command + Delete	在文件上为安全删除文件，弹出确认框
Shift + F6	重命名文件
Command + F6	更改签名
Command + Option + N	一致性
Command + Option + M	将选中的代码提取为方法
Command + Option + V	提取变量
Command + Option + F	提取字段
Command + Option + C	提取常量
Command + Option + P	提取参数

# Part 8：VCS / Local History（版本控制 / 本地历史记录）

快捷键	作用
Command + K	提交代码到版本控制器
Command + T	从版本控制器更新代码
Option + Shift + C	查看最近的变更记录
Control + C	快速弹出版本控制器操作面板

# Part 9：Live Templates（动态代码模板）

快捷键	作用
Command + Option + J	弹出模板选择窗口，将选定的代码使用动态模板包住
Command + J	插入自定义动态代码模板

# Part 10：General（通用）

快捷键	作用
Command + 1…Command + 9	打开相应编号的工具窗口
Command + S	保存所有
Command + Option + Y	同步、刷新
Control + Command + F	切换全屏模式
Command + Shift + F12	切换最大化编辑器
Option + Shift + F	添加到收藏夹
Option + Shift + I	检查当前文件与当前的配置文件
Control + `	快速切换当前的 scheme（切换主题、代码样式等）
Command + ,	打开 IDEA 系统设置
Command + ;	打开项目结构对话框
Shift + Command + A	查找动作（可设置相关选项）
Control + Shift + Tab	编辑窗口标签和工具窗口之间切换（如果在切换的过程加按上 delete，则是关闭对应选中的窗口）



## 终端光标快捷键

移到行首：control+a

移到行尾：control+e



原文链接🔗

[IntelliJ IDEA For Mac 快捷键](https://dancon.gitbooks.io/intellij-idea/content/keymap-mac-introduce.html)

[IntelliJ IDEA 常用快捷键 之 Mac 版](https://blog.csdn.net/qq_35246620/article/details/78263380)

<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>