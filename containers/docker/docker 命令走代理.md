# docker 命令走代理

docker 命令分为三种场景，dockerd，docker run，docker build。

通常配置all_proxy，http_proxy，https_proxy无法生效，需按照下面的方式配置。

如果代理地址是本机localhost，则需加入 --network host 才会生效

## dockerd

拉取镜像是，其实是dockerd在执行，所以要为它配置代理，它是一个守护进程，所以要增加systemd的配置。
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```

在proxy.conf （可以命名为任意xxx.conf文件）写入

```conf
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

## docker run

docker >= 17.07

~/.docker/config.json
```json
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://proxy.example.com:8080",
     "httpsProxy": "http://proxy.example.com:8080",
     "noProxy": "localhost,127.0.0.1,.example.com"
   }
 }
}
```

## docker build

docker build 时注入参数

```bash
docker build . \
    --build-arg "HTTP_PROXY=http://proxy.example.com:8080/" \
    --build-arg "HTTPS_PROXY=http://proxy.example.com:8080/" \
    --build-arg "NO_PROXY=localhost,127.0.0.1,.example.com" \
    -t your/image:tag
```

## 重启生效

针对 dockerd

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
