# 安全配置

有一些设置是敏感的，通过文件系统的权限来保护是不足够的。基于这种场景，elasticsearch提供了一个keystore，可以通过密码保护。`elasticsearch-keystore`就是用来管理与设置keystore的工具。

> 注意
>
> 所有的指令都必须使用运行elasticsearch的用户来执行。

## 创建keystore

使用`create`指令来创建`elasticsearch.keystore`：

```bash
bin/elasticsearch-keystore create
```

`elasticsearch.keystore`文件将被创建在` elasticsearch.yml`文件的旁边。

## 列出keystore的配置

使用`list`指令来列出`elasticsearch.keystore`的设置：

```bash
bin/elasticsearch-keystore list
```

## 添加一个String设置

一些敏感的字符串，像云平台插件的一个认真参数，可以通过`add`指令来添加：

```bash
bin/elasticsearch-keystore add the.setting.name.to.set
```

工具将提示这个值得设置。如果要通过控制台展示，使用`--stdin`参数：

```bash
cat /file/containing/setting/value | bin/elasticsearch-keystore add --stdin the.setting.name.to.set
```

## 删除设置

使用`remove`指令来从`keystore`中删除配置：

```bash
bin/elasticsearch-keystore remove the.setting.name.to.remove
```