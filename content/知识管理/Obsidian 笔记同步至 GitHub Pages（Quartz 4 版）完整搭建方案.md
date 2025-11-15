
## 一、方案目标



1. 本地用 Obsidian 管理 Markdown 笔记

2. 通过 Git 同步笔记至 GitHub 仓库

3. 基于 Quartz 4 自动构建静态站点

4. 部署到 GitHub Pages 实现公网访问（仅需一次密码输入，后续全自动化）

## 二、前期准备

### 1. 工具与账号



| 名称                  | 用途                  | 备注                                                                                       |
| ------------------- | ------------------- | ---------------------------------------------------------------------------------------- |
| GitHub 账号           | 存储代码、部署 Pages       | 需创建仓库：`用户名.github.io`                                                                    |
| Git                 | 本地代码版本控制、同步至 GitHub | 安装后配置用户名 / 邮箱：`git config --global user.name "名字"`；`git config --global user.email "邮箱"` |
| Obsidian            | 本地笔记编辑工具            | 需将笔记目录与 Git 仓库关联                                                                         |
| PowerShell（Windows） | 配置 SSH 密钥、执行 Git 命令 | 管理员模式运行                                                                                  |

### 2. 仓库与分支规划



* **GitHub 仓库**：`用户名.github.io`（必须是这个命名，GitHub Pages 自动识别）

* **核心分支**：`v4`（Quartz 4 推荐分支，用于存储笔记源码和配置）

* **部署分支**：`gh-pages`（自动生成，存储 Quartz 构建的静态文件，无需手动修改）

## 三、步骤 1：本地 Obsidian 与 Git 仓库关联

