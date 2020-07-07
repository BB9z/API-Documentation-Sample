# 演示接口

可以调用的演示接口，通过 GitHub Pages 提供的静态数据，只支持 GET 方法，路径中需要带 json 扩展名。

可用的调用目前有：

数据 | 文档
----|----
[成功响应](success.json)   | [响应：成功或失败](../Sample/Home#response_success)
[失败响应](failure.json)   | [响应：成功或失败](../Sample/Home#response_success)
[/account/signinup/code](account/signinup/code.json) | [通过手机登录注册发送验证码](../Sample/Account#SignInUpSend)
[/account/signinup](account/signinup.json) | [通过手机登录注册](../Sample/Account#SignInUp)
[/account/login/mobile/code](account/login/mobile/code.json) | [手机登录发送验证码](../Sample/Account#LoginMobileCode)
[/account/login/mobile](account/login/mobile.json) | [手机验证码登录](../Sample/Account#LoginMobile)
[/account/login/password](account/login/password.json) | [密码登录](../Sample/Account#LoginPassword)
[/account/register/code](account/register/code.json) | [注册验证发送](../Sample/Account#RegisterCode)
[/account/register](account/register.json) | [注册](../Sample/Account#Register)
[/account/login/token](account/login/token.json) | [登录并刷新 token](../Sample/Account#LoginToken)
[/account/info](account/info.json) | [获取账户信息](../Sample/Account#AccountInfo)
[/account/info](account/info.json) | [修改账户信息](../Sample/Account#AccountEdit)
[/account/otac/send](account/otac/send.json) | [重置密码前发送验证码](../Sample/Account#OTACSend)
[/account/otac/verify](account/otac/verify.json) | [重置密码前身份验证](../Sample/Account#OTACVerify)
[/account/password/reset](account/password/reset.json) | [找回密码](../Sample/Account#PasswordReset)
[/account/password](account/password.json) | [修改密码](../Sample/Account#PasswordChange)
