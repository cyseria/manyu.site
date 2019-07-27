# 个人博客

## start

```bash
# 初次安装
git submodule update --init --recursive

# 后续更新
git submodule update
```

本地开发

```bash
# 本地调试
hugo server -D
# ip 访问（服务器调试）
hugo server --bind="0.0.0.0" -v -w -p 8080 -b http://192.168.1.8
```

编译

```
HUGO_ENV=production hugo --gc --minify
```



