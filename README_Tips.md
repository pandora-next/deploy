## 生成share token网址
https://ai.fakeopen.com/token

## pandora next的hagugingface部署示例
https://gordonchan-pandora-next.hf.space 来自大佬的https://github.com/renqabs/pdrn1

## pandoara next项目的应用
docker run -d -p 3000:3000
-e OPENAI_API_KEY= \ #这里可以写fk? -e BASE_URL=http://192.168.1.80:8181
yidadaa/chatgpt-next-web:latest #自定义 base_url使用pandora https://baseurl/prefix
#演示网站https://huggingface.co/spaces/gordonchan/pandora-web 直接填fk—

docker run -d -p 3002:3002
-e OPENAI_API_KEY=
-e OPENAI_API_BASE_URL=http://192.168.1.80:8181
chenzhaoyu94/chatgpt-web:latest
