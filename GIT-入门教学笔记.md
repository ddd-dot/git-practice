# Git 入门教学笔记

> 面向完全没用过 git 的人。全程在 macOS + VS Code 环境下实操。
> 教学原则：**一次只做一件事，敲完看输出，看懂了再往下。**

---

## 目录

- [第 0 章 核心概念：四个区域](#第-0-章-核心概念四个区域)
- [第 1 章 准备工作](#第-1-章-准备工作)
- [第 2 章 只读三件套：status / log / diff](#第-2-章-只读三件套status--log--diff)
- [第 3 章 第一轮：新建文件走完整流程](#第-3-章-第一轮新建文件走完整流程)
- [第 4 章 第二轮：修改已有文件](#第-4-章-第二轮修改已有文件)
- [第 5 章 命令速查表](#第-5-章-命令速查表)
- [第 6 章 VS Code 面板对照表](#第-6-章-vs-code-面板对照表)
- [第 7 章 常见困惑 Q&A](#第-7-章-常见困惑-qa)
- [第 8 章 踩过的坑](#第-8-章-踩过的坑)
- [第 9 章 下一步学什么](#第-9-章-下一步学什么)

---

## 第 0 章 核心概念：四个区域

**这是全部 git 知识的地基。后面所有命令都是在这张图上搬运文件。**

```
①工作区            ②暂存区            ③本地仓库          ④远程仓库
(你编辑的文件)      (待打包的清单)      (.git 目录)       (GitHub 上)
     │                  │                  │                  │
     │─── git add ─────>│                  │                  │
     │                  │─ git commit ────>│                  │
     │                  │                  │─── git push ────>│
     │<───────────────── git clone / git pull ─────────────────│
```

### 三个必须理解的点

**1. 前三个区域都在你自己电脑上。**
只有 `push` / `pull` / `clone` 联网。很多人以为 `commit` 就上传了 —— 没有，commit 只是存进你本地的 `.git` 文件夹。

**2. 为什么要有"暂存区"这个中间层？**
因为你一次可能改了 10 个文件，但其中只有 3 个属于同一件事。暂存区让你**挑选**这次要打包哪些。

这是 git 比"网盘同步"高级的地方 —— 网盘是全量同步，git 是你主动组织的、**有意义的**变更记录。

**3. commit 是不可变的存档点。**
每个 commit 有唯一 ID（如 `ab1b2c9`），记录了"那一刻所有文件的完整快照 + 谁 + 何时 + 为什么"。这就是能回退的原因。

### 它们物理上在哪？

**都在同一个项目文件夹里，只是位置不同：**

```
~/git-practice/                    ← ①工作区 = 你能看到的文件
├── README.md                          Finder 里能看见、能双击打开
│
└── .git/                          ← ②③都藏在这个隐藏文件夹里
    ├── index                      ← ②暂存区（就是一个文件！约 137 字节）
    ├── objects/                   ← ③本地仓库：所有提交、所有历史版本
    ├── refs/                      ← 分支指针（main 指向哪个提交）
    ├── HEAD                       ← 你现在在哪个分支
    ├── config                     ← 本仓库配置（远程地址等）
    └── COMMIT_EDITMSG             ← 上次写的提交信息
```

**验证方法**（做完一次 commit 后跑）：

```bash
ls -la ~/git-practice/.git
```

对比时间戳：`git init` 时创建的文件是一个时间，`index` / `objects/` / `COMMIT_EDITMSG` 会是你 **commit 那一刻**的时间。你敲的命令确实改动了这些文件。

**由此推出三个结论：**

- 删掉 `.git` = 抹掉所有历史，但文件本体还在（变回普通文件夹）
- 复制整个项目文件夹给别人 = 连历史一起给了（`git clone` 就是通过网络做这件事）
- `.git` 通常比可见文件大得多，很正常 —— 它装的是全部历史快照

### git ≠ GitHub

| | 是什么 |
|---|---|
| **git** | 2005 年 Linus Torvalds 写的版本控制工具，装在你电脑上，**完全离线可用** |
| **GitHub** | 2008 年成立的**商业网站**，提供 git 托管，外加 issue / PR / Actions 等协作功能 |
| **remote** | 你的本地仓库记录的"另一个仓库在哪" |

remote 可以指向 GitHub、GitLab、Gitee、公司自建服务器，**甚至是你自己硬盘上的另一个文件夹或移动硬盘**。git 眼里"远程"只意味着"另一个 `.git` 仓库"。

**所以：`init` / `add` / `commit` / `status` / `log` / `diff` 全部不需要网络。**

---

## 第 1 章 准备工作

### 1.1 命令在哪敲？

三个地方任选：

| 方式 | 怎么做 |
|---|---|
| **Terminal.app** | `Command + 空格` → 输入 `Terminal` → 回车 |
| **VS Code 内置终端** | 菜单 Terminal → New Terminal，或 `Control` + `` ` `` |
| **Claude Code 会话** | 输入框第一个字符打 `!`，后面跟命令 |

### 1.2 检查环境

```bash
git --version                    # 有版本号就是装好了
git config --global user.name    # 你的名字
git config --global user.email   # 你的邮箱
```

**如果后两条是空的**，先配置（提交时会记录这个身份）：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

### 1.3 认识路径符号

`~` 代表你的**用户主目录**：

```
~/git-practice   等同于   /Users/你的用户名/git-practice
```

**它就是一个普通文件夹**，跟"桌面""下载"没有区别。

**实用技巧 —— 用 Finder 打开它，让抽象变具体：**

```bash
open ~/git-practice     # 用 Finder 打开这个文件夹
open .                  # 打开当前所在文件夹
```

在 Finder 里按 `Command + Shift + .`（句号）可以显示/隐藏隐藏文件，能看到 `.git`。

---

## 第 2 章 只读三件套：status / log / diff

**这三条命令绝对安全 —— 只看不改，随便敲。**

> 学 git 最大的心理障碍是怕弄坏。先把安全的命令用熟，建立信心。

### 2.1 `git status` —— 现在什么变了

```bash
git status          # 完整版，带大白话解释和下一步提示
git status -s       # 简短版，符号表
git status -s -u    # 简短版 + 展开文件夹
```

**状态符号：**

| 符号 | 含义 | git 认识这文件吗 | 危险性 |
|---|---|---|---|
| `M` | Modified，已跟踪文件被改了 | 认识，有历史 | 安全，能对比、能回退 |
| `D` | Deleted，已跟踪文件被删了 | 认识 | 安全，能恢复 |
| `A` | Added，已暂存 | 认识 | 安全 |
| `??` | Untracked，全新文件 | **不认识** | ⚠️ **删了永久丢失，无历史** |

**两列规则（重要）：**

```
第 1 列 = 暂存区状态
第 2 列 = 工作区状态
```

| 显示 | 读法 |
|---|---|
| `` `_M` ``（前面有空格） | 改了，**没**暂存 |
| `` `M_` ``（后面有空格） | 改了，**已**暂存 |
| `MM` | 暂存后又改了 |
| `??` | 两列都是 ?，完全未跟踪 |

**⚠️ 折叠陷阱：** 输出里结尾带 `/` 的行是**整个文件夹**。如果一个文件夹里所有文件都未跟踪，git 会折叠成一行，不逐个列。

> 真实案例：某仓库 `git status -s` 显示 23 行，加 `-u` 展开后是 **35 个文件** —— 3 个折叠的文件夹里藏着 15 个文件。

数一下有多少处改动：

```bash
git status -s | wc -l
```

**💡 读 git 的提示：** `git status` 输出里括号里那些 `(use "git add <file>..." to ...)` —— **git 在告诉你下一步能敲什么**。卡住时先读提示，答案常常就在里面。不用背命令。

### 2.2 `git log` —— 看历史

```bash
git log --oneline           # 每条提交压成一行
git log --oneline -5        # 只看最近 5 条
git log -5 --format="%h %an %ar %s"    # 哈希 作者 多久前 信息
```

**顺序是最新在上，往下越来越早。**

输出读法：

```
07cb874d feat: quench animation demo hook, visualizer fixes
└──┬───┘ └─┬─┘ └──────────────┬───────────────────────┘
   │       │                  └── 提交信息，写给未来的你看的
   │       └── 类型前缀（约定俗成，非 git 强制）
   └── commit 唯一 ID（哈希前 8 位），回退/对比/引用的凭据
```

**常见类型前缀：**

| 前缀 | 用于 |
|---|---|
| `feat:` | 新功能 |
| `fix:` | 修 bug |
| `docs:` | 文档 |
| `style:` | 格式/样式（不影响逻辑） |
| `refactor:` | 重构 |
| `test:` | 测试 |
| `chore:` | 杂活（依赖更新、配置调整等） |

### 2.3 `git diff` —— 具体改了哪几行

```bash
git diff              # 工作区 ↔ 暂存区（我改了但还没 add 的）
git diff --staged     # 暂存区 ↔ 最新提交（我这次 commit 会包含什么）★
git diff HEAD         # 工作区 ↔ 最新提交（全部改动）
git diff --stat       # 只看每个文件改了多少行（总览）
git diff 文件名        # 只看某个文件
```

**★ `git diff --staged` 是最实用的一条 —— commit 之前先看它一眼，确认你要提交的正是你以为的那些东西。这能拦下大部分"手滑提交了不该提交的东西"。**

（`--staged` 也可写作 `--cached`，完全等价，老教程常见后者。）

**输出读法：**

```diff
diff --git a/README.md b/README.md      ← a/=旧版  b/=新版
index 3684962..4ef7de7 100644           ← 内容的两个版本编号
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@                           ← 旧版1行 → 新版2行；@@ 是定位坐标
 # 我的练习仓库                           ← 开头是空格 = 这行没变，给你定位用
+这是第二行                              ← + = 新增
-被删掉的行                              ← - = 删除
```

- 一处**修改**会同时出现一个 `-` 和一个 `+`
- 只有 `+` = 纯新增；只有 `-` = 纯删除
- **git 只显示改动附近**，中间没动的几十行直接跳过

**⚠️ 两个常见困惑：**

1. **`git diff` 看不到 `??` 文件。** 因为 git 不认识它们，没有"之前的版本"可对比。文件必须先被 commit 过一次，diff 才有意义。
2. **`git add` 之后 `git diff` 就空了。** 不是文件被吞了 —— 因为工作区和暂存区内容已经一样了。要看内容得用 `git diff --staged`。

**⚠️ 分页器：** 输出很长时终端会进入分页模式，底部显示 `:` 或 `(END)`，键盘好像失灵。这时候 **空格翻页、`q` 退出**。这是 `less` 分页器的正常行为，不是卡死。

---

## 第 3 章 第一轮：新建文件走完整流程

**在一次性的练习仓库里做，别拿真项目练手。**

### 3.1 创建仓库

```bash
mkdir -p ~/git-practice && cd ~/git-practice && git init
```

输出：`Initialized empty Git repository in /Users/xxx/git-practice/.git/`

这一步就是创建了那个隐藏的 `.git` 文件夹（图里的③本地仓库）。

**去看一眼**（让抽象变具体）：

```bash
open ~/git-practice
```

Finder 弹出一个**空文件夹** —— 因为 `.git` 是隐藏的。

### 3.2 创建一个文件

```bash
cd ~/git-practice && echo "# 我的练习仓库" > README.md && ls -la
```

命令拆解：

| 部分 | 作用 |
|---|---|
| `cd ~/git-practice` | 进入文件夹 |
| `&&` | 前一条成功就继续下一条 |
| `echo "文字"` | 在屏幕上"说"这句话 |
| `> README.md` | **改成写进文件**（文件不存在就自动创建） |
| `ls -la` | 列出文件夹内容 |

**⚠️ `>` 和 `>>` 的区别（很容易手滑）：**

| 符号 | 作用 |
|---|---|
| `>` | **覆盖**，原内容全没了 |
| `>>` | **追加**，加在文件末尾 |

**验证**：回到 Finder 窗口，`README.md` 真的出现了。命令行操作的就是 Finder 里能看到的同一批文件，没有魔法。

`ls -la` 输出里，开头字母 `d` = 文件夹，`-` = 普通文件。

### 3.3 看 git 怎么说

```bash
git status
```

```
On branch main

No commits yet                    ← 仓库历史是空的

Untracked files:                  ← 未跟踪
  (use "git add <file>..." to include in what will be committed)
        README.md

nothing added to commit but untracked files present
```

**`Untracked` 的准确含义**：文件躺在文件夹里，git 看得见，但**没纳入管理** —— 不记录历史，不知道它改了什么，删了也救不回来。

**git 不会自动管理任何文件，必须你明确点名。** 这跟网盘"放进去就自动同步"完全不同，是刻意设计的 —— 因为很多文件（编译产物、缓存、几百 MB 的虚拟环境）根本不该进版本库。

### 3.4 `git add` —— 放进暂存区

```bash
git add README.md && git status
```

```
Changes to be committed:                      ← 新分区！这就是暂存区
  (use "git rm --cached <file>..." to unstage) ← git 又在告诉你怎么撤销
        new file:   README.md
```

变化：`Untracked files` 分区消失 → 出现 `Changes to be committed`。

**⚠️ 关键认知：`git add` 完全不改动文件内容。** 它只是在清单上打勾，说"这个我打算下次提交"。文件在硬盘上一个字节都没变，你去 Finder 里打开还是原样。

### 3.5 `git commit` —— 存进历史

```bash
git commit -m "docs: add readme"
```

```
[main (root-commit) ab1b2c9] docs: add readme
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

| 部分 | 意思 |
|---|---|
| `main` | 提交到了 main 分支 |
| `root-commit` | 这是仓库的**第一个**提交（以后不会再出现） |
| `ab1b2c9` | 这次提交的唯一 ID |
| `1 file changed, 1 insertion(+)` | 1 个文件，新增 1 行 |

### 3.6 确认干净

```bash
git status
```

```
On branch main
nothing to commit, working tree clean
```

**`working tree clean`** = 文件夹内容和最新提交完全一致，没有任何未保存的改动。

这是 git 里最让人安心的一句话 —— 你现在可以放心关机、放心去改文件。

### 3.7 连上 GitHub 并上传

**先确认目前没有远程：**

```bash
git remote -v
```

**输出空白** = 这是个纯本地仓库，跟网络毫无关系。

> `origin` 是"我的主要远程仓库"的**习惯叫法**，不是关键字，就像变量名。所有人都这么叫，别标新立异。

**上传（两种方式）：**

<details>
<summary><b>方式 A：用 gh CLI（一条命令搞定，推荐）</b></summary>

前提：装了 GitHub 官方 CLI 并登录过（`gh auth status` 检查）。

```bash
gh repo create git-practice --private --source=. --push
```

| 部分 | 作用 |
|---|---|
| `gh repo create` | 在你的 GitHub 账号下新建仓库 |
| `git-practice` | 仓库名 |
| `--private` | **私有**，只有你能看见 |
| `--source=.` | 用当前文件夹作为源，自动配置 `origin` |
| `--push` | 建好后立刻推上去 |

</details>

<details>
<summary><b>方式 B：手动三步（理解原理）</b></summary>

1. 去 github.com 网页上点 **New repository** 建一个空仓库（**别勾选** "Add a README"，否则会冲突）
2. 告诉本地仓库远程在哪：
   ```bash
   git remote add origin https://github.com/你的用户名/git-practice.git
   ```
3. 推上去（`-u` 建立追踪关系，只需第一次加）：
   ```bash
   git push -u origin main
   ```

</details>

**⚠️ `--private` vs `--public`：** 公开仓库**任何人都能看到你所有代码和提交历史**，一旦被 fork 或被搜索引擎收录，删了也可能留痕。练习用 private，传真项目前想清楚。

**成功输出：**

```
https://github.com/你的用户名/git-practice
To https://github.com/你的用户名/git-practice.git
 * [new branch]      HEAD -> main
branch 'main' set up to track 'origin/main'.
```

最后一行很重要 —— **本地 `main` 和远程 `origin/main` 建立了追踪关系**。以后只需敲 `git push`，不用每次写 `git push origin main`。

**去网页看一眼：**

```bash
open https://github.com/你的用户名/git-practice
```

GitHub 会自动把 `README.md` 渲染显示在首页（这是它的约定）。

**至此四个区域全部走通：**

```
①工作区 ✅ → ②暂存区 ✅ → ③本地仓库 ✅ → ④远程仓库 ✅
```

---

## 第 4 章 第二轮：修改已有文件

**第一轮是"新建文件"，这一轮是"修改已跟踪的文件" —— 这才是日常最常做的事。**

### 4.1 修改文件

```bash
cd ~/git-practice && echo "这是第二行" >> README.md
```

注意是 **`>>`**（追加），不是 `>`（覆盖）。

**没有输出 = 成功**（内容被导向文件，没打到屏幕上）。如果文字打在屏幕上了，说明你漏了 `>>`，文件没被修改。

### 4.2 看状态

```bash
git status -s
```

```
 M README.md
```

**`M` 前面有一个空格** = 改了，但没暂存。

**跟第一轮对比**：上次是 `??`（git 不认识），这次是 ` M`（git 认识它，且发现内容变了）。**这就是"跟踪"的价值** —— 一旦 commit 过一次，git 就开始盯着这个文件，任何改动都逃不掉。

### 4.3 看差异（这次能用了）

```bash
git diff
```

```diff
@@ -1 +1,2 @@
 # 我的练习仓库
+这是第二行
```

第一轮文件是 `??` 时 `git diff` 看不到它；现在被跟踪了，git 手上有上次提交的快照，可以对比了。

这次**没有 `-` 行**，因为你是追加，没删任何东西。

### 4.4 暂存

```bash
git add README.md && git status -s
```

```
M  README.md
```

**`M` 跑到第 1 列去了**（后面跟空格），跟刚才的 ` M` 正好相反：

| 显示 | 第1列(暂存区) | 第2列(工作区) | 含义 |
|---|---|---|---|
| `` `_M` `` | 空 | M | 改了，**没**暂存 |
| `` `M_` `` | M | 空 | 改了，**已**暂存 |

### 4.5 提交前的最后检查

```bash
git diff              # 空的！
git diff --staged     # 这里才有内容
```

**这是新手最常见的困惑。** 解释见 [2.3 节](#23-git-diff--具体改了哪几行)。

```
工作区 ──── git diff ────→ 暂存区 ──── git diff --staged ────→ 最新提交
```

### 4.6 提交

```bash
git commit -m "docs: add second line"
```

```
[main 492cb4f] docs: add second line
 1 file changed, 1 insertion(+)
```

这次**没有 `root-commit`** 了 —— 那是第一个提交的专属标记。

### 4.7 看一个新状态

```bash
git status
```

```
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

**`ahead of 'origin/main' by 1 commit`** = 本地比 GitHub 多 1 个提交。

| 术语 | 含义 | 怎么办 |
|---|---|---|
| **ahead** | 你有、远程没有 | `git push` |
| **behind** | 远程有、你没有 | `git pull` |

### 4.8 推送

```bash
git push
```

```
To https://github.com/你的用户名/git-practice.git
   ab1b2c9..492cb4f  main -> main
```

`ab1b2c9..492cb4f` = 远程从旧提交更新到了新提交。

**这次不用写 `git push origin main`** —— 上次建立了追踪关系，git 已经知道该推去哪。

---

## 第 5 章 命令速查表

### 只读（绝对安全，随便敲）

| 命令 | 作用 |
|---|---|
| `git status` | 现在什么变了（带解释和下一步提示） |
| `git status -s` | 简短版符号表 |
| `git status -s -u` | 简短版 + 展开折叠的文件夹 |
| `git log --oneline` | 提交历史，一行一条 |
| `git log --oneline -5` | 只看最近 5 条 |
| `git log --grep="关键词"` | 按提交信息搜索历史 |
| `git diff` | 工作区 ↔ 暂存区 |
| `git diff --staged` | 暂存区 ↔ 最新提交 ★ 提交前必查 |
| `git diff HEAD` | 工作区 ↔ 最新提交 |
| `git diff --stat` | 只看改动量总览 |
| `git remote -v` | 远程仓库地址 ★ push 前确认要传去哪 |

### 会写（改变状态，但都可撤销）

| 命令 | 作用 |
|---|---|
| `git init` | 创建仓库 |
| `git add 文件` | 工作区 → 暂存区 |
| `git add .` | ⚠️ 暂存**全部**改动，慎用 |
| `git restore --staged 文件` | 撤销暂存（暂存区 → 工作区） |
| `git commit -m "信息"` | 暂存区 → 本地仓库 |
| `git push` | 本地仓库 → 远程仓库 |
| `git pull` | 远程仓库 → 本地 |
| `git clone 网址` | 下载整个仓库（含全部历史） |

### ⚠️ 有风险的（暂时别用，知道有这回事即可）

| 命令 | 风险 |
|---|---|
| `git checkout .` / `git restore .` | **丢弃未提交的改动，不可恢复** |
| `git reset --hard` | **同上，且会丢提交** |
| `git push --force` | **覆盖远程历史，可能毁掉别人的工作** |

**除了这三个，git 里的操作基本都有后悔药。** 连 commit 都能撤销（`git reset --soft HEAD~1` 退回上一个提交、把改动放回暂存区）。

**所以：大胆用 `add`，它只登记不改内容，随时能撤。**

---

## 第 6 章 VS Code 面板对照表

打开源代码管理面板：左侧栏第 3 个图标，或 `Control` + `Shift` + `G`

> 打不开项目？`code` 命令没装时可以用：
> ```bash
> open -a "Visual Studio Code" ~/git-practice
> ```

### 面板 ↔ 命令 对照

| 面板上的东西 | 等于命令 |
|---|---|
| `Changes` 分区 | `git status` 里第 2 列有标记的 |
| `Staged Changes` 分区 | `git status` 里第 1 列有标记的（暂存区） |
| 悬停文件 → 点 **`+`** | `git add 文件` |
| 悬停文件 → 点 **`−`** | `git restore --staged 文件` |
| **点文件名** | `git diff`，打开左右分栏对比视图 |
| Message 框 + **Commit** 按钮 | `git commit -m "..."` |
| **Sync Changes** 按钮 | `git pull` + `git push`（**先拉后推**） |
| 底部 `main ⟳ 0↓ 1↑` | 0 个要下载，1 个要上传 |

### 状态字母对照

| VS Code | 命令行 | 含义 |
|---|---|---|
| `U` | `??` | Untracked，未跟踪 |
| `M` | `M` | Modified，已修改 |
| `A` | `A` | Added，已暂存的新文件 |
| `D` | `D` | Deleted，已删除 |

### Graph 面板怎么读

```
∨ Graph
  🔄 Outgoing Changes   main          ← 待推送的提交
  ○ docs: add second line  【main】        ← 本地分支指针（空心圈=未推送）
  ● docs: add readme  【origin/main】      ← 远程分支位置（实心点=已推送）
```

**两个标签的相对位置 = ahead/behind 的可视化：**

- `main` 在 `origin/main` **上方** → 你 ahead，需要 push
- 两个标签在**同一行** → 完全同步
- `origin/main` 在上方 → 你 behind，需要 pull

### ⚠️ Commit 按钮的两个坑

**坑 1：提交信息为空时，按钮是禁用的。**
点了没反应不是 bug。Message 框里的灰字是**占位提示**，不是内容。要点进框里真的打字，按钮才会变亮。

**坑 2：暂存区为空时点 Commit，会弹窗问"要不要把全部改动暂存并直接提交？"**
点 Yes 等于 `git add . && git commit` —— 把所有改动一股脑打包。**在有几十个待提交文件的真项目里，这是灾难。看到这个弹窗就停下来想一想。**

### 建议的使用方式

| 动作 | 用什么 | 为什么 |
|---|---|---|
| **看**（status / diff / 历史） | 面板 | 分栏对比、提交图比终端直观得多 |
| **写**（add / commit / push） | 命令行 | 每一步显式、可预期、能复制、能写进脚本 |

等你熟练到"闭着眼知道自己在哪个区"，再全用面板也无妨。

**最佳学习姿势：命令行敲命令，同时开着面板看变化。** 你 `git add` 一个文件，就看着它从 `Changes` 跳到 `Staged Changes`；你 `git commit`，就看着新圆点出现在图顶端。两边互相印证，建立直觉最快。

---

## 第 7 章 常见困惑 Q&A

> 以下都是初学者实际问出来的问题。

### Q1：这些命令我在哪敲？

Terminal.app、VS Code 内置终端，或 Claude Code 输入框（首字符打 `!`）。详见 [1.1 节](#11-命令在哪敲)。

### Q2：`~/git-practice` 里的文件在哪里？我没看到

`~` = 你的主目录（`/Users/你的用户名/`）。**它就是一个普通文件夹**，跟"桌面""下载"并列。

跑 `open ~/git-practice` 就能在 Finder 里看到它。**把抽象变具体，是理解 git 最快的办法。**

### Q3：暂存区、本地仓库都是这个文件夹吗？

**都在同一个项目文件夹里，但位置不同：**

- 工作区 = 你能看到的文件
- 暂存区 = `.git/index`（**就是一个约 137 字节的文件**）
- 本地仓库 = `.git/objects/`

跑 `ls -la ~/git-practice/.git` 亲眼看。做完 commit 后对比时间戳，能看到 `index`、`objects/`、`COMMIT_EDITMSG` 都是你 commit 那一刻更新的。

**"暂存区"这个听起来很玄的东西，物理上就是 137 字节的一个文件。**

### Q4：我点了 `+` 能撤销吗？

**能，完全安全。** 点 `−` 就撤销了，命令行是 `git restore --staged 文件`。

**更重要的是：`git add` 完全不改动文件内容。** 它只是在清单上打勾。你的文件在硬盘上一个字节都没变。反复点来点去、add 完直接关机 —— 都没事。

甚至 **commit 也能撤销**（`git reset --soft HEAD~1`）。

**git 里只有三个操作真正有风险**：`git checkout .` / `git restore .`、`git reset --hard`、`git push --force`。其他都有后悔药。

### Q5：挑选（暂存）到底有什么用？

假设你手头有 35 个改动文件，它们其实是 5 件不相干的事：

| 事 | 文件 |
|---|---|
| A 改配色 | `config.py`、`visualizer.py` |
| B 写文档 | `docs/*.tex` |
| C 生成的数据文件 | `data/*.csv` |
| D 笔记 | `notes/**` |
| E 垃圾，不该进 git | `*.bak`、编译产物 |

**不挑选，全塞进一个 commit：**

```
a1b2c3d 更新了一些东西        ← 35 个文件全在里面
```

半年后想问"配色是什么时候改亮的、改之前长什么样"？你得在这个巨型改动里翻。想单独回退配色而保留文档？做不到，它们绑死了。

**挑选之后：**

```
d4f2a1  style: 调亮可视化器配色           ← A，2 个文件
9b3e77  docs: 补充坐标系设计文档           ← B，4 个文件
1c8a02  feat: 加入布线产出的板子文件        ← C，6 个文件
5f9d31  chore: 同步笔记                   ← D
                                        ← E 根本不提交
```

每个 commit 都能一句话说清，能单独回退、单独查看、单独对比。

**这才是 git 相比"网盘备份"的核心价值 —— 它不是存文件，是存"有意义的变更"。**

### Q6：为什么一定要写提交信息？

**因为 commit 信息是你唯一能搜索历史的入口。**

假设仓库有 1670 个提交，你想知道"配色是什么时候改亮的"：

- 信息写得好 → `git log --grep="color"` 一秒找到
- 全写"更新" → 只能一个个 `git show` 看 diff，1670 次

**存了但找不到，等于没存。**

好的提交信息长这样（来自一个真实开源项目）：

```
#405: symmetric baseline subtraction in kicad_drc_compare (drop pre-existing items)
#338: oracle-reconnect honors the board edge rule (CLI + GUI); register the edge reader
```

信息量：**issue 编号 + 改了哪个模块 + 具体什么行为变了 + 影响范围**。半年后有人报 bug，`git log --grep="drc_compare"` 就能定位。

**本质：提交信息不是给 git 看的（git 只需要哈希），是给未来的你和同事看的。强制你写，是为了逼你为自己留线索。**

**💡 写不出好信息时，说明你的 commit 粒度太大了。** 如果一个 commit 干了 5 件事，你根本没法用一句话描述它。**提交粒度和信息质量是同一个问题。**

### Q7：`git add` 之后 `git diff` 怎么空了？文件被吞了？

没有。**`git diff` 默认对比「工作区 ↔ 暂存区」**，你刚 add 完，这两处内容一样，所以没差异。

要看「暂存区 ↔ 最新提交」用 **`git diff --staged`**。

### Q8：我怎么看暂存区和最新提交的区别？

```bash
git diff --staged
```

**它的输出 = 你这次 commit 会包含的全部内容。** 提交前的最后一道检查。

面板上的等价物：**`Staged Changes` 分区里的东西**，点文件名打开对比视图。

### Q9：远程仓库一定是 GitHub 吗？

**不是。** git 跟 GitHub 没有必然关系，remote 只是一个地址，可以是 GitLab、Gitee、公司服务器、**甚至你自己硬盘上的另一个文件夹或移动硬盘**。

详见 [第 0 章 git ≠ GitHub](#git--github)。

**实际意义**：哪天要把代码交给不用 GitHub 的客户，或公司要求代码不出内网，换个 remote 地址就行，前面学的一切照常用。**不会被 GitHub 绑死。**

### Q10：upstream 和 fork 是什么关系？

**fork = 在 GitHub 上，把别人的仓库完整复制一份到你的账号下。** 网页点一下就完成，全在 GitHub 服务器上，跟你电脑无关。

**为什么要 fork？** 因为你没有别人仓库的写权限，不能往人家仓库推东西。复制一份到自己账号下，你就是那份的主人。

```
        GitHub 上

  原作者/原项目              ← 上游，习惯命名为 upstream
       │                      作者在持续更新
       │ 你 fork 了一份
       ↓
  你的账号/你的副本           ← 习惯命名为 origin
       ↕                      你可以自由推送
       │
    你的电脑
  本地仓库
```

**`upstream` 和 `origin` 都只是习惯叫的名字，不是 git 关键字**，跟变量名一样。

| | origin（你的 fork） | upstream（原作者） |
|---|---|---|
| 拉取 `fetch`/`pull` | ✅ | ✅ 拉作者的新功能 |
| 推送 `push` | ✅ 唯一能推的地方 | ❌ 没权限 |

**典型工作流：**

```bash
git fetch upstream              # 看看原作者更新了啥
git log HEAD..upstream/main     # 列出他有而你没有的提交
git merge upstream/main         # 决定要的话，合并过来
git push origin main            # 再推到自己的 fork
```

**防呆技巧**：可以把 upstream 的 push 地址故意设成一个假地址（如 `DISABLED-read-only`），这样手滑敲 `git push upstream` 会直接失败，而不是真的尝试推送。

**想把改动贡献回原作者**：在 GitHub 网页上发 **Pull Request（PR）**，请求作者合并你的提交。作者可以同意或拒绝。这是开源协作的基本方式。

### Q11：`git remote -v` 输出不空会怎样？

说明本地仓库已经连着某个远程了。三种常见情况：

1. **你是 `git clone` 下来的** —— clone 自动把源地址设为 `origin`。直接 `git push` 即可，不需要 `gh repo create`。
2. **已经建过一次** —— 再跑 `gh repo create` 会报错"remote origin 已存在"。这是保护机制。
3. **有多个远程** —— 一个本地仓库可以连多个远程，各有各的名字（见 Q10）。

**`git remote -v` 是你"push 之前先确认要传去哪"的检查手段。** 传错地方（比如把私有代码推到公开仓库）不容易补救，养成先看一眼的习惯。

### Q12：Sync Changes 和 push 有什么区别？

**Sync = `git pull` + `git push`（先拉后推）。**

| | 做什么 |
|---|---|
| `git push` | 只上传 |
| `git pull` | 只下载 |
| **Sync** | **先下载，再上传** |

**为什么要先拉再推？** 如果别人在你之前推了东西，你直接 push 会被拒绝（git 不允许覆盖别人的提交）。先 pull 合并，再 push 就顺了。

**单人仓库时两者效果一样。** 但多人协作时 pull 可能触发**合并冲突** —— 那时按钮点下去不是"传上去"这么简单。**建议初期用命令行 `git push`，你清楚知道自己只做了一件事。**

### Q13：点 Commit 按钮没反应

**Message 框是空的。** 框里的灰字是占位提示，不是内容。git 强制要求提交信息非空，所以 VS Code 把按钮禁用了。

点进框里真的打字，按钮就会变亮。

**打不进字？** 依次试：点一下窗口让它获得焦点 → 确认光标在框内闪 → 切一下输入法（`Control`+`空格`）→ 把窗口/面板拉宽 → 重启 VS Code。

**别卡在这儿，用命令行 `git commit -m "..."` 一秒搞定，效果完全一样。**

### Q14：看到 `529 Overloaded · Retrying · attempt 6/10` 是什么？

**服务器过载的自动重试提示，不是你的操作出错。**

`529` 是 HTTP 状态码，特指"服务器忙不过来"，跟你的网络、账号、余额无关。工具会自动重试，你什么都不用做。

同类的：`git push` 偶尔也会因为 GitHub 抽风失败，重试一次通常就好。**看到 "retry" 字样，先等它自己重试完。**

---

## 第 8 章 踩过的坑

### 8.1 命令被截断

复制多行命令时，末尾部分可能没带上。比如：

```bash
echo "这是第二行" >> README.md && git status -s
```

只跑了 `echo "这是第二行"` → 文字打在屏幕上，**文件根本没被修改**。

**判断方法**：`echo "文字" > 文件` 成功时**没有输出**（内容进了文件）。如果文字显示在屏幕上，说明重定向部分丢了。

**对策**：不确定时把长命令**拆成两条**分别敲。

### 8.2 每次执行 cwd 会重置

在 Claude Code 的 `!` 模式（以及某些执行环境）下，**每条命令都是全新的 shell**，`cd` 不会保留。所以每条都要自己带上：

```bash
cd ~/git-practice && git status
```

会看到提示 `Shell cwd was reset to ...` —— 这是正常提醒，不是报错。

### 8.3 `!` 必须是第一个字符

在 Claude Code 输入框里，`!` 前面**不能有换行或空格**，否则不会执行，只会当普通文字发送。从代码块复制时注意别把前面的换行带上。

### 8.4 `code` 命令没装

```
zsh: command not found: code
```

**替代方案：**

```bash
open -a "Visual Studio Code" ~/git-practice
```

**永久解决**：VS Code 里按 `Command+Shift+P` → 输入 `shell command` → 选 **`Shell Command: Install 'code' command in PATH`** → 重开终端。

### 8.5 `>` 打成 `>>`（或反过来）

| 符号 | 作用 | 手滑后果 |
|---|---|---|
| `>` | 覆盖 | **原内容全没了** |
| `>>` | 追加 | 多了不想要的一行（好补救） |

写文件时留个心。

### 8.6 GitLens 装不装？

GitLens（"Supercharge Git within VS Code"）是个流行的 VS Code 扩展。

**建议：初学阶段可以不装；已经装了就只用它"看"。**

| 值得用（读） | 建议避开（写） |
|---|---|
| 行内 blame（每行代码旁显示谁改的、何时、哪个 commit） | 一键 commit / push / stash / rebase 按钮 |
| 提交图，比 `git log` 直观 | |
| 点文件看分栏 diff | |

**理由**：它**不教你 git**，是把 git 状态可视化。你还没有心智模型的时候，那些面板只是一堆看不懂的信息。它的按钮式操作会**隐藏**你正在学的三段式流程，容易变成"会点按钮但不懂发生了什么"。

**它最强的行内 blame，在你有 100+ 提交、要追查"这行为什么这么写"时才真正有价值。**

VS Code **自带**的源代码管理面板已经够用于看改动和 diff 了。

---

## 第 9 章 下一步学什么

### 你现在会的

**命令**：`init` `status` `add` `commit` `push` `log` `diff` `diff --staged` `remote -v`

**概念**：四区域模型、暂存区的挑选作用、ahead/behind、追踪关系、origin/upstream、fork、`.git` 各部分的物理位置、面板与命令的对应、git ≠ GitHub

**这些够日常单人使用了。**

### 建议的后续顺序

**① `.gitignore`（优先级最高）**

让 git 永久忽略某些文件。在项目根目录建一个 `.gitignore` 文件：

```gitignore
# 虚拟环境 / 依赖（能重新安装，不该进版本库）
venv/
node_modules/
__pycache__/
*.pyc

# 编译产物（能重新生成）
*.so
*.o
build/
dist/

# 系统垃圾
.DS_Store
Thumbs.db

# 手动备份（有了 git 就不需要这种了）
*.bak
```

**⚠️ 必须在 `git init` 之前或第一次 `git add` 之前写好。** 一旦把几百 MB 的虚拟环境提交进去，清理需要重写历史（`git filter-repo`），很麻烦。

**判断标准**：这个文件能否从别的东西重新生成？
- 能 → **不进** git（编译产物、venv、缓存、下载的依赖）
- 不能 → **进** git（源码、文档、你手工做的设计文件）

**② 练"分批提交"**

在练习仓库里故意造 3 个文件（2 个属于"文档"，1 个属于"代码"），然后：

```bash
git add 文档1 文档2          # 只暂存文档
git status -s                # 观察：2 个 A，1 个 ??
git diff --staged            # 确认只有文档的内容
git commit -m "docs: ..."    # 提交第一件事

git add 代码                  # 再暂存代码
git commit -m "feat: ..."    # 提交第二件事

git log --oneline            # 两条清晰的历史
```

**一次改动，两个 commit，各自说清了自己在干什么。** 这是处理真实项目的必备技能。

**③ 给真实项目建库**

先写 `.gitignore`，再 `git init`，**然后在 `git add` 之前用 `git status` 逐个确认**。

**④ 下载相关**

```bash
git clone 网址        # 下载整个仓库（含全部历史）
git pull             # 拉取远程更新
git fetch            # 只下载不合并（更安全，可以先看再决定）
```

**⑤ 更远的（用到再学）**

- **分支** `git branch` / `git checkout -b` —— 开个平行世界改东西，不影响 main
- **合并冲突** —— 两个人改了同一行时怎么办。得真遇到几次才会
- **撤销操作** `git revert` / `git reset` —— 后悔药的正确用法
- **PR 协作流程** —— 多人项目的标准工作方式

---

## 附：教学者备忘

如果你要用这份笔记教别人，几条经验：

1. **一次只给一条命令，敲完等对方反馈再往下。** 一口气给 4-5 条，初学者会迷失在"我现在在第几步"。

2. **先教只读命令（status/log/diff），建立信心再教写操作。** 学 git 最大的障碍是怕弄坏。

3. **把抽象变具体。** 学生问"文件在哪"时，直接 `open` 给他看 Finder；问"暂存区是什么"时，`ls -la .git` 让他看到 `index` 这个文件和它的时间戳。**概念落地了才记得住。**

4. **命令行 + 图形面板配合。** 命令行负责"做"，面板负责"看懂做了什么"。两边互相印证，建立直觉最快。

5. **用真实仓库举例，但在练习仓库操作。** 拿学生自己项目的 `git status` 讲解，代入感强；但动手练习一定在一次性仓库里，别在有几十个未提交改动的真项目上练。

6. **预告输出。** 敲命令前先说"你会看到 XXX"，让学生带着预期去对照，比事后解释效果好得多。

---

*本笔记整理自一次真实的一对一教学过程，所有 Q&A 都来自学习者实际提出的问题。*
