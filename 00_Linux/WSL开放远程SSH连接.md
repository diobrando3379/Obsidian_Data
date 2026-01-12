```
netsh interface portproxy add v4tov4 listenport=10088 listenaddress=0.0.0.0 connectport=10088 connectaddress=$(wsl hostname -I)
```

```
netsh interface portproxy show all
```

```
netsh interface portproxy reset
```

### **1. 确保WSL的SSH服务已正确配置**

- **安装并启动SSH服务**：

  ```
  sudo apt update && sudo apt install openssh-server
  sudo service ssh start  # 启动SSH服务
  ```

- **检查SSH配置**：

  ```
  sudo nano /etc/ssh/sshd_config
  ```

  确保以下配置项正确：

  ```
  Port 3379                # 默认端口（可自定义）
  ListenAddress 0.0.0.0    # 允许所有IP连接
  PasswordAuthentication yes  # 启用密码登录
  PubkeyAuthentication yes #使用公钥和私钥对来登录 这个最安全
  PermitRootLogin yes      # 允许root登录（可选，根据需求调整）
  ```

  保存后重启SSH服务：

  ```
  sudo service ssh restart
  ```

  关闭WSL后打开

  ```
  wsl --shutdown
  ```
  
  创建密钥对
  
  ```
  cd .ssh
  ```
  
  ```
  ssh-keygen -t rsa -b 4096
  ```
  
  创建**authorized_keys**文件(名称必须一致)
  
  ```
  chmod 600 authorized_keys
  ```
  
  将id_rsa.pub的内容复制到**authorized_keys**中

------

### **2. 配置Windows防火墙**

- 允许SSH端口通过Windows防火墙：
  1. 打开 `控制面板` → `系统和安全` → `Windows Defender 防火墙` → `高级设置`。
  2. 新建入站规则：允许TCP和UDP端口 `3379`（或自定义端口）。

------

### **3. 设置WSL网络模式**

![[image-20250312122602855.png]]

### **4. 获取Windows主机的IP地址**

- 在Windows中打开命令提示符，运行：

  ```
  ipconfig
  ```

  找到本地局域网的IPv4地址（如 `192.168.1.100`）。

------

### **5. 从另一台电脑连接**

- 在另一台电脑上使用SSH客户端连接：

  ```
  ssh -p 3379 <WSL用户名>@<Windows主机的IP>
  ```

  示例：

  ```
  ssh -p 10088 diobrando@10.61.145.225
  ```
  
  内网穿透连接 将10088映射到公网端口55078 测试密码登录后关闭ssh密码登录来保证安全
  
  ```
  ssh -p 55078 diobrando@frp-bar.com
  ```
