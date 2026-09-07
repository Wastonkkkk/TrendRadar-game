# TrendRadar 部署说明（花生大王专版）

> 4 步上手，让抖音游戏推广热点监控飞书群 30 分钟内跑起来。

---

## 📁 项目位置

```
D:\douyinbanyun\trendradar\
├── config\
│   ├── config.yaml              ← 主配置（已精简为 抖音/微博/百度 3 个平台）
│   └── frequency_words.txt      ← 频率词（已按游戏推广场景写好词组）
├── .github\workflows\
│   └── crawler.yml              ← GitHub Actions 主流程（默认每小时跑 1 次）
└── DEPLOY.md                    ← 本文件
```

---

## 🚀 4 步上手

### 第 1 步：Fork 仓库到你的 GitHub

浏览器打开：
**https://github.com/sansan0/TrendRadar**

点右上角 **"Use this template"** → **"Create a new repository"**

- Repository name: 建议 `TrendRadar-game`（或随便起）
- 选 **Public**（GitHub Actions 免费额度更宽松）
- 点 **"Create repository"**

---

### 第 2 步：把本目录的改推送上去

打开 PowerShell 或 Git Bash：

```bash
cd D:\douyinbanyun\trendradar

# 把 origin 改成你刚 fork 的仓库地址
git remote remove origin
git remote add origin https://github.com/你的用户名/TrendRadar-game.git

# 推上去
git add .
git commit -m "feat: 精简为游戏推广版（抖音/微博/百度）"
git push -u origin main
```

> 💡 如果你 fork 时仓库叫别的名字，把命令里的 `TrendRadar-game` 换成你的。

---

### 第 3 步：配飞书 Webhook Secret

**3.1 创建飞书群机器人**

1. 打开你截图里那个「热点监控」飞书群（或者建一个新的）
2. 群右上角 **更多** → **设置** → **群机器人** → **添加机器人** → **自定义机器人**
3. 名称：`TrendRadar 热点监控`
4. 头像/描述随便填，点 **添加**
5. ⚠️ **复制那个 webhook 地址**（格式 `https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxx`）
6. 点完成

**3.2 填进 GitHub Secrets**

回到你 fork 的 GitHub 仓库页面：
1. 顶部 **Settings** → 左侧 **Secrets and variables** → **Actions**
2. 点 **"New repository secret"**
3. Name（必须严格一致）：`FEISHU_WEBHOOK_URL`
4. Secret：粘贴刚才复制的 webhook 地址
5. 点 **"Add secret"**

> ⚠️ 千万不要把 webhook 地址写进代码里（会被 GitHub 公开），必须放 Secrets。

---

### 第 4 步：手动触发首次运行

1. 仓库顶部点 **Actions** 标签
2. 左侧选 **"Get Hot News"**
3. 右侧点 **"Run workflow"** → 绿色按钮 **"Run workflow"**
4. 等待 2-3 分钟
5. 打开飞书群 → 应该收到第一条热点推送 ✅

---

## ⏰ ⚠️ 7 天签到机制（重要！）

GitHub Actions 公共资源有 7 天试用期：
- 每 7 天会**自动停止**
- 必须**手动触发一次**"Check In" workflow 才能续期
- 忘了就停，3 个月累计跑不了几次

**每周日晚上 9 点提醒自己去签到**（可设个手机闹钟）：
1. 仓库 → **Actions** → **"Check In"** → **"Run workflow"**
2. 3 秒搞定，续期成功

> 想彻底免签到？改用 **Docker 部署**（自己机器 7×24 跑），本目录 `docker/` 下有 compose 文件，参考 README 里 "Docker 部署" 章节。

---

## 🎚️ 调频率（可选）

**当前默认**：每小时 1 次（`33 * * * *`）

**改成 30 分钟一次**（更紧）：
1. 编辑文件 `.github\workflows\crawler.yml`
2. 找到 `- cron: "33 * * * *"`，改成 `- cron: "*/30 * * * *"`
3. 提交推送

**改回 1 小时**：同理改回 `"33 * * * *"`。

> ⚠️ 越频繁 = GitHub Actions 配额消耗越快 + 7 天试用版越早停。**默认 1 小时够用**。

---

## ✏️ 改词 / 改配置

### 加新游戏（最常见）

打开 `config\frequency_words.txt`，在 `[WORD_GROUPS]` 下找位置加：

```yaml
[新游戏名]
新游戏搜索词1
新游戏搜索词2
!不想看的
@3
```

改完 → GitHub Actions 页面 → "Get Hot News" → "Run workflow" 立刻生效。

### 关掉某个平台

打开 `config\config.yaml` 第 64-74 行（platforms.sources），把不想监控的整块（4 行）删掉，提交推送。

当前生效 3 个：
- `douyin` 抖音 ✅
- `weibo` 微博 ✅
- `baidu` 百度热搜 ✅

---

## 🛠️ 故障排查

| 症状 | 原因 | 解决 |
|---|---|---|
| 飞书收不到消息 | Webhook 没填 / 填错 | 检查 GitHub Secrets 里 `FEISHU_WEBHOOK_URL` 是不是 webhook 完整地址 |
| Actions 报红 | 配置文件 YAML 缩进错 | 用 https://www.yamllint.com/ 校验，缩进必须是 2 空格 |
| 飞书收到但全是空的 | 频率词没匹配上 | 看 `config\frequency_words.txt` 是不是写错了，或词太冷门没人搜 |
| 7 天后停了 | 试用版到期 | 跑一次 "Check In" workflow 续期，或切 Docker |
| 推送重复太多 | 报告模式是 current | 把 `config.yaml` 第 156 行 `report.mode` 改成 `"incremental"`（只推新增） |
| 推送太少/太安静 | @数字太小 | 把 `frequency_words.txt` 里各词组末尾的 `@3` 调大（0=不限） |
| Actions 报"权限不足" | 仓库是 Private | 改成 Public（GitHub 免费 Actions 只对 Public 仓库友好） |

---

## 📞 项目原始仓库

- GitHub: https://github.com/sansan0/TrendRadar
- 完整文档: https://trendradar.sandev.cc/zh/docs/quick-start/
- 可视化配置编辑器: https://sansan0.github.io/TrendRadar/

---

**完成时间戳**：2026-09-07 14:30  
**部署者**：健聪（自动）  
**项目位置**：`D:\douyinbanyun\trendradar\`
