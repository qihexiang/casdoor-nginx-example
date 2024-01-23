# Casdoor内网HTTPS访问

## 证书申请

如果没有证书，可以使用acme.sh进行证书的申请，使用docker中的acme.sh，指令如下：

```bash
mkdir certs
cd certs
docker run --rm -it \
    -v "$(pwd)":/acme.sh \
    -e CF_Token=exampletokenhere \
    -e CF_Zone_ID=examplezoneidhere \
    neilpang/acme.sh --issue --server letsencrypt --dns dns_cf \
    -d "tcub.site" \
    -d "*.tcub.site"

cd -
```

## Nginx配置文件

根据使用的Nginx版本，下载对应的源码包，从其中复制原始配置文件`conf`目录，复制到此处的`nginx`。此仓库中已有1.25.3版本所用的默认配置文件。

在`http`块中，可以声明反向代理上游服务器：

```conf
upstream casdoor {
    # casdoor是Docker Compose网络中根据services生成的主机名
    server casdoor:8000;
}
```

本例中，该配置放在`upstream.conf`中，在`nginx.conf`中通过`include upstream.conf;`调用。

解除https对应的`server`块（100行），修改证书和密钥文件的位置，本例中，通过Docker卷挂载到`/certs`目录下，对应位置见配置文件。

反向代理配置为：

```conf
location / {
    proxy_pass http://casdoor;
    proxy_set_header Host $host;
}
```

类似的，本例中通过`include casdoor.conf;`引入。

## Docker Compose

参考配置文件。

> rootless启动的docker和podman需要在防火墙中开放8443/tcp的访问，`firewall-cmd --add-port=8443/tcp`

## 添加解析

通过域名服务进行解析或本地hosts解析外部访问地址。添加后，可以通过https协议在8443端口访问到Casdoor服务。
