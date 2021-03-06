---
layout: post
title: 如何重构一个大型历史项目——经验详情页改版总结
category : program
tagline: "原创"
tags : [program]
keywords: [重构, 改版, 总结]
description: 本文将会介绍重构一个大型历史项目的方方面面
---
{% include JB/setup %}

挺长时间没写博客了，你以为我去玩了吗？其实我是去做了一个大项目。

开场之前先来个热身吧，最近撸了一个小游戏《[看你有多色](http://yanhaijing.com/color/)》，源代码在[这里](https://github.com/yanhaijing/color)，欢迎大家交流。

项目做完后在内部做了次分享，本文就是这次分享的文字版，警告：本文略长，建议备好咖啡，来一段愉快的阅读之旅吧，我保证不会让你失望的。

本文将会介绍如何重构一个大型历史项目的方方面面。

本文将会按照下面的目录展开分享，本文仅讨论技术。

- 简介
- 准备
- 时间
- 团队
- 技术
- 架构
- 边界
- 收益
- 总结

## 简介
这次要改版的是经验最重要的页面之一——[经验wap详情页](http://jingyan.baidu.com/article/fdffd1f850e750f3e88ca169.html)，流量巨大，历史悠久，就导致这必然是一个复杂的问题。

## 准备
项目上的准备就是PM提供的MRD和设计师提供的PSD，前端的准备基本是先熟悉旧版页面的逻辑，我本人其实也不熟悉这个页面。

## 时间
项目开始前肯定要给出排期，那么接下来就说排期和时间管理的问题。

### 预估时间
前端项目的时间预估最准确的就是从UE图预估，拿到UE图后将其拆分成各个组件，然后评估每个模块的开发时间，其实思想就是分治，将不可评估任务拆分成可评估任务。

这次的项目因为是就模块改版，还有一个维度就是看就模块老代码，这次老模块是一个X，工作量未知，所以留了一些buffer，结果还真派上了用场，在老代码里跌倒了几次。/(ㄒoㄒ)/~~

举个例子：

- 导航 2h
- 头部 3h
- 尾部 4h
- 。。。

### 风险
这次遇到可能影响进度的风险点主要都是来自经验的老代码，历史数据结构，各种边界case。

有些东西，你不看老模块的代码是不可能知道的，所以还是要把老代码全部看懂才行。

### 合作
本次两个人一起开发，就涉及到如何合作的问题，在开始之前我制定了一些规则。

才学了语义化的提交信息，我准备尝试一下，我指定的规范在这里《[我的提交信息规范](http://yanhaijing.com/git/2016/02/17/my-commit-message/)》。

最后我们一共提交了100多个commit，基本上等同于100多个小功能，我提倡勤提交，这样方便两个人同步进度。

因为页面被拆分成了模块，而我们的项目中支持模块化开发，所以两个人合作起来还是挺容易的，每个人负责自己的模块，互不干扰。

### 效率
根据人月神话来说，团队人越多可能效率越低，那么如何保证两个人的效率和一个人时是一样的呢？我们有如下利器来保证：

- 模块化开发
- 良好的基础框架
- 合理的项目（功能）拆分

## 技术
接下来说说我们这次改版的技术细节吧。

### 指导原则
技术上大体可以分为以下几类：

- 新技术
- 陈旧技术
- 成熟技术
- 稳定技术

对于新技术我们还是持敬畏的态度，这次的选型就是选择成熟稳定的技术。 

### 技术栈
来说说我们这次的技术栈吧

- fis-plus
- sass
- <del>es6（新技术）</del>
- <del>promise（新技术）</del>
- template
- zepto，gmu

## 架构
架构是个最大的问题，我们从下面四个方面来说说：

- 目录
- 结构
- 样式
- 行为

### 目录
我们的目录规范就是fisp的目录规范，主要目录如下：

- page 模版文件
- test 测试数据
- widget 组成页面的组件

### 模版
我们用的是smarty模版，对应的就是结构——HTML，比如我们的模版文件是index.tpl，其内部有各个组件拼装而成。

模版可以继承，我们的业务模版都继承通用模版，这样的好处就是方便给所有页面添加通用的一些东西。

还可以在父模版挖一些坑，在自模版填，这个功能叫做block。

举个例子吧，比如如下图我们在父模版挖一个title的坑，因为每个页面的title可能都不一样，在子页面我们会向填坑

再来说说widget，一般的一个页面会被拆分成一个个模块（组件），然后又每个模块去拼装页面。

举个例子，下面左边就是拆分完的页面，右边是伪代码。

![]({{BLOG_IMG}}289.png)

### 样式
这次改版我们的样式决定使用sass这个预处理器。

我们将样式精心分类，总共有以下三类：

- 公用css
- 组件css
- mixin

我们定义了两个mixin：

    @mixin clearfix() {
      &::after {
        content: "";
        display: table;
        clear: both;
      }
    }

    @mixin size($width, $height: $width) {
      width: $width;
      height: $height;
    }

_variables.scss里面我们只定义了几个能够抽取成公共的颜色变量

再来说说组件样式，一个典型的组件代码大概如下：

    @import "mixin";
    @import "variables";

    .wgt-step-read {
        background: $bg-color;
        a {
            color: $color-primary;
        }
        .arrow {
            @include size(5px, 9px);
        }
    }

其编译后的css如下所示
    
    .wgt-step-read {background: #fff;}
    .wgt-step-read a {color: #333;}
    .wgt-step-read .arrow {width: 5px; height: 9px}

用了sass后我们可能会忽视嵌套问题，任意嵌套，我们规定嵌套不能超过三层。

**小贴士**：这里介绍一个css选择符的两种逻辑，我推荐的其实是中庸思想，不走极端。

    .wgt-step-read .arrow // 嵌套
    .wgt-step-read-arrow // 组合类名

    .wgt-step-read ul p span // bad
    .wgt-step-read .pv // 添加类名，解决上面嵌套层级的问题

再来说说我们样式的指导原则：

- wgt前缀（区分模块和其他类）
- 嵌套不要超过三层（添加类名解决）
- 合理使用类名（语义）
- 合理使用mixin（区分extend）
- DRY

最后说说布局之争，项目开始时我们有如下两种布局可以选择：

- px
- rem

最终我们选的还是传统的px，因为我们为我们是知识型页面，页面多为文字；其次px比rem更简单。

在我的逻辑看起来我用大屏是未来看到更多的文字，而不是更大的字体，如果我想看更大的字体我自己调整字体就好了。

### 行为
行为这里展开讲的地方不太多，因为按照模块拆分以后，每个js都不太大，强调的一点就是不要再拼接字符串了，而是改用前端模版。

## 边界
边界这块在内部分享中介绍了很多内容，这里绝大部分都不能展开叙述，因为可能会涉及到一些内部的数据；当然我会将一些比较通用的知识，不涉及经验的具体细节。

### 起源
对于经验而言，经过了多年的沉淀，已经没有人能够熟悉的知道他的方方面面，那么我们如何理清楚，经验的一些属性、分类、边界情况呢？

答案就是追本溯源，而不是舍本逐末，我开始的出发方向就错了，我一头跳进了某一个case中，还好后来及时修正了这种错误的思想。

举个例子，我们是如何追本溯源的呢？大概从以下一些方面：

- 产生经验的地方
- 后端数据
- 老代码

最直观的其实就是生产经验的地方，也就是说每个case都能在这里找到出处。

其次就是看模版数据，再次就是看经验的老代码。

总而言之，经验的复杂程度超乎我的想象。

### 一条线的变迁
有太多内容都不能公开给大家，想来想去就说这个吧，还比较有意思。

对你没看错，就是下面这条线，就是她！！！

![]({{BLOG_IMG}}292.jpg)

看看我是怎么实现的吧，如下图，对你们看错，separator-line就是这条线

![]({{BLOG_IMG}}293.jpg)

不知道你会不会鄙视我，我选择了一种不太优雅的实现方式，为了装饰性增加了多余的html。

不知道你会如何实现这条线，来说说我的实现方式吧。

最开始时，我把它放在了tag-wp的bodder-bottom上，后来我发现tag-wp竟然可能不存在，o(╯□╰)o

我想那我就放在abstract-wp上吧，也许聪明的你猜到了解决，是的，这个元素也可能没有。

最后我只能搞一个单独元素出来了，o(╯□╰)o

这个问题其实我们也可以换个思路，就峰回路转了，我们可以不把这条线考虑成装饰性元素，而是考虑成分割线，是不是就好理解了呢，分割线就是该用一个元素来表示（hr），虽然我这里没有用hr，但是内心总是不那么忐忑了，谁让我是强迫症呢，不写出优雅的代码我会吃不下，睡不着的。

## 收益
最后来说说这次改版的收益吧。

### 可维护性
旧的代码，陈年失修，而且一直在打补丁，自我感觉改版后，可维护性上了一个量级，这里就不能给大家贴代码展示了。

不过有一个数据还是可以拿来展示一些的，其实新旧版都要处理同样的逻辑的，同样的逻辑旧版用了107行，而新版只用了73行。

## 总结
这次改版让我明白了一个道理，页面中的任何元素都可能没有，需要考虑到容错处理；设计不能一蹴而就，往往需要反复打磨，反复修改，或者推翻重来；凭空的设计完美却不实用，好的设计必须结合产品内容，项目细节。

### 优点
这次改版我认为在下面几点上都是做的不错的地方：

- 架构
- 新技术
- 团队合作

### 缺点
如果说优缺点的话，我认为唯一的缺点就是时间的预估了，时间预估上的失误还是很明显的。如果时间不能准确预估需要留好buffer。

### 表扬
我需要表扬我队友的效率，开发真的很快。比我快太多。

### 批评与自我批评
我要批评我队友的效率，复制粘贴了太多老代码，自己也没看，而且代码规范都没遵守。

我也要批评我自己的效率，想得太多，做的太少，应该提高自己的效率。

## 附
本文的[百度脑图](http://naotu.baidu.com/file/5a45700dae2df3912c8587a8491b27fe?token=86b24083b15e144d)。