### 1. 初始化本地 Git 仓库
1. 远程fork [jackyzha0/quartz v4](https://github.com/jackyzha0/quartz)分支，命名使用自己的GitHub username，当然，如果这个仓库不作为个人的博客站点首页，命名可以使用呢其他名称；
2. 新建本地文件夹（如 `D:\Obsidian-Git`），作为笔记与代码的统一目录
3. 把远程仓库克隆到本地
### 2. 关联 GitHub 远程仓库
1. 进入 GitHub 仓库 `用户名.github.io`，复制 **SSH 地址**（如 `git@github.com:Zoot-running/Zoot-running.github.io.git`）
2. 本地关联远程仓库：
```powershell
git remote add origin git@github.com:Zoot-running/Zoot-running.github.io.git
```
### 3. Obsidian 关联笔记目录
1. 打开 Obsidian → 打开仓库文件夹（`D:\Obsidian-Git`）
2. 在仓库根目录新建 `content` 文件夹（Quartz 4 规定的笔记目录），所有 Obsidian 笔记放在 `content` 下（可建子文件夹如 `content/知识管理`）
3. 新建 `.gitignore` 文件，忽略 Obsidian 配置文件（避免上传冗余内容）：
```gitignore
.DS_Store
.gitignore
node_modules
public
prof
tsconfig.tsbuildinfo
.obsidian
.quartz-cache
private/
.replit
replit.nix
content/.trash/
```
## 四、步骤 2：配置 SSH 密钥（避免每次提交输密码）
这么做是为了在powershell中使用git，如果习惯使用git bash，就走git bash 的配置，我没有配置。
### 1. 生成并加载 SSH 密钥
1. 执行以下命令生成密钥（一路回车，默认路径即可）：
```powershell
ssh-keygen -t filename -C "你的GitHub邮箱"
```
1. 配置 PowerShell 启动时自动加载密钥，新建 PowerShell 配置文件：
```powershell
notepad $PROFILE  # 打开配置文件，若无则自动创建
```
1. 粘贴以下内容（等待 500ms 确保服务启动）：
```
# ==============================================================
# PowerShell Profile: Load SSH Key
# ==============================================================

# 你的 SSH 密钥路径
$sshKeyPath = Join-Path $env:USERPROFILE ".ssh\key_file_name"

# 等待 ssh-agent 服务启动（给系统一点时间）
Start-Sleep -Milliseconds 500 

# 加载 SSH 密钥
if (Test-Path $sshKeyPath) {
    Write-Host "Loading SSH key: $sshKeyPath"
    ssh-add $sshKeyPath
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "SSH key loaded successfully."
    } else {
        Write-Warning "Failed to load SSH key. You might need to enter your passphrase."
    }
} else {
    Write-Warning "SSH key not found at: $sshKeyPath. Please check the path."
}
```
保存配置文件，重启 PowerShell，首次启动输入 SSH 密钥密码（仅一次）。
### 2. 配置 Git 使用 Windows 自带 SSH

避免 Git 用自带 SSH 导致密钥不识别，执行以下命令：
```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```
## 五、步骤 3：部署 Quartz 4 并配置构建
### 1. 关键配置文件修改（核心步骤）
#### （1）`quartz.config.ts`（Quartz 4 主配置）
路径：仓库根目录 → `quartz.config.ts`，修改以下 3 处：
```TypeScript
import { QuartzConfig } from "./quartz/cfg"
import * as Plugin from "./quartz/plugins"

/**
 * Quartz 4 Configuration
 *
 * See https://quartz.jzhao.xyz/configuration for more information.
 */
const config: QuartzConfig = {
  configuration: {
    pageTitle: "我的知识库",  // 可选：改成你想要的站点标题（比如和Obsidian仓库同名）
    pageTitleSuffix: "",
    enableSPA: true,
    enablePopovers: true,
    analytics: {
      provider: "plausible",
    },
    locale: "en-US",
    baseUrl: "https://username.github.io",  // 修正：必须是完整HTTPS地址（你的GitHub Pages域名）
    output: "public",  // 关键添加：指定构建输出到public目录（对齐官方默认和工作流）
    ignorePatterns: ["private", "templates", ".obsidian"],
    defaultDateType: "modified",
    theme: {
      fontOrigin: "googleFonts",
      cdnCaching: true,
      typography: {
        header: "Schibsted Grotesk",
        body: "Source Sans Pro",
        code: "IBM Plex Mono",
      },
      colors: {
        lightMode: {
          light: "#faf8f8",
          lightgray: "#e5e5e5",
          gray: "#b8b8b8",
          darkgray: "#4e4e4e",
          dark: "#2b2b2b",
          secondary: "#284b63",
          tertiary: "#84a59d",
          highlight: "rgba(143, 159, 169, 0.15)",
          textHighlight: "#fff23688",
        },
        darkMode: {
          light: "#161618",
          lightgray: "#393639",
          gray: "#646464",
          darkgray: "#d4d4d4",
          dark: "#ebebec",
          secondary: "#7b97aa",
          tertiary: "#84a59d",
          highlight: "rgba(143, 159, 169, 0.15)",
          textHighlight: "#b3aa0288",
        },
      },
    },
  },
  plugins: {
    transformers: [
      Plugin.FrontMatter(),
      Plugin.CreatedModifiedDate({
        priority: ["frontmatter", "git", "filesystem"],
      }),
      Plugin.SyntaxHighlighting({
        theme: {
          light: "github-light",
          dark: "github-dark",
        },
        keepBackground: false,
      }),
      Plugin.ObsidianFlavoredMarkdown({ enableInHtmlEmbed: false }),
      Plugin.GitHubFlavoredMarkdown(),
      Plugin.TableOfContents(),
      Plugin.CrawlLinks({ markdownLinkResolution: "shortest" }),
      Plugin.Description(),
      Plugin.Latex({ renderEngine: "katex" }),
    ],
    filters: [Plugin.RemoveDrafts()],
    emitters: [
      Plugin.AliasRedirects(),
      Plugin.ComponentResources(),
      Plugin.ContentPage(),
      Plugin.FolderPage(),
      Plugin.TagPage(),
      Plugin.ContentIndex({
        enableSiteMap: true,
        enableRSS: true,
      }),
      Plugin.Assets(),
      Plugin.Static(),
      Plugin.Favicon(),
      Plugin.NotFoundPage(),
      // Comment out CustomOgImages to speed up build time
      // Plugin.CustomOgImages(),  // 可选：如果之前构建报错，可先注释掉（非核心插件）
    ],
  },
}

export default config
```
#### （2）`.github/workflows/quartz.yml`（CI 自动构建部署）
路径：仓库根目录 → `.github/workflows/quartz.yml`，替换为以下内容
```yaml
name: Deploy Quartz to GitHub Pages
on:
  push:
    branches: [v4]
  workflow_dispatch:

permissions:
  contents: write
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 确保拉取完整历史（Quartz 需用到）

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm install

      - name: Build Quartz site（输出到 public 目录）
        run: npx quartz build  # 按配置输出到 public 目录

      - name: Upload build artifacts（仅上传构建产物）
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public  # 只上传 public 目录（含 index.html、笔记等）

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages（官方组件，稳定无错）
        id: deployment
        uses: actions/deploy-pages@v4
```
#### （3）添加 `.nojekyll` 文件（禁用 GitHub 默认 Jekyll 构建）
在仓库根目录新建空文件，命名为 `.nojekyll`（无内容），作用：告诉 GitHub Pages 直接使用 Quartz 构建的静态文件，不触发 Jekyll 二次构建。
## 六、步骤 4：GitHub 仓库设置（部署关键）
### 1. 配置 GitHub Pages 部署源
1. 进入 GitHub 仓库 → 点击顶部 `Settings` → 左侧 `Pages`
2. **Build and deployment** 部分：
* `Source`：选择 `GitHub Actions`（不是 `Deploy from a branch`）
* 无需手动设置分支，CI 工作流会自动关联 `gh-pages`
### 2. 配置 Workflow 权限（避免 403 错误）
1. 仓库 `Settings` → 左侧 `Actions` → `General`
2. **Workflow permissions**：
* 选择 `Read and write permissions`
* 勾选 `Allow GitHub Actions to create and approve pull requests`
3. 点击 `Save` 保存
## 七、步骤 5：同步笔记并验证部署
### 1. 本地同步笔记到 GitHub
1. 在 Obsidian 中编辑笔记（所有笔记放在 `content` 下，需至少 1 篇 `.md` 文件，如 `content/知识管理/首页.md`）
2. PowerShell 执行以下命令提交推送：
```powershell
\# 查看变更
git status
\# 添加所有变更（包括笔记、配置文件）
git add .
\# 提交说明（自定义，如“添加知识库首页笔记”）
git commit -m "Add homepage note to content"
\# 推送到GitHub v4分支
git push origin v4
```
### 2. 验证 CI 构建部署
1. 进入 GitHub 仓库 → 点击顶部 `Actions` → 查看最新的 `Deploy Quartz to GitHub Pages` 工作流
2. 正常状态：
* `build` 任务：绿色对勾（构建成功，生成 `index.html`）
* `deploy` 任务：绿色对勾（部署成功，输出站点地址）
1. 查看构建产物：进入仓库 `gh-pages` 分支，确认根目录有 `index.html`、`static/`、`content/` 等文件（无嵌套 `public` 文件夹）
### 3. 访问站点
1. 部署成功后等待 3-5 分钟（GitHub Pages 缓存刷新）
2. 访问地址：`https://用户名.github.io`
3. 正常效果：显示 Obsidian 笔记生成的静态页面，支持笔记跳转、标签分类（Quartz 4 自带功能）
## 八、常见问题排查（所有讨论过的错误解决）

| 问题现象                                  | 原因分析                               | 解决方案                                                                           |
| ------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------ |
| 每次 Git 提交都要输密码                        | Git 用自带 SSH 客户端，未识别系统 SSH 密钥       | 执行 `git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"` |
| CI 构建报 “Unable to locate config file” | 混淆 Quartz 版本，用了 Hugo 配置            | 按步骤 3 配置 `quartz.config.ts`，指定 `output: "public"`                              |
| CI 报 “Liquid syntax error”            | 用了 Jekyll 构建，与 Quartz 冲突           | 添加 `.nojekyll` 文件，禁用 Jekyll                                                    |
| 站点 404，gh-pages 无 index.html          | 无有效笔记或插件报错导致未生成首页                  | `content` 下至少 1 篇 `.md` 笔记，有index.mdcunz ；                                     |
| 站点显示 XML 内容（无样式）                      | 仅生成站点地图 `index.xml`，无 `index.html` | 同 404 解决方案，确保 CI 构建生成 `index.html`                                             |
| CI 报 “Invalid workflow file”（YAML 错误） | YAML 缩进 / 参数错误                     | 替换 `quartz.yml` 为步骤 3 中的官方配置，确保缩进用空格                                           |
| CI 报 “Permission denied 403”          | Workflow 权限不足                      | 步骤 4 中设置 Workflow 为读写权限                                                        |

## 九、日常使用流程（自动化闭环）
1. **本地编辑**：在 Obsidian 中修改 / 新增笔记（均放在 `content` 下）
2. **提交推送**：PowerShell 执行 `git add .` → `git commit -m "备注"` → `git push origin v4`
3. **自动构建**：GitHub Actions 自动触发，构建静态站点并部署到 `gh-pages`
4. **访问更新**：等待 3 分钟，访问站点即可看到最新笔记（无需额外操作）