# 桃桃记账官网

本目录是「桃桃记账」官网静态站点，通过主工程中的 `official-website/` 目录维护，并发布到 GitHub Pages 仓库 `changer0/taotao-ledger-official-website`。

| 项 | 值 |
| --- | --- |
| 上游仓库 | `https://github.com/changer0/taotao-ledger-official-website.git` |
| subtree 前缀 | `official-website/` |
| 远程别名 | `official-site-origin` |
| 上游分支 | `main` |
| 默认 Pages 地址 | `https://changer0.github.io/taotao-ledger-official-website/` |
| 正式域名 | `https://taotao.zhanglulu.com.cn/` |

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

## 发布官网

官网发布分两步：**先把主工程提交推到 `origin/main`，再把 `official-website/` 目录发布到独立官网仓库。**

发布前先确认当前在主工程根目录：

```bash
cd /Users/lemon/Documents/MyProjects/02-owner/taotao-ledger-flutter
```

### 1. 本地预览与检查

```bash
python3 -m http.server 8787 --directory official-website
```

浏览器打开：

```text
http://127.0.0.1:8787/
```

至少检查首页、用户协议和移动端布局。确认无误后按 `Ctrl + C` 停止本地服务。

### 2. 提交并推送主工程

只暂存本次官网相关文件，不要顺手提交工作区里其他未完成内容：

```bash
git status --short
git add official-website
git add docs/ai/logs/<本次官网日志>.md
git commit -m "docs(official-website): 更新官网"
git push origin main
```

如果本次没有新增 AI 日志，第二条 `git add docs/ai/logs/...` 可以省略。

### 3. 优先尝试标准 subtree 发布

```bash
git subtree push --prefix=official-website official-site-origin main
```

如果成功，继续执行下面的「5. 发布后验证」。

### 4. subtree 提示 `no new revisions were found` 时

当前仓库历史中出现过 `git subtree push` 无法识别新修订、但 `official-website/` 实际已经变化的情况。此时不要把它当作发布成功，改用**目录快照发布**。

先生成当前官网目录对应的 Git tree，并读取官网远端 `main` 的父提交：

```bash
SITE_TREE=$(git rev-parse HEAD:official-website)
REMOTE_PARENT=$(git ls-remote official-site-origin refs/heads/main | awk '{print $1}')
```

如果官网远端已经存在 `main`，创建一个只包含当前官网目录内容的新提交：

```bash
PUBLISH_COMMIT=$(printf 'Publish official website snapshot\n' | \
  git commit-tree "$SITE_TREE" -p "$REMOTE_PARENT")
```

然后推送：

```bash
git push official-site-origin "$PUBLISH_COMMIT":main
```

如果远端还是空仓库、`REMOTE_PARENT` 为空，则第一次发布改为：

```bash
PUBLISH_COMMIT=$(printf 'Publish official website snapshot\n' | git commit-tree "$SITE_TREE")
git push official-site-origin "$PUBLISH_COMMIT":main
```

这套方式不会复制整个 Flutter 主工程，只会把 `official-website/` 当前目录树作为官网仓库根目录发布。

### 5. 发布后验证

先确认官网远端内容与当前 `official-website/` 完全一致：

```bash
git fetch official-site-origin main

LOCAL_SITE_TREE=$(git rev-parse HEAD:official-website)
REMOTE_SITE_TREE=$(git rev-parse FETCH_HEAD^{tree})

echo "local : $LOCAL_SITE_TREE"
echo "remote: $REMOTE_SITE_TREE"
```

两个 tree hash 必须一致。

然后验证正式站点：

```bash
curl -I https://taotao.zhanglulu.com.cn/
```

应返回 `HTTP 200`。GitHub Pages 发布完成后，再用浏览器打开：

```text
https://taotao.zhanglulu.com.cn/
```

如仍看到旧资源，先强制刷新浏览器，再检查 HTML/CSS/图片资源上的缓存版本参数是否已经更新。

### 一次完整发布的推荐顺序

```text
本地预览
→ git status 检查工作区
→ 只提交官网相关文件
→ git push origin main
→ git subtree push
→ 如 subtree 无新修订则执行目录快照发布
→ 对比 local/remote tree hash
→ 检查正式域名 HTTP 200
→ 浏览器最终验收
```

## 维护注意

- `index.html` 是官网首页，必须包含应用名称、产品展示、应用介绍、用户协议、版权所有者和联系方式等基础信息。
- `terms.html` 是用户协议页面。
- 隐私政策当前复用线上隐私站：`https://privacy.zhanglulu.com.cn/`。
- 如果运营主体、联系邮箱、隐私政策 URL 或用户协议发生变化，需要同步更新官网、App 内入口和相关合规文档。
- 官网独立仓库只应包含 `official-website/` 目录里的内容；不要把 Flutter 主工程文件直接推入官网仓库。
- 发布前必须先看 `git status --short`，避免把其他功能开发中的未完成文件混入官网提交。
- 不要把官网做成只有几行文字或单张图片，这不满足应用审核常规要求。
