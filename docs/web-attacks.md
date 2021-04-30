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


## CORS
***Cross_Origin Resource Sharing*** 仅仅适用于浏览器环境,并且它是是一种安全的机制来允许一个源(origin)对另外一个源(origin)发起一个请求。所有浏览器都遵循同源策略，意味着在默认情况下，一个源的脚本不能对不同源的站点发起请求 - 但是如果服务器配置了合适的 CORS heads, 这个策略会有选择性地放松。因此 CORS 是一种有选择性地放松安全地方式，而不是一直紧绷地策略。

当一个网站发起了一个 XHR 请求到另外一个源，浏览器首先发起了一个预先地 OPTIONS 的请求 - 如果发起请求的的域名在目标源服务器响应的允许源列表中，那么该请求才被发出。

注意 CORS 不为拥有默认 headers 的 GET HEAD POST 的请求发起预请求

一个 OPTIONS 请求的响应的关键 headers:

1. *access-control-allow-credentials:* 表示是否发送 cookies, 如果设置 true 的同时，ajax 请求设置 withCredentials = true, 浏览器的 cookies 将会被发送到目标服务器

2. *access-control-allow-origin:* 目标服务器允许发起请求的源列表, 或者 '*' 来允许任何源发起请求。如果 *access-control-allow-credentials* 已经设置了，那么不允许设置 *access-control-allow-origin* 为 '*', 否则浏览器会拒绝该请求

3. *access-control-allow-method:* 允许请求的 HTTP 方法列表 - POST, PUT等

之所以 *access-control-allow-origin* 不能被设置为 '*' 当 *access-control-allow-credentials* 被设置的时候，是为了保护开发人员走捷径，只是简单的设置 '*', 然后过了一段时间时间，把这些设置都忘记了 - 所以不能设置 '*' 的行为强制开发人员思考你提供的 API 将会如何被消费。

CORS 通常是一种代码味道 - 举个例子，CORS 的一种常用用法是允许多个站点获取通过一个 API, 就像 *mybankpersonal.com* 和 *mybankbusiness.com* 需要同时请求 *mybank.com* - 在这个例子中, 如果 web 站点服务使用相同的域名(mybank.com), CORS 可以被完全避免对于访问 API 网关, 无需任何 CORS 头需要被设置，并且浏览器可以使用完全的同源策略和提供最高级的安全。总而言之, CORS 仅仅为有选择性的减少安全而服务，而不是增加它的安全。

换言之，使用 CORS 是不可避免的, 如果你为使用浏览器请求的第三方提供 API

当然, CORS 对于浏览器之外的请求是不想干的, 例如: 使用 Curl 或者 使用 服务端到服务端的通信。