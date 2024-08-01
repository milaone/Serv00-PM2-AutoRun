# Serv00-PM2-Autorun


旨在通过判断Web服务是否运行，否则通过远程方式登录Serv00的ssh运行脚本以恢复PM2快照 
本项目分三种方式来解决

### 利用vps远程运行serv00中的pm2保活命令

#### -Serv00中生成密钥对
```
ssh-keygen -t rsa -b 4096 -C "dino@milaone.app"

```




### 利用openwrt远程check https://memos.milaone.app 的运行状态，出错就ssh登录Serv00的ssh运行脚本



### 利用github的Actions中自动工作流脚本每5分钟check一下 https://memos.milaone.app 的运行状态，出错就ssh登录运行脚本