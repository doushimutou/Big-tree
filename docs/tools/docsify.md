# dosify的使用与搭建过程



### 下载与安装

​	全局下载并安装dosify

```bash
npm i docsify-cli -g
```

​	如果要在`./docs`子目录中编写文档，则可以使用`init`命令。

```bash
docsify init ./docs
```

在之后`init`完成后，你可以看到在文件列表`./docs`子目录。

- `index.html` 作为入口文件
- `README.md` 作为主页
- `.nojekyll` 防止GitHub Pages忽略以下划线开头的文件

您可以轻松地在中更新文档`./docs/README.md`

本地启动预览：

```bash
docsify serve docs
```



### 配置侧边栏

​	首先创建_sidebar.md文件，在index.html中添加如下配置

<script>   
    window.$docsify = {     
        loadSidebar: true   
    } 
</script>
<script src="//unpkg.com/docsify/lib/docsify.min.js"></script>
此外还需要创建一个nojekyll`in `./docs`来防止GitHub Pages忽略以下划线开头的文件

**具体_sidebar.md配置示例：**

```markdown
* Introduction
  * [简介](README.md)
* JAVA基础
  * [set详解](java-base/set.md)
* JAVA设计模式
  * [单例模式](java-design/singleton.md) 
* JAVA高并发
  * [concurrentHashmap详解](java-concurrent/Concurrent-hashmap.md)
* VUE
* NODEJS
* 工具
  * [docsify的基础使用](tools/docsify.md)

```

### 配置导航栏

首先创建_navbar.md文件，在index.html中添加如下配置：

```markdown
//设置导航
      loadNavbar: true,
```

-navbar.md 文件实例

```
* Introduction
    * [简介](README.md)

* JAVA
    * [基础]
    * [This]
* ...
 * [基础]
 * [This]
```



### 配置搜索栏

在index.html中添加search 插件

```html
//设置搜索插件
      search: {
        paths: 'auto',
        placeholder: '🔍 搜索 ',
        noData: '😞 找不到结果! ',
        // 搜索标题的最大程级
        depth: 6
      },
<script src="https://cdn.jsdelivr.net/npm/docsify@4/lib/plugins/search.js"></script>
```



### 配置首页

在index.html中配置coverpage

```
<script>
  window.$docsify = {
    coverpage: true
  }
</script>
```

添加_coverpage.md文件，示例文件如下：

```markdown
<img width="180px" style="border-radius: 50%" bor src="https://nodejsred.oss-cn-shanghai.aliyuncs.com/nodejs_roadmap-logo.jpeg?x-oss-process=style/may">

# Node.js技术栈指南

- 本文档是作者从事 ```Node.js Developer``` 以来的学习历程，旨在为大家提供一个较详细的学习教程，侧重点更倾向于 Node.js 服务端所涉及的技术栈，如果本文能为您得到帮助，请给予支持！

[![stars](https://badgen.net/github/stars/Q-Angelo/Nodejs-Roadmap?icon=github&color=4ab8a1)](https://github.com/Q-Angelo/Nodejs-Roadmap) [![forks](https://badgen.net/github/forks/Q-Angelo/Nodejs-Roadmap?icon=github&color=4ab8a1)](https://github.com/Q-Angelo/Nodejs-Roadmap)

[GitHub](<https://github.com/Q-Angelo/Nodejs-Roadmap>)
[开始阅读](README.md)

```



### 配置分页

在index.html中添加pagination插件

```html
pagination: {
        previousText: '上一篇',
        nextText: '下一篇',
        crossChapter: true
      },
<script src="//unpkg.com/docsify-pagination/dist/docsify-pagination.min.js"></script>
```

### 配置页面导航

在index.html中添加docsify-page-toc配置：

```html
'page-toc': {
        tocMaxLevel: 3,
        target: 'h1, h2, h3'
      },
 <script src="//unpkg.com/docsify-page-toc/dist/docsify-page-toc.js"></script>
```



### 官网地址

更多详细配置请看dosify官网：https://docsify.js.org/

