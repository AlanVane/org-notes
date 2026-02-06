@See https://v2raya.org/docs/prologue/installation/docker/

## 功能介绍
v2raya 是一个使用 Web UI 来配置 v2ray 的服务，方便好用，特别适合没有图形界面的 linux server 系统。
  
主要功能：
- 自动定时从机场订阅地址中获取更新配置
- 提供透明代理，系统代理，代理端口
- 提供路由服务，自动更新 GFWList
- 可视化的配置页面
- 几乎所有机场都能支持，完美替代 clash 功能
- 支持多节点负载均衡

项目地址：https://github.com/v2rayA/v2rayA

## 安装
我喜欢在 `Podman` 容器中使用最新版，然后使用 systemd 配置开机自动启动。

### 为你的 v2raya 存放创建数据和配置文件夹

```bash
mkdir -p ~/.config/v2raya
```
### 获取镜像
```bash
podman pull docker.io/mzz2017/v2raya
```

### 创建容器
```bash
podman create -it \
  --name v2raya \
  --restart=always \
  --label io.containers.autoupdate=registry \
  --cgroup-parent=v2raya.slice \
  --security-opt no-new-privileges \
  --cap-drop all \
  --network host \
  --memory=500M \
  --volume ~/.config/v2raya:/etc/v2raya:z \
  proxy.vvvv.ee/mzz2017/v2raya
```

成功运行后，默认的 Web UI 端口为：2017。

## 使用
  
通过访问 http://localhost:2017 ，初始登录建立管理账号.

### 创建服务
```bash
bash -c \
  "mkdir -p ~/.config/systemd/user \
  && cd ~/.config/systemd/user \
  && podman generate systemd --new --files --name v2raya \
  && systemctl --user daemon-reload"
```

### 开启容器自动更新
```bash
systemctl --user enable --now podman-auto-update.timer
```

### 开启 v2rayA 服务

现在你可以用systemd控制v2rayA服务了。

查看服务状态：
```bash
systemctl --user status container-v2raya.service
```

设置开机自启动，并且现在开始运行：
```bash
systemctl --user enable --now container-v2raya.service
```

打开浏览器，访问 [http://localhost:2017](http://localhost:2017/)
然后你可以在系统设置中使用代理： `http://localhost:20171` 。

### 服务器环境运行
v2rayA服务会跟随用户会话一起启动/停止。如果你想让v2rayA随系统启动，并且在用户会话结束后保持运行，使用以下命令：
```bash
loginctl enable-linger
```
