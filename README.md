# DockerWorkflow
step by step show unity develop integrate docker workflow

## Goals

- 服务器容器化部署，本地只关心逻辑开发。部署使用方法、常用命令、通用配置，在这个演示中调试成熟。
- 在任意一台新主机上，用一个docker-compose文件，即可启动服务器。
- CI/CD，服务器开发过程中，提交即触发构建。
- 记录 dockerhub/github 整个使用流程。 另外还有 gitlab/Harbor/云仓库..使用到再说。


## 项目准备

1. 根目录存在 Server, Client 两个演示项目。
	- Server 是 ASP.NET Core Web API 示例项目：天气API。
	- Client 是 空目录，为了精准把握提交目录的写法，同时也与真实项目结构一致。
2. 启动服务器，确保正常访问 API。
3. 创建 Dockerfile，添加外部访问。
- ASP.NET Core 默认只监听 localhost，而容器需要监听 0.0.0.0 才能被外部访问。
```
ENV ASPNETCORE_URLS=http://+:80
```
4. docker build 打包。
```
docker build -t 给镜像取名 .
docker build -t Github用户名/给镜像取名 .
```

## DockerHub 手动

docker login 验证码+跳转网页登陆
docker login -u 【dockerhub的用户名】
或交互式提示输入密码，输入2次后登录成功。

docker tag local-image:tagname new-repo:tagname
	- docker tag webapi:1.0 setsuodu/webapi:1.0
docker push new-repo:tagname
	- docker push setsuodu/webapi:1.0
	
https://hub.docker.com/repositories 查看是否上传成功

## Github 手动

1. 创建 Personal access tokens (classic)
	- https://github.com/settings/tokens
	- 名称（Note）
	- 过期时间：不过期
	- 权限：
	✅ read:packages
	✅ write:packages
	✅ delete:packages（可选，用于删除镜像）
	✅ repo（如果仓库是私有的，需要这个权限）
	- 保存好token，只显示一次。
2. 登录 GHCR：
```
echo ghp_AbCdEf1234567890xyz | docker login ghcr.io -u setsuodu --password-stdin
```
登陆成功，显示
```
login Succeeded
```
3. 构建并标签镜像：
```
docker build -t ghcr.io/your-username/your-vs-project:latest .
docker build -t ghcr.io/setsuodu/weather-api:latest .
```
4. 推送：(200M+，主要是dotnet环境)
```
docker push ghcr.io/your-username/your-vs-project:latest
docker push ghcr.io/setsuodu/weather-api:latest
```
5. 镜像将出现在仓库的 Packages 标签下。
https://github.com/setsuodu?tab=packages
6. docker真机上运行
```
docker pull ghcr.io/setsuodu/weather-api:latest
docker run -p 5000:80 weather-api:latest
```

## Github CI