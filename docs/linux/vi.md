## linux系列- vi 编辑器

### vi

#### vi三种模式：

> insert模式

只有在Insert mode下，才可以做文字输入，按「ESC」键可回到命令行模式。

> command mode 命令模式

控制屏幕光标的移动，字符、字或行的删除，移动复制某区段及进入Insert mode下，或者到 last line mode。

> last line mode 底模式(可并入命令模式)

将文件保存或退出vi，也可以设置编辑环境，如寻找字符串、列出行号……等。



#### vi的基本操作:

1. ##### 进入insert模式

   在command mode下输入 i\O\a 直接进入insert mode ; 进入insert 模式后,文件底部显示 `--insert--`,此时可以在文件中输入文字

2. ##### 从insert模式切换到command模式

   `esc` 键退出insert模式

3. ##### 退出vi及保存文件

   在「命令行模式（command mode）」下，按一下「：」冒号键进入「Last line mode」，例如：
   : w filename （输入 「w filename」将文章以指定的文件名filename保存）
   : wq (输入「wq」，存盘并退出vi)
   : q! (输入q!， 不存盘强制退出vi)
   
4. ##### 在command code模式下搜索

   从文件的尾部向头部搜索:     `? 搜索内容`

   从文件的头部向尾部搜索:      `/搜索内容`

   光标跳至下一个搜索内容:	 n

   光标跳至下一个搜索内容: 	N

   向上翻页: `ctrl+b` 

   向下翻页:`ctrl+f`

##### 附上大神总结的图:

<img src="https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/18EE2F2769574776AD33D06FF786F09C/8321"  />


### 常用命令：

