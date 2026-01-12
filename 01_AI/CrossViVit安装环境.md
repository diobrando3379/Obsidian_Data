[CrossViVit](https://github.com/gitbooo/CrossViVit)安装环境

环境必须安装在python=3.9

```
pip install torch==1.13.1 \
            torchtext==0.14.1 \
            torchdata==0.5.1 \
            torchmetrics==0.11.0 \
            pytorch-lightning==1.8.6

```

```
pip install -r /home/LVM_date2/zjnu/qianlf/CrossViVit/requirements.txt --no-deps
```

### 为什么这样能行

- **手动先装**：你明确控制好了 torch 系列包的版本，保证它们之间互相兼容。
- **再 `--no-deps`**：pip 不再尝试为这些包重新做版本求解，就不会再报 “torchtext 要求 torch==1.13.0”、“torchdata 要求 torch==1.13.1” 这种矛盾了。