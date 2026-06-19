# 桃桃记账官网

本目录是「桃桃记账」官网静态站点，计划通过 git subtree 纳入主工程维护，并发布到 GitHub Pages 仓库 `changer0/taotao-ledger-official-website`。

| 项 | 值 |
| --- | --- |
| 上游仓库 | `https://github.com/changer0/taotao-ledger-official-website.git` |
| subtree 前缀 | `official-website/` |
| 远程别名 | `official-site-origin` |
| 上游分支 | `main` |
| 默认 Pages 地址 | `https://changer0.github.io/taotao-ledger-official-website/` |

## 本地预览

这个站点不需要构建工具。可以直接打开 `index.html`，也可以用静态服务预览：

```bash
python3 -m http.server 8787 --directory official-website
```

打开：

```text
http://localhost:8787/
```

## 首次配置 remote

新 clone 下来的仓库没有 `official-site-origin`，需要在仓库根目录执行一次：

```bash
git remote add official-site-origin https://github.com/changer0/taotao-ledger-official-website.git
```

检查：

```bash
git remote -v | grep official-site-origin
```

## 发布

`git subtree push` 只会推送已经提交的历史。改完站点后，先在主工程提交，再发布：

```bash
git add official-website
git commit -m "docs(site): 添加桃桃记账官网"
git subtree push --prefix=official-website official-site-origin main
```

如果上游仓库还是空仓库，第一次 `subtree push` 会创建 `main` 分支。

## 维护注意

- `index.html` 是官网首页，必须包含应用名称、产品展示、应用介绍、用户协议、版权所有者和联系方式等基础信息。
- `terms.html` 是用户协议页面。
- 隐私政策当前复用线上隐私站：`https://privacy.zhanglulu.com.cn/`。
- 如果运营主体、联系邮箱、隐私政策 URL 或用户协议发生变化，需要同步更新官网、App 内入口和相关合规文档。
- 不要把官网做成只有几行文字或单张图片，这不满足应用审核常规要求。
