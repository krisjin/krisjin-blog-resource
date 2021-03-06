title: 互联网金融产品如何利用大数据做风控？
date: 2015-10-13 19:42:31
categories: 架构
tags:
---


由于互联网金融涉及货币发行（比特币）、第三方支付、投资理财（网络银行、保险、基金、证券、财富管理）、信贷（P2P、众筹、网络微贷）、征信等等，各个领域的风控策略并不尽相同，不能一概而论，下面讨论只能涵盖了常见的风控策略。 

个人认为“大数据”除了强调数据的海量外，更重要的在于用于风控的历史数据的广度和深度，其中：

数据的广度：指用于风控的数据源多样化，任何互联网金融企业并不能指望依据单一的海量数据就解决风控问题，正如在传统金融风控中强调的“交叉验证”的原则一样，应当通过多样化的数据来交叉验证风险模型。以下的风控策略也如此，可能对同一风险事件采用了多种策略。 

数据的深度：指用于风控的数据应当基于某个垂直领域真实业务场景及过程完整记录，从而保证数据能够还原真实的业务过程逻辑。

一个关于数据深度典型的反例：第三方支付貌似有丰富的真实交易记录，但由于大部分场景下无法获取交易商品的详细信息及用户身份，在用于风控时候价值大打折扣。

回到题主的话题：互联网金融产品如何利用大数据做风控。大致有如下一些常见方法：

1、基于某类特定目标人群、特定行业、商圈等做风控 

由于针对特定人员、行业、商圈等垂直目标做深耕，较为容易建立对应的风险点及风控策略。 
例如： 
针对大学生的消费贷，主要针对大学生人群的特征 
针对农业机具行业的融资担保。 
针对批发市场商圈的信贷。

2、基于自有平台身份数据、历史交易数据、支付数据、信用数据、行为数据、黑名单/白名单等数据做风控 

身份数据：实名认证信息（姓名、身份证号、手机号、银行卡、单位、职位）、行业、家庭住址、单位地址、关系圈等等。
交易数据/支付数据：例如B2C/B2B/C2C电商平台的交易数据，P2P平台的借款、投资的交易数据等。 
信用数据：例如P2P平台借款、还款等行为累积形成的信用数据，电商平台根据交易行为形成的信用数据及信用分（京东白条、支付宝花呗），SNS平台的信用数据。 
行为数据：例如电商的购买行为、互动行为、实名认证行为（例如类似新浪微博单位认证及好友认证）、修改资料（例如修改家庭及单位住址，通过更换频率来确认职业稳定性）。 
黑名单/白名单：信用卡黑名单、账户白名单等。 

3、基于第三方平台服务及数据做风控 

互联网征信平台（非人行征信）、行业联盟共享数据（例如小贷联盟、P2P联盟） FICO服务 
Retail Decisions(ReD)、Maxmind服务 
IP地址库、代理服务器、盗卡/伪卡数据库、恶意网址库等 
舆情监控及趋势、口碑服务。诸如宏观政策、行业趋势及个体案例的分析等等

4、基于传统行业数据做风控 

人行征信、工商、税务、房管、法院、公安、金融机构、车管所、电信、公共事业（水电煤）等传统行业数据。 

5、线下实地尽职调查数据 

包括自建风控团队做线下尽职调查模式以及与小贷公司、典当、第三方信用管理公司等传统线下企业合作做风控的模式。 
虽然貌似与大数据无关，但线下风控数据也是大数据风控的重要数据来源和手段。


——————————

现在提大数据风控的文章，大都是讲个幌子，一点实质内容都没有。

今年早些时候因为工作的原因，对这方面有点兴趣，某次跟一位老师谈到大数据风控的时候，听到一些大数据风控的实际例子，我在这里给大家分享一下。

我们来看一下传统的信贷风控模式，贷前，贷中，贷后三部分中最看重的是贷前，而对贷中贷后并不是非常注重。

而这样的思想在互联网金融上是绝对要不得的。互联网金融的客户什么牛鬼蛇神都有，其降低风险的主要手段其实并不是完善而大量的数据收集、统计和分析。

而是风险的分摊。

这也是金融行业最简单的贷款风险控制手段。如果我做十笔就可能会亏一笔，那我每九笔的利润至少要能摊平这一次的亏损。

大数据的使用对于确定盈亏出清利率提供了相对合理的手段。

似乎这样就已经可以了？只要事先选择合理的利率和合适客户就解决问题了？

可惜事实总比你想的更奇葩。数据是靠谱的，分析却可以不靠谱，人却可以更不靠谱，这点我不多说，大家都明白。

互联网的大数据在这一点上是比不上传统的信贷风控手段的。

那位和我聊天的老师在这一点上说的很有意思。

他说，互联网金融做的客户，多半是银行不想做不愿做的，它们只是捡了别人不要的东西，哪天银行真想要了，买来就是。

诚然这里面多少有些一厢情愿，不过前半句倒是事实。从一开始，互联网金融就选择了传统信贷所难以下手的市场。谨记这点便引出了为什么我要在前面说，互联网金融绝对不能放松贷中和贷后的风控。

而恰巧，大数据能帮互联网金融做到的最棒的部分，还就是贷中和贷后。

关于贷中管理，这位老师讲了一个很有意思的案例。

他提到有某家金融机构，使用大数据监控某个区域内企业的流水，如果某段时间流水出现了异常，那么他们就会派人去调查具体发生了什么事。

这种方法在现行的传统风控手段中也是很常规的，但大数据给我们带来的便利除了降低人力成本，更主要的是可以发掘更多的判断依据。

尤其在借款人有意隐瞒目前经营状况的时候，一些经营外的数据就有可能产生意义。

举个简单的例子，如果借款人有打算跑路了，那除了现金流的变化，也会有些其他的变化，比如购买旅行箱，订机票，国外相关网站的浏览。

而在贷后方面，大数据的介入除了给我们提供分析手段，更方便了我们对于客户需求的发掘。或许在以后，银行可以为个体提供更为贴心和前瞻性的服务，而这些也是大数据应用的重点。

关于大数据的前景：

大数据目前不一定是单独一家企业可以掌握的，以后也不一定。

目前出现的对大数据的应用情况是，用户数据的采集和共享。
在未来的一段时间内，因为法律法规不健全，必然会出现严重侵犯个人隐私的商业行为，而大数据会不会因此受阻，我个人是抱悲观态度的。

大数据会有用，但首先要降低金融服务的成本。无论数据如何，最终还是需要人。而人的成本只会越来越高。
以上是我对大数据应用的一点思考。可能不完全切题，欢迎讨论。

来源：[http://www.zhihu.com/question/27715270](http://www.zhihu.com/question/27715270)