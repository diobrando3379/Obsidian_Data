## VsCode远程连接服务器后安装Github Copilot无法使用

#### 在Vscode的settings中搜索Extension Kind



最下面添加配置

```
"remote.extensionKind": {
        "GitHub.copilot": ["ui"],
        "GitHub.copilot-chat": ["ui"],
    }
```



> “ui”：扩展在本地客户端运行
> “workspace”：扩展在远程服务器运行

这两个扩展始终在 本地客户端运行，即使你连接了远程开发环境。