## SecurityReview

主要针对业务设计阶段的安全评审，方案由`涂鸦安全`SDL 团队的小伙伴整理

# 目录索引

- [访问控制和鉴权](#访问控制和鉴权)
  - [越权访问-平行越权和垂直越权](#越权访问-平行越权和垂直越权)
  - [越权访问-前端隐藏菜单](#越权访问-前端隐藏菜单)
  - [未授权访问](#未授权访问)
- [支持用户输入内容](#支持用户输入内容)
  - [使用富文本](#使用富文本)
  - [支持用户插入超链接](#支持用户插入超链接)
  - [允许用户插入 iframe 源](#允许用户插入iframe源)
- [数据传输](#数据传输)
  - [中间人劫持](#中间人劫持)
  - [不必要的数据泄露](#不必要的数据泄露)
- [私有协议](#私有协议)
- [前端纯静态页面](#前端纯静态页面)
- [数据下载和导出](#数据下载和导出)
- [账号管理](#账号管理)
  - [账号权限变更](#账号权限变更)
  - [管理员新建账号](#管理员新建账号)
  - [变更或绑定邮箱/手机号](#变更或绑定邮箱/手机号)
- [通过第三方账号登录](#通过第三方账号登录)
- [授权登陆第三方](#授权登陆第三方)
- [获取外部资源](#获取外部资源)
- [退出登陆](#退出登陆)
- [文件上传](#文件上传)
- [查找或添加账号](#查找或添加账号)
  - [添加子账号](#添加子账号)
  - [查找或添加账号](#查找或添加账号)
  - [根据手机号/邮箱查找或添加账号](#根据手机号/邮箱查找或添加账号)
- [短信/邮箱/图形验证码](#短信/邮箱/图形验证码)
  - [图形验证码](#图形验证码)
  - [短信/邮箱验证码](#短信/邮箱验证码)
- [登录状态更改密码](#登录状态更改密码)
- [密码找回](#密码找回)
  - [用户信息传输](#用户信息传输)
  - [密码重置](#密码重置)
  - [短信/邮箱验证码](#短信/邮箱验证码)
- [用户登录](#用户登录)
  - [设计缺陷-登陆失败提示](#设计缺陷-登陆失败提示)
  - [用户登录-数据传输安全](#用户登录-数据传输安全)
  - [登录行为](#登录行为)
- [用户注册](#用户注册)
  - [用户注册-设计缺陷](#用户注册-设计缺陷)
  - [用户注册-数据传输安全](#用户注册-数据传输安全)
  - [用户注册-数据存储安全](#用户注册-数据存储安全)
- [日志规范](#日志规范)
  - [日志规范-数据存储安全](#日志规范-数据存储安全)

# 访问控制和鉴权

## 越权访问-平行越权和垂直越权

· 威胁描述  
平行越权和垂直越权  
· 安全需求  
遵循默认最小化权限原则，产品/项目经理对每个功能点必须明确给出访问权限定义。研发按照权限定义设计和开发接口权限校验逻辑。  
· 安全方案  
公共接口公共数据无需鉴权。
同类用户间的数据必须在后端根据 session 信息判断用户传入参数对应数据的归属关系，限制用户只能访问自己的数据。
权限不同的用户，低权限用户未经授权禁止访问高权限账号可访问的接口或数据。
用户 ID 和各种核心业务属性索引 ID 等，都需要使用足够长度和合理随机梳生成的方式生成的 ID，无法被猜测，同时可能的话，不允许重复。

## 越权访问-前端隐藏菜单

· 威胁描述  
单纯前端隐藏菜单等权限控制方式对攻击者完全无效。可轻易绕过，达到越权访问的目的。  
· 安全需求  
权限校验必须在后端进行。  
· 安全方案  
访问控制基于后端控制服务，不允许仅通过前端隐藏功能或页面的方式执行权限控制。后端执行的访问控制/鉴权规则和前端展现给用户的规则必须一致。

## 未授权访问

· 威胁描述  
业务规划上只开放给某个指定调用方的接口，如果未进行访问控制限制，可能导致接口被未授权恶意调用，产生风险。  
· 安全需求  
遵循最小权限原则，减小风险暴露面。  
· 安全方案  
可在平台上配置“访问控制”。根据业务需要设置黑白名单。

# 支持用户输入内容

## 使用富文本

· 威胁描述  
如果未对内容进行严格的过滤，可能产生 XSS 漏洞，攻击者可插入恶意 js 代码，对用户发起攻击。  
· 安全需求  
白名单的方式过滤标签属性。  
· 安全方案  
必须使用白名单的方式过滤标签属性

## 支持用户插入超链接

· 威胁描述  
未对用户输入的超链接参数进行过滤校验，攻击者可通过插入 javascript:伪协议的方式执行恶意 js 代码，或者插入恶意网址，对用户实施钓鱼攻击。  
· 安全需求  
过滤超链接参数值。  
· 安全方案  
1、白名单校验传入参数值（即 a 标签 href 属性值）是否以 http://或者 https://开头（强制）
2、白名单校验域名，只允许插入指定域名下的超链接（建议，具体看业务功能需求）

```
参考正则：^(http|https)://([\\w-@:]+\\.)_(tuya\\.com|wgine\\.com)(/._)?$
```

## 允许用户插入 iframe 源

· 威胁描述
未对用户输入的 iframe src 属性（如：视频源地址）进行过滤校验，攻击者可通过插入 javascript:伪协议的方式执行恶意 js 代码，或者在嵌入的页面中插入构造的恶意代码，以及插入钓鱼页面覆盖原有的父页面对用户实施钓鱼攻击。插入违法内容，产生平台安全合规事故。

· 安全需求
校验传入参数，前端限制 iframe 最大长宽，启用沙箱机制。
· 安全方案
1、src 属性必须校验协议，限制 http 和 https，同时进行 url 白名单正则校验。限制只允许插入指定域名下的 iframe 源，防止攻击者插入恶意页面。

```
参考正则：^(http|https)://([\\w-@:]+\\.)_(youtube\\.com|bilibili\\.com)(/._)?$
```

2、固定长宽，或限制最大长宽，防止子页面覆盖父页面。
3、使用沙箱（sandbox）机制，遵循最小必要原则配置相应选项满足业务需求

# 数据传输

## 中间人劫持

· 威胁描述
中间人劫持
· 安全需求
加密传输和消息认证
· 安全方案
基本要求：
公网开放的服务必须走 tls 协议加密传输。例如使用 https、mqtts，禁止直接使用 http、mqtt。
特殊要求：
适用于 APP/智能硬件/小程序等客户端或终端相互之间或与云端的交互
敏感的业务参数，如敏感用户信息，必须进行额外的加密处理，比如 AES-GCM-128。需使用具备安全强度的算法，并需要合理使用算法。密钥建议绑定用户身份状态，一次性，或仅在一定限制条件下有效
请求 body 和主要公共参数，比如接口版本号，客户端信息，时间戳等，都需要加签，签名算法建议 HMAC-SHA256。加签密钥可使用身份认证信息计算生成，比如使用 cookie/session/token。

## 不必要的数据泄露

· 威胁描述
为方便起见，许多开发人员通过 API 端点公开对象的所有属性。 这旨在允许前端开发人员访问所有必需的资源，但可能会导致意外的数据泄露。
· 安全需求
服务端 API 响应数据过滤
· 安全方案
准确定义要在 API 函数中返回的对象属性，而不是返回整个对象；

仅返回客户端从您的 API 函数请求的数据，而不是返回所有可用数据并期望客户端过滤它；

限制 API 函数中查询可能影响的记录数，以防止数据库记录的大量更新或泄露，根据 API 常规请求量，配置好 API 短时间过多访问的监控告警。

# 前端纯静态页面

· 威胁描述
前端 html、js、css 代码中存在内部域名/接口地址、用户手机号、密码等硬编码的敏感信息，未经检查发布后会导致敏感信息泄漏。

· 安全需求
敏感信息自查
· 安全方案
对代码进行敏感信息自查

# 数据下载和导出

· 威胁描述
未加密数据扩大了数据泄露风险

· 安全需求
加密数据文件
· 安全方案 1.使用安全的算法加密数据或对应数据文件。推荐 AES128/256 或 PGP 等算法。

2.密钥通过邮件或短信等用户特有的安全渠道，分发给客户。

· 威胁描述 1.数据下载链接未进行权限校验，拿到 URL 可以直接下载文件。

2.其他权限绕过下载数据文件的情况。

· 安全需求
保证数据文件下载流程中的文件安全。
· 安全方案 1.下载数据文件之前进行用户身份的二次验证，可以是当前用户密码等。

2.核实下载数据文件的用户和当前登录用户，以及数据归属用户是不是完全统一。

3.数据文件下载链接需要进行权限校验，不可直接访问直接能够下载。或者文件通过用户注册邮件发送。

4.如果是文件下载链接无法直接进行用户身份认证，链接需要包含时效不超过 15 分钟的一次性临时 token 作为认证。

# 账号管理

## 账号权限变更

· 威胁描述
当存在安全漏洞或者管理员电脑被他人控制时，权限未经授权变更未通知管理员，除了会无法第一时间得知安全风险，还会导致用户对产品安全失去信任。
· 安全需求
敏感操作变更需通知用户。
· 安全方案
当权限设置发生变更时，应记录好日志，并短信/邮件通知用户是否是本人在操作，告知可能存在的安全风险。

## 管理员新建账号

· 威胁描述
默认高权限账号会增加人为操作失误的安全风险。
· 安全需求
遵循最小权限原则。
· 安全方案
新建账号默认最小权限，需要额外管理员添加，才能产生权限。

## 变更或绑定邮箱/手机号

· 威胁描述
变更邮箱/手机号通常需要验证码的方式完成验证，功能实现时，当变更操作与短信/邮箱验证码验证拆分为两步操作时，容易导致攻击者直接调用最后一步变更/绑定操作，绕过验证校验，变更或绑定为任意邮箱/手机号。
· 安全需求
确保每一步关键操作都进行验证码校验。
· 安全方案
尽量在同一个接口中完成校验和关键业务操作，避免在前端将关键操作与短信/邮箱验证码验证拆分为两个接口。

如果一定要拆分，后端必须在关键业务操作时再次校验验证码是否正确。实现方案二选一：
1、每个接口都传入验证码参数进行校验。操作完成后，验证码失效。
2、验证码通过时，在 redis 中存储验证码校验状态，后面的接口调用，查询状态，并消费掉记录。

# 通过第三方账号登录

· 威胁描述
例如企业微信授权登录。可能产生 url 跳转漏洞、xss 漏洞。漏洞参考：
· 安全需求
严格校验跳转的目标 url，防止被恶意利用进行跳转钓鱼。
· 安全方案
参数校验正则可参考：^(http|https)://([\w-@:]+.)(tuyacn|tuyaus|tuyaeu|tuyain|wgine).com(/.)?$
上面只是参考，域名根据实际需求定，遵循最小必要原则。

# 授权登陆第三方

· 威胁描述
一旦 code 被攻击者窃取，可能在有效时间内劫持用户账号。
· 安全需求
遵循 oauth2 安全规范
· 安全方案
授权 code 必须保证一次有效，且时间限制最多不能超过 5 分钟，超时自动失效。

· 威胁描述
url 跳转漏洞、xss 漏洞、code 劫持都会直接影响用户账户安全。
· 安全需求
遵循 oauth2 安全规范
· 安全方案
授权跳转 url 进行白名单校验，遵循最小必要原则，防止 url 跳转漏洞劫持 code，或导致 XSS、钓鱼事件发生。

# 获取外部资源

· 威胁描述
SSRF（服务端请求伪造）漏洞，黑客可使用此功能点作为代理，对公司内网发起攻击。

· 安全需求
限制传入参数，防止 SSRF 漏洞。
· 安全方案
以下方案按优先级从高到低选择：
1、请求参数不传 url，传 id，后台存储 id 和 url 的关系。通过 id 取 url 进行加载。url 必须保证可信任。
2、不适用上述方案的，采用白名单的方式，校验 url 中的协议，域名，端口。
3、无法进行域名白名单限制的，请求服务需要与内网其他服务隔离，并且分配单独的公网 ip，防止通过线上出口白名单访问办公内网服务。后端代码里对域名进行解析，并判断是否为私有 ip 段。不符合规则的，不进行下一步的请求处理。

# 退出登陆

· 威胁描述
用户退出登陆，如果未及时清理会话信息，可能增加账号被劫持的风险。
· 安全需求
技术实现上要与功能逻辑保持一致，并且后端接口需要能够防止 CSRF 攻击。
· 安全方案
1、用户登出后应立即清理会话及其相关登录信息，此操作在前端后端均要同时触发。后端接口仅支持 POST 请求，禁止使用 GET 请求。防止 CSRF 攻击导致的拒绝服务漏洞（自动退出）。

2、如果登出接口需要支持跨域访问，请使用 csrf-token 或 CORS 白名单等方案，防止 CSRF 攻击。避免直接 GET 请求跳转访问。

# 文件上传

· 威胁描述
文件上传漏洞可能导致各种安全问题，最严重的可能会导致黑客获取服务器控制权限。
· 安全需求
使用统一上传方案，进行统一的安全管控。
· 安全方案
使用公司基础服务团队统一提供的文件上传服务，禁止将文件直接上传至应用服务器上。

· 威胁描述
攻击者上传可执行文件，可能导致 XSS，钓鱼等攻击的发生。
· 安全需求
文件类型白名单限制。
· 安全方案
前端接入方案：
必须在后台中配置文件名后缀白名单，遵循最小必要原则，比如图片只允许：png,jpg,gif,jpeg 后缀的文件上传。禁止上传 htm、html、shtml、svg、xml、exe 后缀文件。
服务端接入方案：
通过服务端生成文件后上传至云存储的情况，才允许使用服务端接入方案，否则一律使用前端上传方案。
服务端拼接前端输入生成 html 文件的场景中，必须进行严格的内容过滤。纯文本进行 html 实体转义

# 查找或添加账号

## 添加子账号

· 威胁描述
恶意攻击者可无限制批量创建子账号，导致对应手机号/邮箱无法注册登录。
· 安全需求
子账号数量上限限制。
· 安全方案
平台根据实际业务情况设置默认的子账号数量上限。

· 威胁描述
账号未做隔离区分，可通过添加账号功能重置任意账号密码。
· 安全需求
账号隔离，判断归属。
· 安全方案
子账号之间，必须隔离，判断归属。

如果允许同一手机号成为多个用户下的子账号，则必须在登录时，有办法区分归属，否则会导致账号混乱，用户无法登录。

## 查找或添加账号

· 威胁描述
攻击者可通过接口获取任意用户的敏感信息。
· 安全需求
接口返回信息遵循最小必要原则。
· 安全方案
接口返回信息遵循最小必要原则，仅返回满足功能的必要信息，避免返回多余的用户敏感信息。

同时注意，账号管理类功能通常不具备通讯录属性，只要能区分账号即可满足业务需求，除非极特殊情况，默认安全合规要求手机号、邮箱等个人信息展现时要进行脱敏（后端接口脱敏）

## 根据手机号/邮箱查找或添加账号

· 威胁描述
攻击者可自动化批量遍历用户，泄漏平台注册用户数据。
· 安全需求
防止攻击者遍历用户
· 安全方案
功能接口必须限制仅能在登陆态下访问，并从账号维度限制接口调用频率和单位时间调用总次数。

# 短信/邮箱/图形验证码

## 图形验证码

· 威胁描述
后端相关功能接口未验证，导致验证码失效，各类敏感功能的风控措置被绕过。
· 安全需求
保证验证码功能符合安全规范完整实现。
· 安全方案
目前公司统一使用极验验证码。使用时，后端接口必须进行校验，防止前端生效，后端不生效的情况发生。

验证码使用一次后失效，再次请求接口需要重新验证。

## 短信/邮箱验证码

· 威胁描述
验证码回显会导致验证码措施失效，产生任意用户账号劫持，任意用户密码重置等风险。
· 安全需求
禁止任何接口返回信息中携带验证码。
· 安全方案
短信/邮箱验证码禁止通过接口返回数据返回给前端。
· 威胁描述
短信/邮箱炸弹漏洞可导致用户被骚扰，影响企业形象。短信接口未做发送次数限制还会导致直接的资金损失。

· 安全需求
短信/邮箱验证码发送接口防刷。
安全方案
1、登录状态
短信/邮箱验证码接口必须设置单位时间内发送频率和发送总量限制。限制依据包括：当前登录用户、接收短信/邮件的目标（收信方）。
2、非登录状态
增加图形验证码（比如极验），防止机器自动化重放请求消耗资源。
同时对目标手机号/邮箱进行单位时间内发送频率和发送总量限制。

· 威胁描述
验证码被成功猜解。
· 安全需求
保证验证码长度和随机性，降低验证码被猜解成功的概率。
· 安全方案
1、短信/邮箱验证码输错 5 次后自动失效，需要重新获取新的验证码。
2、短信/邮箱验证码必须满足有效期最长不超过 30 分钟的限制（推荐 1-5 分钟），过期后立即失效。
3、短信/邮箱验证码使用一次后应该立即失效。

# 登录状态更改密码

## 图形验证码

· 威胁描述
用户离开电脑未锁屏等场景下，密码被他人修改。

· 安全需求
关键敏感操作二次验证。
· 安全方案
用户可以在登录状态更改其密码，并且更新密码需要验证用户最近的密码。

# 密码找回

## 用户信息传输

· 威胁描述
敏感信息在传输过程中泄漏。可能导致用户敏感信息被窃取等危害。
· 安全需求
数据传输安全要求
· 安全方案 1.验证码和登录凭证不允许直接在 GET 请求 URL 参数或错误详细中展示。 2.密码需要经过哈希后传输。（新业务哈希不允许使用 MD5，推荐使用 SHA256。）

# 用户登录

## 设计缺陷-登陆失败提示

· 威胁描述
如果设置“用户不存在”这类提示，攻击者可通过遍历账号的方式，判断用户是否在平台注册过，导致用户信息泄漏。
· 安全需求
统一错误提示
· 安全方案
用户登录失败的提示信息应为“用户名或密码错误”/“登录失败”等模糊提示。

## 数据传输安全

· 威胁描述
敏感信息在传输过程中泄漏。
· 安全需求
数据传输加密安全要求。
· 安全方案 1.登录凭证不允许直接在 URL 参数或错误详情中展示。 2.密码需要经过哈希后传输。（新业务哈希不允许使用 MD5，推荐使用 SHA256。）

## 登录行为

· 威胁描述
攻击者可暴力猜解用户密码，或实施撞库攻击。
· 安全需求
缓解暴力猜解。
· 安全方案
要有缓解暴力破解的能力，建议方案：
1、【必须】使用图形验证码（例如：极验）
2、单个账号，密码错误 5 次，锁定一段时间。
3、基于 IP 或设备指纹，做连续错误次数的账号锁定。

· 威胁描述
用户账号被盗，异常登录，用户无法第一时间知晓。
· 安全需求
加入风控策略，减小用户账号被盗后的风险。
· 安全方案
异常登录告警，可以通过用户注册邮件等方式发送告警。

# 用户注册

## 用户注册-设计缺陷

· 威胁描述
1、短信被刷导致直接资金损失。
2、短信炸弹漏洞影响用户。
3、防止遍历用户，导致平台注册用户信息泄漏。
4、导致自动化批量注册垃圾账户。
· 安全需求
短信/邮箱接口防刷
· 安全方案
短信/邮箱发送接口应限制针对同一手机号/邮箱的请求频率，同时使用图形验证码（例如极验）防止接口被刷，防止遍历手机号的方式消耗短信资源。

· 威胁描述
任意账号注册
· 安全需求
对账号注册行为真实性进行验证。
· 安全方案
应有二次审核机制，如手机/邮箱验证码等方式验证。

· 威胁描述
可使用弱密码、用户名枚举用户
· 安全需求
密码复杂度要求
· 安全方案
前端校验，密码长度至少 8 位。（目前主流合规要求至少 12 位） 密码复杂度为 3，密码必须包含大写字母、小写字母、数字、符号其中三种。

## 用户注册-数据传输安全

· 威胁描述
敏感信息在传输过程中泄漏。
· 安全需求
数据传输安全要求。
· 安全方案 1.登录凭证不允许直接在 URL 参数或错误详细中展示。 2.密码需要经过哈希后传输。（推荐使用 SHA256）

## 用户注册-数据存储安全

· 威胁描述
明文密码泄漏
· 安全需求
密码存储加密要求
· 安全方案
密码存储加盐哈希。 盐随机生成，至少 32 位，和密码分开存储。哈希函数不允许使用 MD5，推荐使用 SHA256。

# 日志规范

## 日志规范-数据存储安全

· 威胁描述
用户隐私泄漏
· 安全需求
日志中的敏感数据必须加密/脱敏后保存。
· 安全方案
用户敏感信息能不在日志中保存的尽量不在日志中保存。对于必须要保存的日志，按照数据脱敏/加密 规范进行处理后保存。
