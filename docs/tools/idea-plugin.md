

# 类图与时序图

## 类图：

> 首先梳理清楚类与类之间有哪几种关系：

### 依赖：

依赖关系是用一套带箭头的虚线表示的；如下图表示A依赖于B；他描述一个对象在运行期间会用到另一个对象的关系；

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/BD6B19BB5BBB497CBBD49FCE140A41F9/8421)

显然，依赖也有方向，双向依赖是一种非常糟糕的结构，我们总是应该保持单向依赖，杜绝双向依赖的产生；

注：在最终代码中，依赖关系体现为类构造方法及类方法的传入参数，箭头的指向为调用关系；依赖关系除了临时知道对方外，还是“使用”对方的方法和属性；

### 泛化（继承）：

继承关系为 is-a的关系；两个对象之间如果可以用 is-a 来表示，就是继承关系：（..是..)

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/7DD6DD364973447BB1DCBFD4F0AE7AD5/8432)

### 实现：

实现关系用一条带空心箭头的虚线表示；

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/1C13D71972DD43D99680BA13EDB796D1/8430)

java中最终表现为implements

### 聚合:

表示一种弱的‘拥有’关系，即has-a的关系，体现的是A对象可以包含B对象，但B对象不是A对象的一部分。 **两个对象具有各自的生命周期**。

**表示方法：** 
聚合关系用**空心的菱形+实线箭头**表示。

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/37CE6F0AF34D480FB3CA00BDEBA6C4E9/8428)

### 组合:

组合是一种强的‘拥有’关系，是一种contains-a的关系，体现了严格的部分和整体关系，**部分和整体的生命周期一样**。
**表示方法：**  
组合关系用**实心的菱形+实线箭头**表示，还可以使用连线两端的数字表示某一端有几个实例。



![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/D0043BBBAD354890A7794C0DE6BFEB51/8426)

## 时序图：

时序图(Sequence Diagram)，又名序列图、循序图，是一种UML交互图。它通过描述对象之间发送消息的时间顺序显示多个对象之间的动态协作。

### **时序图的几种元素**

#### **角色：**

系统角色，可以是人或者其他系统，子系统。以一个小人图标表示。

#### **对象：**

对象位于时序图的顶部,以一个矩形表示。对象的命名方式一般有三种：

  1 对象名和类名。例如：华为手机:手机、loginServiceObject:LoginService。

  2 只显示类名，不显示对象，即为一个匿名类。例如：:手机、:LoginSservice。

  3 只显示对象名，不显示类名。例如：华为手机:、loginServiceObject:。

#### **生命线：**

时序图中每个对象和底部中心都有一条垂直的虚线，这就是对象的生命线(对象的时间线)。以一条垂直的虚线表。

#### **控制焦点：**

控制焦点代表时序图中在对象时间线上某段时期执行的操作。以一个很窄的矩形表示。

#### **消息：**

同步消息：消息的发送者把控制传递给消息的接收者，然后停止活动，等待消息的接收者放弃或者返回控制。用来表示同步的意义。以一条实线+实心箭头表示。

异步消息：消息发送者通过消息把信号传递给消息的接收者，然后继续自己的活动，不等待接受者返回消息或者控制。异步消息的接收者和发送者是并发工作的。以一条实线+大于号表示。

返回消息：返回消息表示从过程调用返回。以小于号+虚线表示。

### **自关联消息：**

表示方法的自身调用或者一个对象内的一个方法调用另外一个方法。以一个半闭合的长方形+下方实心剪头表示。



**实例：**

![](https://note.youdao.com/yws/public/resource/a5e638392f6684aedc1068423c7aa4aa/xmlnote/D78A7E426F094ABCAF5B4B0A33923B6B/6988)



## idea的插件介绍使用

学会了类图和时序图的基础，工作中codeReview时，要快速弄清关系结构，自己画类图和时序图耗时好了，使用idea帮助我们快速生成类图和时序图。

### Diagram类图

首先下载**UML-SUPPORT**插件

在类的维度，右键点击showDiagram，生成的类图示例如图：

![](https://note.youdao.com/yws/public/resource/47834f4144ec032013f0f0f68b3ce163/xmlnote/1D6C44C392BD40C19A0DC35DEC0FD68A/8437)



### SequenceDiagram 时序图

首先下载**SequenceDiagram**插件

在方法的维度，右键点击SequenceDiagram,生成的时序图示例如下：

![](https://note.youdao.com/yws/public/resource/a5e638392f6684aedc1068423c7aa4aa/xmlnote/64ED4BE52D26481297007E04DE721AFF/8440)



