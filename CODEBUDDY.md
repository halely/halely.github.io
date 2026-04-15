# CODEBUDDY.md This file provides guidance to CodeBuddy when working with code in this repository.

## 常用命令

- **安装依赖**: `npm install`  
  使用根目录的 `package.json` 安装 Hexo、部署插件和主题依赖。仓库里有 `pnpm-lock.yaml`，但 GitHub Actions 当前使用的是 `npm install`，为了和线上构建保持一致，优先使用 npm。

- **本地预览**: `npm run dev`  
  启动 `hexo server`，用于本地预览博客。它会读取根目录 `_config.yml`、`source/` 内容和当前主题 `hexo-theme-butterfly-dev`，适合边改文章边看效果。

- **本地预览（同上）**: `npm run server`  
  与 `npm run dev` 等价，都是 `hexo server`。如果脚本说明里写的是 `server`，直接用这个命令即可。

- **静态构建**: `npm run build`  
  执行 `hexo generate`，把站点生成到 `public/`。提交前如果只想验证“能不能正常生成站点”，这是最直接的检查命令。

- **清理生成产物**: `npm run clean`  
  执行 `hexo clean`，清理 Hexo 缓存和生成结果。遇到模板、资源或配置改完后预览异常时，通常先跑一次再重新 `build` 或 `dev`。

- **Hexo 部署**: `npm run deploy`  
  执行 `hexo deploy`，使用根 `_config.yml` 里的 git deploy 配置，把生成后的站点部署到配置的 GitHub Pages 分支。

- **创建新文章**: `npx hexo new post "文章标题"`  
  按 `scaffolds/post.md` 模板创建文章，默认写入 `source/_posts/`。这个仓库的文章 front matter 默认包含 `cover`、`tags`、`categories`。

- **创建独立页面**: `npx hexo new page "about"`  
  按 `scaffolds/page.md` 模板创建页面。适合新增类似 `about`、`movies` 这样的目录页或说明页。

- **运行全部测试**: **未配置**  
  根 `package.json` 没有测试脚本，仓库里也没有测试框架依赖。不要假设存在 Jest、Vitest 或 Playwright。

- **运行单个测试**: **未配置**  
  当前仓库没有单测体系，因此不存在“单个测试命令”。如果后续要补测试，需要先引入框架和脚本。

- **Lint**: **未配置**  
  根 `package.json` 没有 `lint` 脚本，也没有 ESLint / Stylelint / Prettier 之类的依赖。不要把 lint 当成现成流程依赖。

## 架构概览

这是一个 **Hexo 5 博客仓库**，根目录就是站点工程本体，而不是“博客源码 + 外部主题包”的轻量壳子。根 `_config.yml` 选择主题为 `hexo-theme-butterfly-dev`，同时定义站点 URL、permalink、分页、git 部署和一些插件配置；根 `package.json` 只保留了最核心的 Hexo 命令：本地预览、构建、清理和部署。

### 1. 三层结构：站点壳、内容层、主题源码层

- **站点壳** 在仓库根目录，由 `_config.yml` 和 `package.json` 驱动，决定站点级 URL、生成目录、部署方式和所启用的主题。
- **内容层** 在 `source/`，包括文章、独立页面、数据文件和统一静态资源。
- **主题源码层** 在 `themes/hexo-theme-butterfly-dev/`，其中不仅有模板和样式，还有构建期的 Hexo 扩展脚本；也就是说，这个主题是仓库的一部分，改主题行为时通常直接改这里。

这三个层次协同工作：根配置决定站点怎么生成，`source/` 决定“有什么内容”，主题源码决定“这些内容最终怎么被渲染、增强和交互化”。

### 2. 内容层不是只有文章，页面行为大量依赖 front matter

`source/_posts/` 是普通文章目录；文章模板来自 `scaffolds/post.md`，默认 front matter 带有 `cover`、`tags`、`categories`。`source/` 顶层还有若干功能型页面，如 `tags`、`categories`、`link`、`music`、`Gallery`。这些页面不是靠目录名硬编码生效，而是主要依赖 front matter 里的 `type` 或 `layout`，再由主题模板分发到对应页面实现。

这个仓库使用 **统一资源目录**，因为根配置里 `post_asset_folder: false`。因此文章和页面图片主要集中在 `source/img/`，而不是每篇文章单独配一个同名资源目录。处理图片路径问题时，优先按“统一静态资源目录”来理解。

此外，`source/_data/link.yml` 是友链页的数据源，不是普通内容文章。凡是看到页面效果来自结构化数据，而不是 Markdown 正文，优先去 `_data` 目录排查。

### 3. 主题源码层由四部分组成：模板、样式、浏览器脚本、Hexo 扩展脚本

`themes/hexo-theme-butterfly-dev/` 的关键部分如下：

- `layout/`: Pug 模板，决定页面骨架与模块拼装。
- `source/css/`: Stylus 样式，入口是 `source/css/index.styl`。
- `source/js/`: 浏览器端脚本，核心是 `utils.js` 和 `main.js`。
- `scripts/`: Hexo 构建期扩展，注册 helper、tag 和 filter。

理解这个仓库时最重要的一点是：**很多“页面功能”并不是只在模板里实现，而是模板 + 构建期扩展 + 浏览器脚本共同完成。**

### 4. 渲染主链路：配置注入到页面，再由前端脚本接管交互

