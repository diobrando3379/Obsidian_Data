```java
{
  "picBed": {
    "current": "aliyun",
    "aliyun": {
      "accessKeyId": "LTAI5tAygNuaHBDQ8nzuVrKw",
      "accessKeySecret": "ujAcqARCfXaVcCj4vuLUMEIXLwaXC1",
      "bucket": "typora-dio3379",
      "area": "oss-cn-hangzhou",
      "path": "",
      "customUrl": "",
      "options": ""
    }
  },
  "picgoPlugins": {}
}
```



在MacOS上有些麻烦

```
brew install node
```

```
npm install picgo -g
```

picgo-core需要选择`自定义命令`

命令一般为：

```
/opt/homebrew/bin/node /opt/homebrew/bin/picgo upload
```

node路径和picgo路径

确保路径正确：

```
which node
which picgo
```

