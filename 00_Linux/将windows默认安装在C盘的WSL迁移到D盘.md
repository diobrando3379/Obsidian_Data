### 将windows默认安装在C盘的WSL迁移到D盘

```
wsl --shutdown
```

```
wsl --export Ubuntu-24.04 D:\Ubuntu-24.04.tar
```

```
wsl --unregister Ubuntu-24.04
```

自己创建文件夹`D:\Ubuntu-24.04`

```
wsl --import Ubuntu-24.04 D:\Ubuntu-24.04 D:\Ubuntu-24.04.tar --version 2
```

## 可选

删除`C:\Users\33796\AppData\Local\Packages`中的ubuntu文件夹

***

### 修改默认用户为普通用户

WSL2 的默认用户可以通过修改 `/etc/wsl.conf` 文件来设置。如果该文件不存在，你可以手动创建它。

#### 使用 `vim` 编辑 `/etc/wsl.conf`

1. 打开 WSL2 终端。

2. 输入以下命令以 `root` 身份编辑 `/etc/wsl.conf` 文件：

   bash

   复制

   ```
   sudo vim /etc/wsl.conf
   ```

3. 在 `vim` 中，按 `i` 进入插入模式，然后添加以下内容：

   ```
   [user]
   default=your-username
   ```

   将 `your-username` 替换为你的普通用户名（例如 `user`）。

4. 按 `Esc` 退出插入模式，然后输入 `:wq` 保存并退出 `vim`。