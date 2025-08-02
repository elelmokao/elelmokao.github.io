---
title: Docker Cheat Sheet
date: 2024-01-02 00:00:00 +0800
categories: [CheatSheet, Docker]
tags: [docker, cheatSheet]
# TAG names should always be lowercase
---
Update Date: 2024-01-02

## 1. Docker
### Build Images
```bash
docker build
```
[suffixes](https://docs.docker.com/engine/reference/commandline/build/#options)
> `--no-cached`: 避免在build 時被cache 住，而使修改的部分無法build


### Check Images
```bash
docker images
```
### Run Images
```bash
docker run {ImageName}
``` 
[suffixes](https://docs.docker.com/engine/reference/commandline/run/#options)

## 2. Dockerfile
### Format
The `dockerfile` helps us run the image automatically by just `docker run`.

The example is as follow [^df1]:
```Dockerfile
FROM centos:7
MAINTAINER jack

RUN yum install -y wget

RUN cd /

ADD jdk-8u152-linux-x64.tar.gz /

RUN wget http://apache.stu.edu.tw/tomcat/tomcat-7/v7.0.82/bin/apache-tomcat-7.0.82.tar.gz
RUN tar zxvf apache-tomcat-7.0.82.tar.gz

ENV JAVA_HOME=/jdk1.8.0_152
ENV PATH=$PATH:/jdk1.8.0_152/bin
CMD ["/apache-tomcat-7.0.82/bin/catalina.sh", "run"]

```
`FROM`: 會使用到的docker image 名稱<br>
`MAINTAINER`: 單純紀錄是誰寫的，也可用email<br>
`RUN`: 指令後面加linux command，可用來安裝或設定image 所需的東西
`ADD`: 把local 的東西複製到image 裡。若是tar.gz 被複製進去時，會自動解壓縮。類似的有像是`COPY`，但`ADD`更進階一些[^df2]
`ENV`: 設定環境變數
`CMD`: 在執行`docker run` 後啟動image 以後台的方式執行。

## Reference
[^df1]: https://ithelp.ithome.com.tw/articles/10191016?sc=hot
[^df2]: https://yeasy.gitbook.io/docker_practice/image/dockerfile/add
