# PandoraNext

## 简单介绍

* Pandora Cloud + Pandora Server + Shared Chat + BackendAPI Proxy = `PandoraNext`（[演示站](https://chat1.zhile.io)）
* 支持GPTs，最新UI。
* 支持多种登录方式：（相当于Pandora Cloud）
  * 账号/密码
  * Access Token
  * Session Token
  * Refresh Token
  * Share Token
* 可内置tokens（可使用上述所有Token），支持设置密码。（相当于Pandora Server）
* 可配置共享的tokens，会有一个功能等同[chat-shared3.zhile.io](https://chat-shared3.zhile.io)的共享站（目前4331个普号、116个Plus）。
* 为全代理模式，你的用户只需要跟你的部署网络能通即可。
* 可启动为BackendAPI Proxy模式，直接使用`Access Token`调用`/backend-api/`的接口。
* 还有疑问，那就进Telegram群让大家围观围观：[@ja_netfilter_group](https://t.me/ja_netfilter_group)

## 手动部署

* 在[Releases](https://github.com/pandora-next/deploy/releases)中下载对应操作系统和架构的包。
* 解压后修改同目录中的`config.json`至你需要的参数。
* [获取license_id](#%E5%85%B3%E4%BA%8E-license_id)填写在`config.json`中，这是必须的前置步骤！
* 各种Linux/Unix系统使用`./PandoraNext`启动即可。
* Windows系统双击`PandoraNext.exe`即可，当然最好在cmd中启动。

## Docker Compose 部署

* 仓库内已包含相关文件和目录，拉到本地，[获取license_id](#%E5%85%B3%E4%BA%8E-license_id)填写在`data/config.json`中。
* `data`目录中包含`config.json`、`tokens.json`示例文件可自行修改。
* `docker-compose up -d` **原神启动！**

## Docker 部署

```bash
$ docker pull pengzhile/pandora-next
$ docker run -d --restart always --name PandoraNext --net=bridge \
    -p 8181:8181 \
    -v ./data:/data \
    -v ./sessions:/root/.cache/PandoraNext \
    pengzhile/pandora-next
```

* 容器内默认监听`8181`端口，映射宿主机的`8181`端口，可自行修改。
* 你可以映射目录到容器内的`/data`目录，`config.json`、`tokens.json`和[获取license_id](#%E5%85%B3%E4%BA%8E-license_id)填写在`config.json`中。
* 你可以映射目录到容器内的`/root/.cache/PandoraNext`目录，保留登录的`session`，避免重启容器登录状态丢失。

## Nginx 配置

```
server {
	listen 443 ssl http2;
	server_name chat.zhile.io;
	
	charset utf-8;
	
	ssl_certificate      certs/chat.zhile.io.crt;
	ssl_certificate_key  certs/chat.zhile.io.key;

	...省略若干其他配置...
	
	location / {
		proxy_http_version 	1.1;
		proxy_pass 		http://127.0.0.1:8181/;
		proxy_set_header	Connection		"";
		proxy_set_header   	Host			$http_host;
		proxy_set_header 	X-Forwarded-Proto 	$scheme;
		proxy_set_header   	X-Real-IP          	$remote_addr;
		proxy_set_header   	X-Forwarded-For    	$proxy_add_x_forwarded_for;
		
		proxy_buffering off;
		proxy_cache off;
		
		send_timeout 600;
		proxy_connect_timeout 600;
		proxy_send_timeout 600;
		proxy_read_timeout 600;
	}

	...省略若干其他配置...
}
```

* Nginx建议开启`http2`。
* 以上仅为推荐配置，可根据具体情况进行改动。
* 建议开启`ssl`也即`https`，否则浏览器限制将无法复制网页内容。

## config 配置

* 以下是一个示例`config.json`文件

```json
{
  "bind": "127.0.0.1:8181",
  "timeout": 600,
  "proxy_url": "",
  "license_id": "",
  "public_share": false,
  "site_password": "",
  "setup_password": "",
  "server_tokens": true,
  "server_mode": "web",
  "captcha": {
    "provider": "",
    "site_key": "",
    "site_secret": "",
    "site_login": false,
    "setup_login": false,
    "oai_username": false,
    "oai_password": false
  },
  "whitelist": null
}
```

* `bind`指定绑定IP和端口，在docker内，IP只能用`0.0.0.0`，否则映射不出来。
* **如果你不打算套nginx等反代，`bind`参数的IP请使用`0.0.0.0`！！！**
* `timeout`是请求的超时时间，单位为`秒`。
* `proxy_url`指定部署服务流量走代理，如：`http://127.0.0.1:8888`、`socks5://127.0.0.1:7980`
* `license_id`指定你的License Id，可以在[这里获取](#%E5%85%B3%E4%BA%8E-license_id)。
* `public_share`对于GPT中创建的对话分享，是否需要登录才能查看。为`true`则无需登录即可查看。
* `site_password`设置整站密码，需要先输入这个密码，正确才能进行后续步骤。充分保障私密性。
* `setup_password`定义一个设置密码，用于调用`/setup/`开头的设置接口，为空则不可调用。
* `server_tokens`设置是否在响应头中显示版本号，`true`显示，`false`则不显示。
* `server_mode`默认为`web`模式，新增`proxy`模式，可以将你部署的服务当作一个`ChatGPT`接口反代使用。会话额度消耗为`4`倍，无并发限制。
* `captcha`配置一些关键页面的验证码。
    * `provider`验证码提供商，支持：`recaptcha_v2`、`recaptcha_enterprise`、`hcaptcha`、`turnstile`、`friendly_captcha`。
    * `site_key`验证码供应商后台获取的网站参数，是可以公布的信息。
    * `site_secret`验证码供应商后台获取的秘密参数，不要公布出来。有些供应商也称作`API Key`。
    * `site_login`是否在全站密码登录界面显示验证码，`true`或`false`。
    * `setup_login`是否在设置入口登录界面显示验证码，`true`或`false`。
    * `oai_username`是否输入用户名界面显示验证码，`true`或`false`。
    * `oai_password`是否在输入登录密码界面显示验证码，`true`或`false`。
* `whitelist`邮箱数组指定哪些用户可以登录使用，用户名/密码登录受限制，各种Token登录受限。内置tokens不受限。
* `whitelist`为`null`则不限制，为空数组`[]`则限制所有账号，内置tokens不受限。
* 一个`whitelist`的例子：```"whitelist": ["mail2@test.com", "mail2@test.com"]```

## tokens 配置

* 以下是一个示例`tokens.json`文件

```json
{
  "test-1": {
    "token": "access token / session token / refresh token",
    "shared": true,
    "show_user_info": false
  },
  "test-2": {
    "token": "access token / session token / refresh token",
    "shared": true,
    "show_user_info": true,
    "plus": true
  },
  "test2": {
    "token": "access token / session token / refresh token / share token",
    "password": "12345"
  }
}
```

* `token`支持示例文件中所写的所有类型。`session token`和`refresh token`可自动刷新。
* 每个key被称为`token key`，可在登录框作用户名输入。如上：`test-1`、`test-2`等，随意更改。
* 如果设置了`password`则输入完`token key`进入输入密码页面输入匹配。
* 如果设置`shared`为`true`，则这个账号会出现在`/shared.html`中，登录页面会出现它的链接。
* 如果设置`shared`为`true`，则这个账号不能再在用户名登录框进行登录。
* `/shared.html`中的账号和共享站功能相同，可以自行设置隔离密码进行会话隔离。
* `plus`用来标识`/shared.html`上账号是否有金光，没有其他作用。
* `show_user_info`表示`/shared.html`共享时是否显示账号邮箱信息，GPTs建议开启。

## 设置界面

* 必须先在`config.json`中设置`setup_password`为非空！
* 浏览器打开：`<Base URL>/setup`，其中`<Base URL>`是你部署服务的地址。

## 关于 license_id

* 在这里获取：[https://dash.pandoranext.com](https://dash.pandoranext.com)
* 复制`License Id:`后的内容，填写在`config.json`的`license_id`字段。
* 注意检查不要复制到多余的空格等不可见字符。
* **没有固定IP的情况**，IP变动后会自动尝试重新拉取。

## 其他说明

* 如果你发现网页上不能复制，开启`https`或者使用`127.0.0.1`。
* 如果你正在自定义页面元素，请保留：
  * `Powered by PandoraNext`文字和链接。
  * `About PandoraNext`文字和链接。
* 如果要抹，请在**一百多个js**文件中去除若干`dom检测`代码。
* 抹了后果如何我不说。
* `PHP`是世界上最好的编程语言。

## 贡献者们

> 感谢所有让这个项目变得更好的贡献者们！

[![Contributors](https://contrib.rocks/image?repo=pandora-next/deploy)](https://github.com/pandora-next/deploy/graphs/contributors)

## Star历史

![Star History Chart](https://api.star-history.com/svg?repos=pandora-next/deploy&type=Date)