站点生成时，Hexo 会先读取根配置、内容和主题配置。主题中的 `scripts/events/init.js` 会在 `before_generate` 阶段做两件关键事：一是校验 Hexo 版本至少为 5；二是把 Hexo 的代码高亮配置同步进主题配置，供模板和样式层共用。

真正的“配置到前端”的桥接点是 `layout/includes/head/config.pug`。它会把搜索、简繁切换、代码块工具、Snackbar、lightbox、lazyload 等设置序列化成浏览器端可读的 `GLOBAL_CONFIG`。随后 `layout/includes/additional-js.pug` 再加载 `utils.js`、`main.js` 以及按需启用的第三方脚本。

前端交互主入口是 `source/js/main.js`。这里不仅处理导航栏、代码块工具栏、图片灯箱、图库布局、目录、评论切换等功能，还会暴露 `window.refreshFn()`，供 PJAX 页面切换后重新初始化页面行为。因此，只改模板但不接入 `main.js`/`refreshFn` 的功能，在启用 PJAX 时往往会“首屏正常、切页失效”。

### 5. 页面模板分工很明确，但页面类型分发藏在少数入口文件里

如果要快速判断一个页面该从哪里改，优先看这些入口：

- `layout/index.pug`: 首页文章流入口。
- `layout/post.pug`: 文章详情页。
- `layout/page.pug`: 独立页面分发器。
- `layout/includes/layout.pug`: 全站 HTML 壳。
- `layout/includes/header/index.pug`: 顶部大图和标题逻辑。

其中 `layout/page.pug` 会按 `page.type` 分发：`tags`、`link`、`categories` 走专用模板，其它页面走默认页面模板。首页文章卡片实际由 `layout/includes/mixins/post-ui.pug` 控制；文章详情页中的分享、版权、奖励、评论和相关文章则主要挂在 `layout/post.pug` 这条链上。

顶部横幅图的来源也不是单点控制：文章页优先用文章 `cover` / `top_img`，普通页面回退到主题 `default_top_img`，首页和归档页则分别使用主题配置里的专用图片。看到“为什么这个页面头图不对”，先看 `layout/includes/header/index.pug`，再看页面 front matter 与主题配置。

### 6. 这个主题把很多能力做成了 Hexo 扩展，而不是纯前端组件

`scripts/` 目录非常关键：

- `scripts/helpers/` 注册模板辅助函数，例如 `page_description`、`urlNoIndex`、归档/分类/标签侧栏生成逻辑等。
- `scripts/tag/` 注册自定义标签，如 `gallery`、`galleryGroup`、`note`、`tabs`。
- `scripts/filters/` 在渲染前后加工内容，例如自动补默认封面、把图片 `src` 改成 lazyload 所需属性。

这意味着一些功能排查要跨三层找：**Markdown 内容有没有正确写标签、构建期 tag/filter 有没有产出正确 HTML、浏览器端脚本和样式有没有把产物增强起来。**

### 7. 两个典型跨文件功能：友链页和图库页

**友链页** 的链路是：`source/link/index.md` 通过 `type: "link"` 进入 `layout/page.pug`，再落到 `layout/includes/page/flink.pug`；真正的数据来自 `source/_data/link.yml`。也就是说，改页面结构去模板，改链接内容去 `_data`。

**图库页** 更典型：`source/Gallery/index.md` 和 `source/Gallery/wallpaper/index.md` 在 Markdown 里使用 `{% gallery %}` / `{% galleryGroup %}`；这些标签由 `scripts/tag/gallery.js` 注册并生成 HTML；外观由 `source/css/_tags/gallery.styl` 控制；最终的 justified gallery 排版和灯箱行为则由 `source/js/main.js` 与 `utils.js` 接手。这个功能不是“改一个文件就完事”的类型。

### 8. 部署与产物：本地 Hexo 部署和 GitHub Actions 并存

`npm run build` 会把站点生成到 `public/`。`.gitignore` 已忽略 `public/`、`db.json`、`node_modules/` 等产物或缓存文件，所以这些通常不应提交。

仓库现在有两条部署路径：

- 根 `_config.yml` 中配置了 `hexo-deployer-git`，可通过 `npm run deploy` 直接推送生成站点。
- `.github/workflows/deploy.yml` 会在 push 后执行 `npm install` + `npm run build`，再把 `public/` 部署到 `master`。

因为 CI 明确使用的是 **npm**，即使仓库里有 `pnpm-lock.yaml`，默认也应按 npm 视角理解依赖安装与部署行为，避免“本地能跑、线上命令不一致”的偏差。

### 9. 仓库特有注意点

- 当前主题配置启用了 **PJAX**，所以新增前端行为时要考虑 PJAX 重新初始化。
- 搜索功能当前是关闭的：`algolia_search.enable` 和 `local_search.enable` 都是 `false`。搜索模板或脚本改了但界面没出现，先确认配置是否启用。
- 主题初始化脚本明确把旧的 `butterfly.yml` 视为废弃配置。如果需要做主题覆盖配置，优先使用根目录 `_config.butterfly.yml` 或直接编辑主题 `_config.yml`，不要再引入旧配置文件名。
- 主题菜单里配置了 `/movies/` 和 `/about/`，但当前 `source/` 下没有对应目录。遇到导航报错或 404，先判断是“菜单配置领先于内容创建”，不要默认是模板问题。
