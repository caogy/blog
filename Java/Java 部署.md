# Java 部署

## 将 jar 包部署为 window 服务

使用 [nssm](https://nssm.cc/download) 将批处理文件(bat) 安装成 Windows 服务。

``` bat
java -jar myjava.jar
```

nssm 使用：

``` nssm
//安装服务
nssm install serviceName

//删除服务
nssm remove serviceName
```
