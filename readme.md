# 个人博客

## Quick Start

```bash
# 初次安装
git submodule update --init --recursive

# 后续更新
git submodule update
```

### 本地开发

```bash
#install
brew install hugo

# 本地调试
hugo server -D

# ip 访问（服务器调试）
hugo server --bind="0.0.0.0" -v -w -p 8080 -b http://192.168.1.8
```

### 编译

推送服务器自动编译（通常不需要这一步，供在 server 调试时使用

```
HUGO_ENV=production hugo --gc --minify
```



