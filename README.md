# QAsystem

第七届中软杯（智能问答系统）

非常感谢杨大帅比和张小可爱为项目所付出的所有心血，最后一次虽有遗憾却不后悔的中软杯

QAQ团队，和我们的logo

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/QAlogo.png" style="zoom:50%;" />

bilibili视频地址：[视频网址](https://www.bilibili.com/video/av35481883)

视频里可以讲解的比较仔细，可以看看。实际上整个项目完成度还很欠缺，较多地方因时间原因未完成，或者只是仓促完成，导致有些部分逻辑很简单，所以效果也不是很好，各位也就当看个思路就好。

---

## 需求介绍

题目要求：[中软杯赛题网址](http://www.cnsoftbei.com/plus/view.php?aid=321)

简略概述要求：
1. 构建一个完整的QA系统
2. 整个系统由三部分构成：前端，后台，知识库
3. 前端：请设计一个程序，实现QA对话界面，该界面可以基于用户提问，自动连接后台、并从知识库寻找答案，并呈现给用户
4. 后台：请设计一个程序从文档中提取尽可能多且质量高的问答对（QA对）
5. 知识库：QA对存储管理的类似于数据库的东西


## 项目架构

### 功能架构划分

分为用户端和管理员端

1. 用户端：用户端为用户使用的页面。用户端提供了用户提问回答，热点问题，智能推荐，闲聊对话等功能。同时页面简洁美观，响应良好，为用户提供了良好的使用体验
2. 管理员端：管理员在后台管理系统的页面。管理员端提供了文档上传，运行网页解析算法和生成QA算法，可视化图表查看数据库内容和热点问题，用户提问情况等图表

### 技术架构划分

分为前端，后台，算法，知识库存储

1. 前端：使用Bootstrap前端框架加上各种前端模块，搭建了具有风格清新，简单朴实的页面，为用户提供了良好的观看体验
2. 后台：使用Django框架，Django作为一款性能优异，轻量级的Python的web框架，能很好的用于本系统的功能支持。作为本系统的后台，为整个系统对外提供流畅服务做到了保障。
3. 算法：算法部分分为网页解析算法和QA对生成算法
4. 知识库：知识库目前使用Elasticsearch搜索引擎的存储模块

### 使用的技术模块

1. 前端：大体上是使用的现成的网页进行了修改（毕竟UI设计不太行，感谢前端大佬），主要使用了Bootstarp，还有一些很多的前端插件，就不一一列举了。
2. 后台：后台主框架是基于Python的Django框架；里面的一些功能模块：比如用户聊天，使用的是图灵的聊天机器人API，可以将这个替换成seq2seq的对话生成（不过需要自己搭模型和训练）；用户意图判断和情感判断使用LUIS（判断是聊天还是提问）；问答系统中的自动补全，准确提问，相似问题是使用Elasticsearch进行查询后返回的；后台数据图表展示使用的是Elasticsearch配套的Kibana；后台的上传模块；后台的APScheduler定时管理模块（用于更新Elasticsearch）。
3. 算法：算法部分因为是设计的基于逻辑的QA对生成，所以没有用到Tensorflow，scikit-learn等深度学习、机器学习的工具，用到了Stanford的语言模块、哈工大的语言工具LTP，主要是使用的LTP了，里面的基于语义的分析，基于成分的分析效果还是挺好的，这个算法主要也是使用了LTP进行分析，然后就是使用很简单逻辑进行拼接生成QA，所以很制杖，效果一般。针对本赛题的数据集华为云帮助手册的效果还可以，但是使用其他数据集的话会大打折扣，主要是因为一是LTP对于简单的短句，逻辑较为简单句子分析的很清晰，但是长难句就不行了；另外也是设计的逻辑代码也很简单，考虑不全面，生成句子的逻辑不够好。
4. 知识库：知识库就是使用的Elasticsearch，使用K-V来保存QA，搜索速度较快。

图例（图中有部分没有介绍）

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E6%8A%80%E6%9C%AF%E6%9E%B6%E6%9E%84.png" style="zoom:50%;" />

## 功能介绍

整个项目主要分成两个模块：

- 用户段的问答系统：问答对话，兴趣推荐，热点问题，闲聊对话
- 后台管理员端：文档录入，QA对生成，知识库管理，数据可视化

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E7%B3%BB%E7%BB%9F%E5%8A%9F%E8%83%BD.png" style="zoom: 67%;" />

详细功能介绍见视频

## 算法部分（QA对）

### QA对生成

这个QA对生成是本项目的核心算法，本项目采用的是简单的基于模板的传统方案（没有使用深度学习技术），目前的做法是通过一段陈述句，转换成疑问句。提问，总得有一段被提问的语句（通常都是陈述句），对这段、这句陈述句提问，就可以生成问句。

有两种方式的问句生成：
- 段落的问句生成，本项目对标题进行构造问句
- 对段落中的每一个单句进行划分，对单句进行问句生成（没有集成到项目中，因为问句生成的太多，需要筛选）

### 问句分类

疑问句的分类（根据论文中的设定）

问句种类 | 问句案例
---|---
人物类 | 谁可以操作云硬盘备份？
地点类 | 在哪里可以登录华为云服务器？
时间类 | 华为云服务器需要多久时间准备？
原因类 | 为什么使用密钥文件无法正常登录Linux弹性云服务器？
数量类 | 弹性云服务器有几种计费方式？
方式类 | 忘记密码如何处理？
定义类 | 什么是命名空间？
描述类 | 区块链服务通道为节点提供了什么？
列表类 | 备份裸金属服务器的操作步骤是哪些？
是否类 | 是否可以进一步提升图像去雾和低光照处理效果？

参考自论文：

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E9%97%AE%E5%8F%A5%E5%88%86%E7%B1%BB.png" style="zoom:50%;" />

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E9%97%AE%E5%8F%A5%E5%88%86%E7%B1%BB2.png" style="zoom:50%;" />

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E9%97%AE%E5%8F%A5%E5%88%86%E7%B1%BB3.png" style="zoom: 50%;" />

### 问句泛化

问句泛化也称为问句复述，主要也是采用哈工大的关于问句复述的论文中（《中文复述问句生成技术研究》2017，曹雨）提到的方式：

1. 句式精简

2. 构建问句模板（复述模板）

3. 使用精简的问句模板去生成其他问句（匹配生成）
4. 然后进行候选问句评价， 选出好的句子
5. 将问句保存起来

但是本项目最后因为效果的原因（构建模板需要花费大量的时间与精力），没有将泛化模块加入整体的架构中，而是采用了最简单的疑问词替换的形式来对问句进行泛化。


## 系统流程
1. 管理员上传文档，网页（两种文件格式，目前本系统主要是解析以**华为云帮助手册的网页**，其他网页需要更改网页解析程序），上传这些文件到服务器端
2. 选择需要生成QA对的文件，调用QA对生成算法，生成QA对存入知识库中
3. 管理员可以前往知识库管理页面，查看所有的QA对，并可以进行增删改查的操作
4. 管理员也可以查看用户的各项信息
5. 用户可以在用户界面进行提问，以获取答案

## 运行流程
整个项目是一个Django的web项目，主要的各种逻辑部分与后台的view.py相连，通过后台调用的逻辑模块。运行的话，直接部署Django，然后使用Django运行整个项目，在浏览器里面输入localhost:xxxx/index.html（好像是这个，记不清了），然后就行了。

> 需要配置的模块：
1. Django
2. Elasticsearch（需要安装，并配置，初始化数据表）
3. Kibana（需要安装并配置）
4. APScheduler
5. LUIS（需要把配置写在view.py里面）
6. 图灵机器人（需要把配置写在view.py里面）
7. MySQL（需要初始化数据表）

## 数据库文件

关于数据库文件的说明

数据库分为两个部分：MySQL与Elasticsearch

- MYSQL的部分用于保存User，QuestionCount，UserMining这三个数据文件，这三个文件可以通过QASystem\models.py 文件生成，这个是Django自动生成的。
  - User表（提问记录表）：
    - userquestion 用户提问的问题
    - question_time 用户提问的时间
  - QuestionCount表（热点问题统计表）：
    - userquestion 统计的问题
    - questioncount 问题被提问的次数
  - UserMining表（用户统计表，用于后台的数据展示）：
    - userip 用户ip
    - userquestion 用户问题
    - usersub 用户问题主题
    - userattention 用户倾向（闲聊还是提问）
    - userlike 用户是否喜欢这个回答以及喜欢程度
    - times 用户提问时间
- Elasticsearch的存储模块用于保存问答对，在安装与配置完Elasticsearch之后，可以使用QASystem\elas.py来创建存储表，表中的字段有：
  - question 问句
  - accuratequestion 准确的答案
  - questionfh1 生成的相似问句1（问句泛化）
  - questionfh2 生成的相似问句2
  - answer 答案，这个与上面的accuratequestion 相同
  - link 问句生成的链接网页
  - subject 主题

## 存在的一些问题

因为当时比赛的时间紧迫，目前这个代码中有些部分还存在一定的问题，例如问句泛化模块并没有集成到整体的流程中（因为问句泛化效果不佳且费时，目前使用的是一种简单的疑问词替换的方式），按句子生成的问句的方式也没有集成进去，因为这样生成的问句太多，需要进行筛选。

## 好看的宣传册

感谢张小可爱为项目制作了非常精美的宣传册，介绍了本项目

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E5%B0%81%E9%9D%A2%2B%E8%83%8C%E9%9D%A2.jpg" style="zoom: 50%;" />

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E9%A1%B9%E7%9B%AE%E7%AE%80%E4%BB%8B%2B%E7%AE%97%E6%B3%952.jpg" style="zoom:50%;" />

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E5%8A%9F%E8%83%BD%E6%A6%82%E8%A6%81%2B%E7%89%B9%E8%89%B2%E5%8A%9F%E8%83%BD.jpg" style="zoom:50%;" />

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E7%89%B9%E8%89%B2%E5%8A%9F%E8%83%BD%2B%E7%AE%97%E6%B3%95%E6%A1%86%E6%9E%B6.jpg" style="zoom:50%;" />

<img src="https://notes-pic.oss-cn-shanghai.aliyuncs.com/%E6%99%BA%E8%83%BD%E9%97%AE%E7%AD%94%E7%B3%BB%E7%BB%9F/%E7%AE%97%E6%B3%95%E5%8A%9F%E8%83%BD.jpg" style="zoom:50%;" />

---

## 问答系统介绍

这边顺便也介绍一下问答系统（因为了解时间不长，可能有误，请谅解），这些也是经过查阅网上资料总结而得的。

问答系统大概分为这几种：
- 基于结构化数据的问答系统
- 基于自由文本的问答系统
- 基于问题答案对的问答系统

其中的结构化数据的问答，应该类似于数据库的匹配查询

基于自由文本，我觉得这个应该就是机器阅读理解，而且这个也是目前比较火的一个方向，这也是我目前比较了解的一个。机器阅读理解有经典的SQUAD数据集，中文的Dureader数据集，基于这些数据集设计的深度学习框架目前效果也不错，可以使用这些模型来搭建问答系统。因为这种基于自由文本的问答，不需要构建问句，直接将问句去文章中匹配即可。难点在于如何将问句匹配文章中的答案，这也是深度学习模型需要处理的问题，而且不需要过多的操作文档，这样减少了文档的信息丢失。机器阅读理解主要有三大难点：表示问题、搜索匹配问题，答案输出问题（这边就不展开了）。

基于QA对，主要是这个知识库，知识库可以是QA对，也可以是目前研究很多的知识图谱。基于QA对的，难点在于Q与A的生成，自然语言生成现在还是一个较难的点，研究的效果也不是很好。基于知识图谱的，难点在于如何构建知识图谱，如何抽取句子中的关系。
