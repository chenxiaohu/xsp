# XSP 使用NodeJS轻松套页面

我从事多年互联网应用后端开发工作，主要采用JSP进行服务端页面开发，也就是常说的用JSP“套页面”。JSP作为一种成熟的页面开发技术对于国内常用的超复杂页面使用起来并不轻松。另外随着移动的发展，多屏幕开发对套页面的工作量要求更大。

那么怎么样才能轻松套页面呢？首先我们看看为什么套页面不轻松。下面我介绍两种典型的情况，也是我们要解决的问题:

1. PC浏览器的复杂页面：由于各种不同的内容太多，造成页面本身非常复杂，而且套页面的人一般是传统的后端开发，只是按照前端的要求硬做，后端逻辑和前端逻辑混合到一个大杂烩页面，相当的痛苦。更夸张的是页面会经常修改，雪上加霜！痛苦指数立刻上升。

2. 手机浏览器的简单页面：其实就算很简单的页面，也会经常调用几个不同系统的功能，对于移动网络来说，请求的次数一定要少（多了不但会慢，而且由于移动网络稳定性问题会放大慢，慢到死），通常会需要服务端做一些请求合并，但是对于JSP来讲，做这种事情代价太大。简单点说就是做不快，响应速度慢，慢有两个后果，第一卡死服务器，第二导致用户体验差，对于移动方面来说这是致命的。（另外对于手机APP的支持情况和手机浏览器的情况基本相同，就是要合并多系统的请求，而且不能慢）

对与第一种问题，方案就是模块化。套用当下最流行的“微服务”和“前后端分离”，分解到REST API加上页面的模块化，每件事情都可以比较轻松。

但是。。。每件事情轻松了，结果又回到了第二种问题，慢！对于网络应用来说，一个慢字就干掉了全部！对于JSP类应用来说（其他PHP，ASP应该一样），慢比死还惨。。

大家可能都听过，在软件开发领域解决所有复杂问题的方法都是分解，分解到不负责的很多块就行。duang~ 分解是分解的简单了，可是装起来慢到死，难道是传说中的换种死法。。。

读到这里，大家可以停下来想一想，我说的问题是不是大家也遇到了，如果大家没有同样的问题，那么恭喜您！不需要看后面的内容了。

如果大家也遇到同样的问题，而且想看看我是如何解决的，请继续。

## 如何轻松

电影《一代宗师》里面主角说过一句话：“功夫两个字，一横一竖......”。我想说的是：“软件开发两个字，一拆一合...”。

将一个复杂的软件，拆成很多个简单的模块，完成第一步。
将很多个简单的模块，合成一个完整的软件，搞定！

如果能轻松的将复杂的软件拆成很多简单的模块(简单是指很容易做，而且很难做错)，再轻松的将这些块合成一个完整的软件，而且这个完整的软件的功能和性能都不错。这就是轻松。。我怎么觉得有点绕~~

好吧，实际软件开发要麻烦点，拆和合之间，还有一个做字。整个流程首先拆成块，再把每个块做好，最后再把这些块合起来。

怎么拆，那是一个艺术，不是XSP的重点。XSP主要关注怎么轻松的做每个块，然后怎么轻松的把各个块合起来，还要又快又省资源。

如果还想知道细节，请继续。

## 轻松做模块

如果后端开发只是做REST API而不用关系怎么套页面，那一定很轻松。好，那我们已经搞定一部分轻松了。

_ _ _

*XSP 的 X 部分是怎么回事？*

API有了，前端用就是了，一切都很好。直到有一天，某个前端需要一个不同自己编码的API。
好吧，给API增加一个参数，指定编码方式。。
又过了一阵子，前端又找过来，想给API一个不同的缓冲时间Header。
好吧，再给API增加个参数，指定缓冲时间Header。。
那么多API，每个都有可能要这两种功能，需不需要每个都加？左右为难

对于这种和业务无关的通用需求，每个API都去搞有点不好，那就拆出来搞个专门的“块”吧。这就是XSP的第一块，proxy中的X。
X就是一个http代理模块，可以指定编码转换和缓冲时间，很容易使用。

对于一个url，
	
	http://xxx.yyy.com/abc/def.jsp?x=y&t=1423123
	
假设他原来是GBK编码的，我想转成UTF-8，那就用如下方式：

	http://xsp.yyy.com/x-g2u/xxx.yyy.com/abc/def.jsp?x=y&t=1423123
	
