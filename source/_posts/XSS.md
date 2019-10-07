---
title: XSS进阶(一)
date: 2019-10-07 15:13:09
tags:
  - WEB
  - CTF
categories: 安全
---
# XSS进阶（一）

直接接着上文http://xss.fbisb.com 中的题，以下的题请使用FireFox进行测试，chrome会默认禁止执行url中的script代码，导致无法弹窗

## L2

先尝试搜索
```
<script>alert(/xss/)</script>
```

然后审查元素：

![u2x2kt.png](https://s2.ax1x.com/2019/10/07/u2x2kt.png)

有两处出现了回显，但是观察第一行，我们的代码直接原封不动的回显出来，估计八成就是HTML实体编码了，这个没办法，没法绕过。那么下面那条语句会不会是一个突破口呢？

尝试构造：`"><script>alert(/xss/)</script>`

成功闭合前面的标签，完成突破，弹窗完毕

## L3

还是先尝试搜索：
```
<script>alert(/xss/)</script>
```

 ，然后审查元素

和上图是一样的，于是尝试`'><script>alert(/xss/)</script>` ，只不过这次是用单引号闭合的

![u2xcTI.png](https://s2.ax1x.com/2019/10/07/u2xcTI.png)

发现<,>,' 都被转义了，那么这个时候就想，既然闭合不了，那么能不能通过给标签新加一个属性，来达到运行JavaScript的目的呢？

尝试:`'onfocus='alert(/xss/)'` 当点击文本框时实现弹窗（觉得实现效果不好的话可以尝试添加其他的事件属性）

## L4

还是先尝试搜索：`"><script>alert(/xss/)</script>`  ，然后审查元素，

![u2x60A.png](https://s2.ax1x.com/2019/10/07/u2x60A.png)

发现尖括号被过滤了，那尝试上面的思路如何呢？构造`'onfocus='alert(/xss/)'` 发现应该把单引号换成双引号去闭合，即：`"onfocus='alert(/xss/)'`完成弹窗

## L5

尝试搜索：`"><script>alert(/xss/)</script>`  ，然后审查元素:

![u2xymd.png](https://s2.ax1x.com/2019/10/07/u2xymd.png)

发现在尖括号中的script会被替换成scr_ipt，导致\<script\>标签无法被正常的识别，同时on也会被替换成o_n,这时候有两个方向，一种是尝试大小写script绕过检测，第二种是利用标签属性，避开使用script，同时也不能使用on事件。

这里经过尝试，大小写无法绕过，那只能是第二种思路了：考虑现在我们还可以利用什么呢？也就是href和src的javascript协议了；

构造`"><a href="javascript:alert(/xss/)">`,然后点击我们自己构造的a标签，实现弹窗，这里可能会有人不解，href不是应该加一个URL嘛，为什么可以执行JavaScript，其实这里确实是一个URL，浏览器将href等于的字符串当作URL处理，然后读到了JavaScript:这个js的伪协议，于是调用js引擎解析后面的代码，实现了js 的执行

## L6

搜索：`"><script>alert(/xss/)</script>`  ，然后审查元素,仍然是上图那样，script会被替换，那试试刚刚得payload：`"><a href="javascript:alert(/xss/)">` ：

![u2xrOH.png](https://s2.ax1x.com/2019/10/07/u2xrOH.png)

发现确实可以闭合，但是href被替换为hr_ef了，那用src呢？打扰了。。src也会被替换，那么on呢？同样被替换，那么我们回归比较上一题得第一种思路可以解决问题嘛？尝试一下：

`"><scRIpT>alert(/xss/)</scRIpt>` 

OK,OVER！

## L7

先尝试一下上一关得payload`"><scRIpT>alert(/xss/)</scRIpt>` 

![u2xRtP.png](https://s2.ax1x.com/2019/10/07/u2xRtP.png)

发现script被替换了，欸，能不能双写绕过？构造`'"><scriscriptpt>alert(/xss/)</scriscriptpt>` 

原理是什么呢？当一个script被替换为空时，外层得script又被组合出来了，形成payload，完成弹窗，OK

## L8

题目形式发生了变化，变成了添加友链，观察一下代码：

![u2xWff.png](https://s2.ax1x.com/2019/10/07/u2xWff.png)

那我们能不能直接：`
javascript:alert(/xss/)` , 结果不行，发现script被替换，但是仔细看，这里是在标签内，我们其实可以尝试使用HTML Encode绕过，尝试一下：
```
&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3a;&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x2f;&#x78;&#x73;&#x73;&#x2f;&#x29;
```

这个其实就是`javascript:alert(/xss/)`得HTML实体编码，由于是在HTML标签中，这些标签会被正常得解析成HTML，但是在后端看来并没有script需要被替换，于是将内容原封不动得返回到前端交由浏览器渲染。

ok，payload有效，完成弹窗

## L9

先尝试一下`javascript:alert(/xss/)`

![u2xhp8.png](https://s2.ax1x.com/2019/10/07/u2xhp8.png)

提示链接不合法，那我们考虑一下，什么样的链接是合法的呢？通过不断尝试，发现后端会判断有没有"http://"这个字符串出现，如果出现，其他都不检测，就认为是合法链接，将用户输入拼接进a标签中

那这时候，我们灵光一显，我构造`javascript:alert(http://)` (记得除了http://外，要HTML encode一下绕过替换)，OK，实现弹窗

## L10

直接啥都没了？没有输入的地方？这我怎么办嘛？别着急，审查一下元素：

![u2x41S.png](https://s2.ax1x.com/2019/10/07/u2x41S.png)

发现猫腻，输入框被隐藏了，那我们只需要把hidden属性去掉，文本框就会显现出来，但是注意这里是没有submit按钮，也就是无法提交我们的表单，那这时候怎么办？----当然是自己加一个标签啊！

![u2x56g.png](https://s2.ax1x.com/2019/10/07/u2x56g.png)

测试一下那个字段会回显：发现t_sort会将我们提交的数据拼接起来，ok，开始构造XSS：

`">onfocus="alert(/xss/)"`

聚焦时实现弹窗

------

OK，今天就先到这里吧，明天接着写后10道题



