# PandoraNext

## 手动部署

* 在[Releases](https://github.com/pandora-next/deploy/releases)中下载对应操作系统和架构的包。
* 解压后修改同目录中的`config.json`至你需要的参数。
* 将获取到的`JWT Token`内容写进同目录的`license.jwt`文件中。
* 启动`PandoraNext`或`PandoraNext.ext`即可。

## Docker 部署

```bash
$ docker pull pengzhile/pandora-next
$ docker run -d --restart always --name PandoraNext --net=bridge -p 8181:8181 \
             -e PANDORA_NEXT_LICENSE="<JWT Token>" pengzhile/pandora-next

```

* 容器内默认监听`8181`端口，映射宿主机的`8181`端口，可自行修改。
* 你可以映射目录到容器内的`/data`目录，`config.json`和`tokens.json`放在其中。
* 自行使用真实的`JWT Token`替换命令中的`<JWT Token>`，没有`<`和`>`，不要搞错。

## Docker Compose 模版

```yaml
version: '3'
services:
  pandora-next:
    image: pengzhile/pandora-next
    container_name: PandoraNext
    network_mode: bridge
    restart: always
    ports:
      - "8181:8181"
    environment:
      - PANDORA_NEXT_LICENSE=<JWT Token>
    volumes:
      - ./data:/data
```

* 对照上述`Docker 部署`的内容自行修改。

* 映射了`/data`目录，要提供`config.json`，这是一个示例：

```json
{
  "bind": "0.0.0.0:8181",
  "timeout": 600,
  "proxy_url": ""
}
```

`/data`目录必须放在`docker-compose.yaml`同目录。

参考示例：
```
├── data
│   ├── config.json
│   └── tokens.json
└── docker-compose.yml
```

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

## 关于 license.jwt文件

* ~~目前手动私聊我，发服务器IP，我手动给你发。~~（手动工作量太大，已停发，请等web自助获取）
* 后续可在web页面中自助获取，正在开发中。
* **没有固定IP无法部署**，你哪怕用`proxy_url`参数指定一个机场代理呢。
* `PHP`是世界上最好的编程语言。
