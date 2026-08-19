---
name: blog-project
description: MyBlog 项目（Vue3 博客）开发规范——在 Blog 子目录中用 pnpm 操作、修改前端文件后运行 ox（oxlint，不可用时回退 eslint）、保持组件/CSS 样式分离与代码审查通过。当任务涉及修改 Blog 前端源码、运行 lint/格式检查、构建或处理 markdown 渲染时加载本 skill。
whenToUse: 需要修改 MyBlog 的 Blog 前端源码、运行包管理/审查/构建命令，或处理 markdown 告示（[!NOTE]/[!WARNING]/[!IMPORTANT]）渲染时
---

# MyBlog 项目开发规范（DSH Skill）

本仓库是 GitHub Pages 部署仓库，前端源码位于 **`Blog/` 子目录**。所有开发、审查、构建命令都必须在 `Blog/` 目录下执行。

## 1. 工作目录（必须正确导航）

- 仓库根（skill 发现根）：`/home/nau/code/MyBlog` —— 存放已构建的站点文件（assets/、content/、config/、index.html 等）与 `Blog/` 源码。
- **实际开发工作目录**：`/home/nau/code/MyBlog/Blog` —— 这是唯一的 package.json 所在目录，所有 `pnpm` 命令都在这里运行。
- 构建输出：`pnpm build` 输出到仓库根的 `tager/` 目录（vite 的 `outDir: ../tager`），再经 `.github/workflows/deploy.yml` 部署到 GitHub Pages。

```bash
# 所有命令先进入工作目录
cd /home/nau/code/MyBlog/Blog
```

> 注意：仓库根目录的 `.gitignore` 忽略了 `Blog`，因此根目录的 git 操作不包含 Blog 源码；不要为了 oxlint 能发现文件而在 Blog 下新建嵌套 `.git` 仓库。

## 2. 包管理：只用 pnpm

- 包管理器固定为 **pnpm**（package.json 已声明 `"packageManager": "pnpm@11.22.0"`），**禁止使用 npm**（npm 的 package-lock.json 已删除，仅保留 pnpm-lock.yaml）。
- `pnpm-workspace.yaml` 已配置 `allowBuilds: esbuild: true`；若 pnpm 报 `ERR_PNPM_IGNORED_BUILDS`，运行 `pnpm approve-builds <pkg>` 而非改用 npm。

```bash
pnpm install          # 安装依赖（勿用 npm install）
pnpm dev              # 启动开发服务器（自动先执行 predev 生成内容清单）
pnpm build            # 构建（自动先执行 prebuild：生成清单 + 清空 ../tager）
pnpm preview          # 预览构建产物
```

> 已知无害提示：`pnpm install` 时 `prepare` 脚本（husky）会打印 `.git can't be found`——因为 Blog 是仓库根的子目录、没有自己的 `.git`。这是预期现象，退出码仍为 0，**不要**为修复它而在 Blog 下新建 `.git`。

## 3. 每次修改前端文件后：运行 ox（不可用则回退 eslint）

修改 `Blog/src/` 下的任意前端文件（`.vue` / `.ts` / `.css` 等）**完成后**，必须立即运行代码检查：

```bash
pnpm run ox          # oxlint（优先）
pnpm run lint        # eslint（兜底，必须通过：0 errors）
pnpm run fmt:check   # prettier 格式检查（必须通过）
```

- `pnpm run ox` 即 `oxlint --no-error-on-unmatched-pattern`。当前仓库布局下，根目录 `.gitignore` 的 `Blog` 条目会让 oxlint 扫不到文件（输出 0 files、退出码 0）——这是预期行为，**此时以 `pnpm run lint`（eslint）为准**，不要试图通过新建嵌套 git 仓库等方式强行修复。
- 若改了代码后 `pnpm run lint` 出现 error（非 warning），必须修复后再继续；warning 可保留。
- 若需自动修复：`pnpm run ox:fix`（oxlint --fix）、`pnpm run lint:fix`（eslint --fix）、`pnpm run fmt`（prettier --write）。

## 4. 代码审查流程（修改完成后对照）

```bash
cd /home/nau/code/MyBlog/Blog
pnpm run lint        # eslint 代码规范（必须 0 errors）
pnpm run fmt:check   # prettier 格式（必须通过）
pnpm run ox          # oxlint（可用时运行）
```

审查清单：
- [ ] Vue 组件与 CSS 样式已分离（组件同名 `.css` 文件，见 `.trae/rules/line.md`）
- [ ] CSS 文件含文件头注释与分节注释
- [ ] 使用 CSS 变量而非硬编码颜色
- [ ] `pnpm run lint` 通过（0 errors）
- [ ] `pnpm run fmt:check` 通过
- [ ] `pnpm run ox` 运行过（0 文件属预期，可忽略）
- [ ] 重启 `pnpm dev` 无报错

（可选）若环境装有 sourcery，可在 Blog 目录运行 `sourcery review .`；未安装时跳过，不影响审查。

## 5. 项目结构与约定

```
Blog/
├── src/
│   ├── components/common/    # 通用组件（每组件一个目录：Name.vue + Name.css）
│   ├── components/windows/   # 窗口组件（ArticleReader、PostsWindow、NoticeWindow 等）
│   ├── composables/          # useXxx.ts 组合式函数
│   ├── styles/               # 全局样式（code-highlight.css 含 markdown 样式）
│   ├── config/ stores/ types/ utils/ router/ views/
├── public/                   # 静态资源（config/、content/、icons/）
├── content/                  # 文章源内容（markdown）
├── scripts/generate-content-manifest.cjs
├── package.json  pnpm-lock.yaml  pnpm-workspace.yaml
├── eslint.config.js          # ESLint 9+ flat config（browserGlobals 含常见浏览器全局）
├── .oxlintrc.json            # oxlint 配置（当前布局下扫不到文件属正常）
└── .prettierrc / .prettierignore
```

关键约定：
- **样式分离**：Vue 组件内不写 `<style>`，样式放同名 `.css` 文件；全局样式放 `src/styles/`。
- **Markdown 渲染**：统一走 `src/composables/useMarkdown.ts` 的 `renderMarkdown()`，所有组件（ArticleReader、PostDetail、PostDetailWindow、AboutWindow、NoticeWindow、PostCard 摘要）都经它渲染。
- **告示语法**：`[!NOTE]` / `[!WARNING]` / `[!IMPORTANT]` 引用块由 `useMarkdown.ts` 的 `processAdmonitions()` 转换为 `<blockquote class="quote-*"><div class="quote-title">…</div><div class="quote-body">…</div></blockquote>`；结构样式在 `src/styles/code-highlight.css`（`.quote-body` 统一收拢多段落/列表/代码，`white-space: pre-line` 保留换行）。ArticleReader/PostDetail 的 scoped CSS 只微调边框色，**不要再给告示加 blockquote 背景或逐段盒子样式**，否则会重现分行错乱。
- **内容文件**：文章在 `Blog/content/zh-CN/posts/<年>/<slug>.md`（构建时也同步到 `public/content/`），改文章后需重新生成 manifest（`pnpm dev`/`pnpm build` 的 pre 脚本会自动做）。

## 6. 工具使用要点

- 查看文件用 read / grep / glob（不要用 shell 的 cat/find/grep 代替）。
- 修改文件用 edit / write；改动后按第 3 节运行检查。
- 运行命令一律 `cd /home/nau/code/MyBlog/Blog && pnpm …`，避免在仓库根目录误执行 npm。
- 涉及构建产物（`../tager`、仓库根 assets/）的改动只应来自 `pnpm build`，不要手工编辑。
