---
description: 容器逃逸漏洞
---

# CVE-2020-15257-容器逃逸-containerd

1.搭建漏洞环境（ubuntu）

apt-cache madison docker-ce

sudo apt-get install docker-ce=18.03.0~ce-0~ubuntu

sudo apt-get install runc

2.查看组件版本

2.1 docker版本

![image-20210223115350889](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223115350889.png?lastModify=1614157954)

![](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223120127997.png?lastModify=1614157954)

2.2 containerd版本

![image-20210223121912694](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223121912694.png?lastModify=1614157954)

3.漏洞复现

3.1 正常创建容器

| Options | Mean |
| :--- | :--- |
| -i | 以交互模式运行容器，通常与 -t 同时使用； |
| -t | 为容器重新分配一个伪输入终端，通常与 -i 同时使用； |
| -d | 后台运行容器，并返回容器ID； |

sudo docker run -itd ubuntu /bin/bash

sudo docker containerd ls

![image-20210223123151119](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223123151119.png?lastModify=1614157954)

查看containerd-shim套接字

netstat -xl \| grep shim

![image-20210223123446745](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223123446745.png?lastModify=1614157954)

3.2 以host模式创建容器

宿主机：随便启动一个容器，让containerd-shim的unix socket暴露出来

```text
docker run -d -it ubuntu /bin/bash
```

现在要启动一个容器，我们通过exp逃逸这个容器：

```text
docker run --rm --net=host -it ubuntu /bin/bash
```

3.3 将编写好的exp传入容器内部：

docker cp exp ce23fae1a50f:/

![image-20210223132820260](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223132820260.png?lastModify=1614157954)

![image-20210223132836891](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223132836891.png?lastModify=1614157954)

3.4 执行exp反弹shell

![image-20210223134504723](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223134504723.png?lastModify=1614157954)

![image-20210223134254565](file://E:/bleke/%E4%B8%B4%E6%97%B6%E5%B7%A5%E4%BD%9C/%E9%A2%86%E5%AF%BC%E4%B8%93%E6%A0%8F/CVE-2020-15257-docker-containerd%E5%A4%8D%E7%8E%B0/success%E7%9A%84%E5%9B%BE%E7%89%87/image-20210223134254565.png?lastModify=1614157954)

成功获得shell

