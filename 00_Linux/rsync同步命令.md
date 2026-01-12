如果你已经可以使用 ssh FD-201 登录远程服务器（FD-201 是服务器别名），可以直接使用 rsync 快速同步文件夹。下面是具体的方法：

**1. 基本 rsync 命令**



在 macOS 终端执行：

```
rsync -avz ~/Downloads/ FD-201:~/qianlf/SeaFog/Data/DataSet/
```

或者从远程服务器同步到本地：

```
rsync -avz FD-201:~/qianlf/SeaFog/Data/DataSet/Fog_2022~2024/Fog_FY4B_ERA5_cloud_V3_2022~2024.feather ~/Downloads/
```

**参数说明**

​	•	-a：归档模式，保留权限、时间戳、符号链接等。

​	•	-v：显示详细信息（可选）。

​	•	-z：压缩数据，提高传输速度。

​	•	--progress：显示同步进度（可选）。

​	•	--delete：删除目标目录中不存在的文件（谨慎使用）。

**2. 断点续传**



如果同步中断，可以使用 --partial 和 --progress 继续：

```
rsync -avz --partial --progress ~/local_folder/ FD-201:/remote_folder/
```

**3. 限制带宽（避免影响网络）**



如果网络带宽有限，可以限制传输速度：

```
rsync -avz --bwlimit=5000 ~/local_folder/ FD-201:/remote_folder/
```

​	•	--bwlimit=5000：限制速率为 5000KB/s（5MB/s）。

**4. 排除某些文件**



如果不想同步某些文件或文件夹，例如 .git 目录或 *.log 文件：

```
rsync -avz --exclude ".git" --exclude "*.log" ~/local_folder/ FD-201:/remote_folder/
```

**5. 只同步有更新的文件**



如果你只想同步发生变化的文件（增量同步）：

```
rsync -avzu ~/local_folder/ FD-201:/remote_folder/
```

​	•	-u 选项：只同步比目标文件新的文件。

**6. 并行加速（推荐）**



rsync 默认是单线程运行，可以用 --info=progress2 提高速度：

```
rsync -avz --info=progress2 ~/local_folder/ FD-201:/remote_folder/
```

如果要更快，可以结合 rsync + tar + ssh（适用于超大文件）：

```
tar cf - ~/local_folder | ssh FD-201 "tar xf - -C /remote_folder/"
```

**7. 使用 rsync 配置 SSH 端口**



如果 SSH 连接 FD-201 需要使用非默认端口（比如 2222），可以这样做：

```
rsync -avz -e "ssh -p 2222" ~/local_folder/ FD-201:/remote_folder/
```

**8. 后台同步（避免 SSH 断连）**



如果同步需要很长时间，可以让 rsync 在后台运行：

```
nohup rsync -avz ~/local_folder/ FD-201:/remote_folder/ > rsync.log 2>&1 &
disown
```

​	•	nohup：忽略 HUP 信号，避免 SSH 断开导致 rsync 终止。

​	•	&：后台运行。

​	•	disown：解除进程与终端的关联，防止退出终端后进程被终止。

**9. 定时自动同步**



如果你想定期同步，可以使用 cron：

```
crontab -e
```

添加如下任务（每 30 分钟同步一次）：

```
*/30 * * * * rsync -avz ~/local_folder/ FD-201:/remote_folder/
```

**10. 总结**

| **任务**       | **命令**                                                     |
| -------------- | ------------------------------------------------------------ |
| 基本同步       | rsync -avz ~/local_folder/ FD-201:/remote_folder/            |
| 断点续传       | rsync -avz --partial --progress ~/local_folder/ FD-201:/remote_folder/ |
| 限制带宽       | rsync -avz --bwlimit=5000 ~/local_folder/ FD-201:/remote_folder/ |
| 仅同步更新文件 | rsync -avzu ~/local_folder/ FD-201:/remote_folder/           |
| 排除某些文件   | rsync -avz --exclude ".git" --exclude "*.log" ~/local_folder/ FD-201:/remote_folder/ |
| SSH 端口       | rsync -avz -e "ssh -p 2222" ~/local_folder/ FD-201:/remote_folder/ |
| 后台运行       | nohup rsync -avz ~/local_folder/ FD-201:/remote_folder/ > rsync.log 2>&1 & disown |

如果你的数据量较大，推荐 **rsync --info=progress2** 来提升传输效率！🚀