如果要加上设置缓冲时间为900秒，如下：

	http://xsp.yyy.com/x-g2u-900/xxx.yyy.com/abc/def.jsp?x=y&t=1423123
	
很轻松，是吧，如果你有相同的需要，很容易就搞定了。这种功能对NodeJS来说，小意思！

_ _ _

*XSP 的 P 部分是怎么回事？*

有了REST API，又可以随便转码并设置缓冲时间，前端在浏览器用的挺好。直到有一天，负责SEO的同事找到前端，说是为了搜索爬虫可以读取，需要这块内容直接有服务端生成数据，不能由浏览器调用REST API再用JS渲染了。好像前几天负责UE的同事也说过直接服务端渲染更顺留点。

以前遇到这样的事情，前端就会把这个模块做成HTML的形式，交由后端用JSP直接套进最终页面里，然后等后端排期，后端开发时再一起调试一次。。
然后每次改版时重复以上几步，有点烦、有点烦有点烦~~~

对于反复出现的不爽情况，再拆一块出来，这就是XSP的另一块，page中的P。

NodeJS的一个重要功能就是在后端完成浏览器在前端完成的渲染功能，给原来的url再加个xspt参数就可以了，参数指定一个模板文件，采用[handlebars](http://handlebarsjs.com/)做模板，当然也可以轻松的换成任何自己喜欢用的NodeJS模板模块。

	http://xsp.yyy.com/p-g2u-900/xxx.yyy.com/abc/def.jsp?x=y&t=1423123&xspt=def.html
	
根据这个新url，让后端同事直接SSI进整页面就行了。
有了page部分，前端就可以自己根据需要用REST API完成界面了，省掉了无休止的的等后端排期，等后端开发，等后端联调的循环。。

> 不用等待别人为轻松之本，当然有时你想趁等别人的功夫好好休息放松就另说了:)

外围的X和P已经介绍了，这两块只能帮助轻松的做模块，应该能帮到你一部分。
最重要的通常最后出场，能读到这里的希望你休息一下，精神好了再看后面的大招，怎么合起来！


## 轻松集成页面

首先，通过多个REST API来集成，最大的问题就是速度问题，假设有10个API调用，每个耗时1秒，如果串行执行，最少要花超过10秒。对于JSP这种请求绑定线程的技术架构来说，要并行执行，就要采用多线程，但是多线程资源代价又太大，如果采用异步IO加少量线程事件驱动的模式，代码非常复杂，而且再增加其他逻辑的话，复杂度就高到没法用了。
但是对于NodeJS来说，要实现并行非常简单，因为调用API时只是等待别的网络服务返回结果而已，用NodeJS的异步事件驱动方式这种情况根本就不用资源，在加上基于ES5的generator功能的co模块，非常优雅的实现了异步事件驱动方式下的流程管理，集成页面的逻辑实现起来相当简单清楚。下面举几个例子说明一下。

> 例一
> 并发调用10个API，等全部完成后渲染页面。

> 例二
> 共有10个API，并发先调用3个，等这3个全部完成后，在根据返回结果生成相关参数并发调用剩下的7个API，全部完成后渲染页面

> 例三
> 并发调用5个API，等全部完成后渲染页面，其中3个API如果出错，则页面报错，其他2个API如果出错系统只是降级

> 例四
> 并发调用5个API，其中有2个可以使用缓存的结果，而且缓存的结果可以是API直接返回的内容，也可以是根据API返回的结果复杂计算后生成的进一步结果

总之，并发加异步加同步加结果或计算结果缓存这些功能可以很容易的组合使用，而且不消耗资源，这种优势是JSP做不到的（或者说非常难做到）。

通过多个API的各种逻辑组合执行完成后，有时用json输出结果就行了，或者用模板套页面。

本模块的url格式如下

	http://xsp.yyy.com/s-200/abc/def.xsp?a=b&c=d&t=1234577888&...

本模块对应XSP的 S 部分，是script，至此XSP(proXy Script Page)三个模块以及全部介绍完毕。

根据我们的实际经验，最复杂的还是最终的用模板套页面部分比较琐碎，但是谁让我们命苦呢。。
毕竟XSP只是解决了拆合的技术问题，如果原来的东西怎么拆都不好搞，那可能就不是开发的技术问题，而是产品的智商问题了（这里请允许我发发牢骚）。。

后面，我想通过例子来说明用XSP是怎么解决API整合及套页面问题的(一拆一合)，另外后续也想说明一下XSP本身是怎么通过一拆一合的方式来设计实现的，如果大家有兴趣我再写吧。还是先看XSP怎么用吧。