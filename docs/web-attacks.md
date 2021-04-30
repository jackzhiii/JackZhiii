# 简述 CSRF, CORS 以及 HTTP Security headers
随着漏洞，入侵和数据偷窃的不断增加，一个 web 应用程序的安全变得尤为重要。

另外一方面，编程人员通常对攻击是如何发生的没有很深的理解，同时也不知道如何预防这些攻击。本文将尝试更近距离的了解一点这些陷阱。

## CSRF
***Cross-Site Request Forgery(跨站请求伪造)***是攻击者强制当前已经登陆站点的用户执行不利于站点的行为，例如伪造转账等行为

这有一个例子说明 ***Cross-Site Request Forgery*** 是如何工作的：
1. 你访问 *evil.com*

2. *evil.com* 拥有一个隐藏的表单，该表单用来提交请求来加载 *mybank.com/transfer-funds*。因为你已经登陆了 *mybank.com/transfer-funds*, 这个请求利用你的 *mybank.com* cookies, 然后悄悄的发起一个从你账户转出资金的转账操作。

3. 因为 *evil.com* 和 *mybank.com* 是不同源的，浏览器拒绝给*evil.com* 提供响应，因为有 **CORS(同源策略)**, 但是攻击者并不关系是否响应，因为资金已经被转账了，攻击者已经达到了目的。

现在如果 *mybank.com* 正确的实现了 **CSRF** 的保护：

1. 每一次 *mybank.com* 为用户提供一个表单，那么 *mybank.com* 服务器都会生成一个 **CSRF** 的 token, 并且作为一个隐藏字段插入到表单中。

2. 如果 *mybank.com* 服务器收到了一个 POST 请求，服务器将会去数据库中检测 **CSRF token**, 如果这个 token 存在且合法，那么这个 POST 请求将会被通过; 如果这个 **CSRF token** 不存在或者不正确，那么这个请求将会被拒绝。

**CSRF** 攻击的目标都是一些改变数据状态的请求(如 POST 和 PUT 请求), 而不是直接偷取数据, 因为攻击者并不在乎伪造请求的响应数据。**CSRF token** 的安全通过无状态的 token 来达到, 这些 token 永远不存存储在 cookies 中或者在永恒不变的存储中。通过这些手段就能抵御 **CSRF** 的攻击。

**CSRF** 保护支持需要被添加到你的应用程序代码中，而不能添加到代理服务层中(例如: Nginx). [A detailed look at CSRF from OWASP](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md)

一个好的实践是永远对 cookies 使用 *相同站点* 指令，来对 CSRF 攻击进行提供保护。