Notejs 使用教程

# 查看当前项目安装的包
```
> npm list -g --depth=0
> npm list -g
```
# 查看根详细配置
```
npm config list
```

# 查看全局包的安装位置
```
> npm root -g
```



# Ollama 操作工具
# 查看 Ollama 版本
```
ollama --version
```
# 查看帮助总览
```
ollama help
```

# 拉取/下载官方模型（不进入对话）
```
ollama pull 模型名
# 示例
ollama pull gemma4:26b
ollama pull qwen2.5:7b

```
# 查看本地已经下载的所有模型（硬盘里存的）
```
ollama list

```
# 删除本地模型（彻底删文件，释放硬盘）
```
ollama rm 模型名
# 示例
ollama rm gemma4:26b

```
# 更新已下载模型到最新版
```
ollama upgrade 模型名
```

## 启动 gemma4:26b 模型
```
ollama run gemma4:26b
```

## 关闭
```
ollama stop gemma4:26b
```

# 查看当前【正在后台占用内存/CPU/GPU】的模型
```
ollama ps

```
# 手动停止正在运行的模型，立刻释放资源
```
ollama stop 模型名
# 示例
ollama stop gemma4:26b
```
