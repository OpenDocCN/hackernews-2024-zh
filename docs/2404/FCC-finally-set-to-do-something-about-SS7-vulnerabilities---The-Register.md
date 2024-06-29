<!--yml

category: 未分类

date: 2024-05-27 12:53:29

-->

# 美国联邦通信委员会（FCC）终于准备采取措施应对SS7漏洞 • The Register

> 来源：[https://www.theregister.com/2024/04/02/fcc_ss7_security/](https://www.theregister.com/2024/04/02/fcc_ss7_security/)

美国联邦通信委员会（FCC）似乎终于加大了保护美国电话网络几十年来存在的漏洞的力度，据称外国政府和监视机构利用这些漏洞远程监听和监控无线设备。

焦点是信令系统第七号（SS7）和Diameter协议，它们被固定和移动网络运营商用来实现网络间的互连，是维系当今电信系统的关键。

根据美国的监督机构和一些立法者，这两种协议都存在安全漏洞，使人们容易受到不必要的窥探。SS7的问题多年来广为人知，至少可以追溯到[2008](https://fahrplan.events.ccc.de/congress/2008/Fahrplan/events/2997.en.html)年，我们在[2010](https://www.theregister.com/2010/04/22/gsm_info_disclosure_hack/)和[2014](https://www.theregister.com/2014/12/26/ss7_attacks/)年曾对此进行过报道。对这些可被利用的缺陷几乎没有任何解决措施。

SS7协议于上世纪70年代中期开发，可以潜在地被[滥用](https://www.theregister.com/2022/02/02/whistleblower_nso_group/)以追踪人们手机的位置；[重定向](https://www.theregister.com/2017/05/03/hackers_fire_up_ss7_flaw/)呼叫和短信以便截取信息；以及[监视](https://www.theregister.com/2018/06/01/wyden_ss7_stingray_fcc_homeland_security/)用户。

Diameter协议于上世纪90年代末开发，包括本地和漫游通话和消息的网络访问和IP移动支持。然而，在传输过程中并不加密原始IP地址，这使得恶意人员更容易进行网络欺骗攻击。

根据FCC[[PDF](https://s3.documentcloud.org/documents/24527269/da-24-308a1.pdf)]的说法，随着覆盖范围的扩展，以及更多网络和参与者的引入，恶意行为者利用SS7和Diameter的机会已经增加。

3月27日，委员会要求电信运营商参与并详细说明他们在防止SS7和Diameter漏洞被滥用以跟踪消费者位置方面所做的工作。

FCC还要求运营商详细说明自2018年以来有关这些协议的任何漏洞利用情况。监管机构希望知道事件发生的日期、具体情况、被利用的漏洞和技术、位置跟踪发生的地点，以及——如已知——攻击者的身份。

这个时间框架很重要，因为在2018年，联邦通信安全、可靠性和互操作性委员会（CSRIC）向FCC发布了几项安全最佳实践，以防止网络入侵和未经授权的位置跟踪。

有兴趣的各方可以在4月26日之前 [提交评论](https://www.fcc.gov/ecfs/search/search-filings)，然后FCC将有一个月的时间作出回应。

### "运营商松懈的安全威胁"

FCC要求公众提交评论，回应美国参议员Ron Wyden的请求，上个月他要求白宫 "应对无线运营商松懈的网络安全实践的严重威胁[[PDF](https://s3.documentcloud.org/documents/24527132/wyden-phone-hacking-letter-to-president-biden.pdf)]。"

根据Wyden的说法，这些威胁是由SS7和直径协议的缺陷引起的，并已被 "威权政府利用进行监视"，获取人们的信息。

"美国需要加强我们对雇佣间谍公司的防御，这些公司帮助外国独裁者威胁美国国家安全、人权和致力于揭露不法行为的记者," Wyden在一份声明中说道。"我期待与FCC合作，通过强制最低网络安全标准来保护美国的电话网络。"

这不是Wyden参议员首次要求政府解决SS7漏洞的问题，也不是他第一次将这些协议缺陷称为国家安全问题。

2023年4月，参议员 [指责AT&T](https://www.theregister.com/2023/04/12/firstnet_cybersecurity_audit_wyden/) "隐瞒了关键的网络安全报告"，该报告涉及用于急救响应人员和美国军方的FirstNet电话网络。

美国参议员Ron Wyden（民主党-俄勒冈州）致函美国政府的CISA和NSA，要求对FirstNet进行每年一次的网络安全审计，原因是SS7的误用问题。

"这些电话网络漏洞正在积极被利用，用于跨境监视," Wyden写道。®
