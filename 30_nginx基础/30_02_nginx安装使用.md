


# 1 Linux (Ubuntu)

1. Aktualisieren Sie ggf. das Verzeichnis Ihres Package Managers mit dem Befehl `apt-get update`
2. Nutzen Sie `apt-get`, um nginx zu installieren: `apt-get install nginx`


der Installationsordner von nginx unter dem Pfad` /usr/local/etc/nginx/`


## 使用 

```sh
sudo nginx -t       # 检查配置文件语法是否正确
sudo systemctl reload nginx   # 重新加载配置（不中断连接）
```


# 2 MacOS

1. Installieren Sie ggf. Homebrew, indem Sie folgenden Befehl im Terminal eingeben: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
2. Nutzen Sie Homebrew, um nginx zu installieren, indem Sie folgenden Befehl im Terminal eingeben: `brew install nginx`


# 3 Windows 


## 3.1 安装 


Laden Sie sich nginx Version 1.25.3 für Windows herunter: https://nginx.org/download/nginx-1.25.3.zip

## 3.2 使用 


https://nginx.org/en/docs/windows.html

cd C:/01_SmallApp/nginx-1.25.3 后 再执行下面的命令 

- nginx -t
    - 测试一下 ngnix , 不会 启动它 
- 启动 nginx 
    - start nginx
    - 不可以用 nginx.exe 或者 nginx 会死机
- 终止
    - stop nginx:   
    - nginx -s stop   	fast shutdown
    - nginx -s quit   graceful shutdown
- 查看 nginx 是否在运行 
    - `tasklist /fi "imagename eq nginx.exe"`
    - `tasklist | findstr nginx`
- 关闭所有 nginx 程序 
    - cmd 中 `@taskkill /f /im nginx.exe`
    - powershell 中 : `Get-Process nginx | Stop-Process -Force`
-  Serverkonfiguration neu laden, conf 变更后 重新载入 
    - nginx -s reload


## 3.3 Run Nginx as Windows service

https://stackoverflow.com/questions/40846356/run-nginx-as-windows-service



## 3.4 **Configure Nginx** with **SSL/TLS Certificate**

https://medium.com/@chandramuthuraj/installing-nginx-on-windows-a-step-by-step-guide-6750575c63e2

