# WeRead KOReader Plugin

> **免责声明**：本项目仅供个人学习和技术研究使用，不得用于商业用途。使用本项目所产生的一切后果（包括但不限于账号封禁、数据丢失等）由使用者自行承担，项目作者概不负责。请遵守微信读书的用户协议和相关法律法规。

在 KOReader 上阅读微信读书书籍和公众号文章、同步阅读时长的插件。

## 功能

**书籍**

- 浏览微信读书书架，搜索书籍
- 下载单章或整本书为 EPUB，直接在 KOReader 中阅读
- 章节内容解码、CSS 样式、图片资源打包
- 自动生成目录（TOC），自动嵌入封面
- 下载并嵌入划线和想法，阅读时可一键显示/隐藏，点击划线查看想法内容

**公众号**

- 浏览已关注的公众号列表
- 下载公众号文章为 HTML（图片内嵌 base64，KOReader 可自由调节字体大小）
- 文章列表本地缓存，无需重复请求

**阅读时间上报**

- 后台自动向微信读书上报阅读时长（默认每 30 秒一次）
- 支持两种目标书籍模式：
  - **自动关联**：打开微信读书缓存书籍时自动上报该书，关闭时自动停止
  - **手动设置**：从书架选择一本固定书籍作为上报对象
- 支持「仅在阅读时上报」或「KOReader 启动即上报」两种触发模式
- 上报状态可在菜单中查看（已上报次数、最近上报时间、错误信息）

**书籍管理**

- 书架支持多种排序方式（最后阅读时间、书名、默认顺序）与筛选（已读完/未读完、已下载/未下载，两组可组合）
- 书籍详情页展示作者、出版社、评分、字数、阅读进度等信息
- EPUB 自动嵌入封面图片
- 缓存管理：查看/清理单本或全部缓存
- 自定义下载目录：可指定书籍/文章的保存位置（默认 `<KOReader 数据目录>/weread/cache`）

## TODO

- [ ] 阅读进度双向同步（KOReader 位置 ↔ 微信读书进度映射）
- [ ] 当前书籍详情页（阅读中展示微信读书元数据）
- [ ] 独立的标注/笔记浏览界面（书签、热门划线聚合查看；阅读时查看划线和想法已支持，见「功能 → 书籍」）

## 贡献 / Contributing

欢迎提交 issue 和 PR。提交前请阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。

Issues and PRs are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting.

Bug 反馈请提供清晰的复现步骤或截图、KOReader 日志、插件版本和 KOReader 版本；PR 请说明解决的问题或新增的特性，并按模板填写测试方式和截图。

For bug reports, include clear reproduction steps or screenshots, KOReader logs, plugin version, and KOReader version. For PRs, describe the problem fixed or feature added, and fill in the testing and screenshot sections in the template.

## 安装

> ⚠️ 请使用**较新版本**的 KOReader，过旧的版本可能导致插件无法加载或启动失败（表现为「工具」菜单下找不到「微信读书」）。已知 `2024.11` 会出问题，`2026.3` 可正常使用；建议升级到最新版。详见 [#14](https://github.com/QiuYukang/weread.koplugin/issues/14)。

将插件目录复制到 KOReader 的 plugins 目录：

```
koreader/plugins/weread.koplugin/
```

重启 KOReader，在菜单中找到：

```
工具 → 微信读书
```

## 登录与认证

插件只支持微信扫码登录，不需要创建或维护配置文件。

扫码前需要先为账号开通微信读书 Skill：

1. 手机打开**微信读书 App**。
2. 进入 **我 → 设置 → 微信读书 Skill**。
3. 点击 **获取 API Key**，确认已经生成个人官方 API Key。
4. 在 KOReader 打开 **工具 → 微信读书 → 微信扫码登录**。
5. 使用微信扫码并在手机端确认；若手机显示四位验证码，请在 KOReader 中输入。

插件会验证 Cookie、用户资料和个人 API Key，全部成功后才一次性保存到 KOReader 的 `settings/weread.lua`。如果登录接口没有返回 API Key，本次登录会失败且不会保存任何凭证；请先按上述步骤开通微信读书 Skill，再重新扫码。

二维码以居中弹窗显示；点击二维码弹窗或按设备按键可主动取消本次登录。登录成功后，一级菜单会从“微信扫码登录”变为“已经登录 · 账号名”，只有清除账号数据后才恢复扫码入口。

点击需要 Cookie 或 API Key 的功能时，如果尚未登录，插件会直接引导进入扫码登录。

### 从旧版本升级

认证数据带有独立的 schema 版本。首次启动本版本时，如果现有 `settings/weread.lua` 没有认证版本号，或版本号低于当前版本，插件会自动清除旧 Cookie（包括 refresh token）、API Key、公众号票据和旧账号信息，并要求重新扫码。书籍/章节缓存、下载记录、缓存目录以及其他用户偏好不会被清除。

开发者可以使用不保存凭证的复现脚本验证同一协议：

```bash
pip install requests 'qrcode[pil]'
python scripts/verify_qr_login.py --open-browser
```

### 凭证更新规则

- 微信读书 Web 请求统一使用设置中保存的 Cookie；响应的 `Set-Cookie` 会自动合并并持久化。
- `/web/login/renewal` 只有明确返回 `succ=1` 才视为续期成功，并持久化响应中的新 Cookie。
- 实际验证确认：扫码接口和 `/web/login/renewal` 都不返回 `x-wr-ticket` 或 `x-wrpa-0`，因此扫码登录无法补齐这两个公众号专用请求头。
- 公众号请求被拒绝时仍会自动续期 Cookie 并重试一次，但不能保证解决依赖公众号专用请求头的错误。
- 完整扫码登录会替换旧 Cookie jar，并清除旧账号的公众号票据，避免跨账号残留。

## 菜单结构

```
微信读书
├── 微信扫码登录 / 已经登录 · 账号名
├── 同步进度           （阅读书籍时显示，开发中）
├── 书籍详情           （阅读微信读书缓存书籍时显示）
├── 显示划线和想法     （阅读书籍时显示，开关）
├── 书架               书架浏览（书籍 + 公众号分类）
├── 搜索               搜索微信读书
├── 阅读时间上报        后台上报阅读时长
│   ├── 启用阅读时间上报
│   ├── 仅在阅读时上报
│   ├── 选择目标书籍
│   │   ├── 自动关联微信读书书籍
│   │   └── 手动设置上报书籍
│   └── 上报状态
├── 设置
│   ├── 缓存管理
│   │   ├── 缓存清理
│   │   └── 缓存目录
│   ├── 进度管理
│   │   ├── 打开时拉取进度（暂不可用）
│   │   └── 关闭时上传进度（暂不可用）
│   ├── 下载内容
│   │   ├── 书籍图片（默认开启）
│   │   ├── 公众号文章图片（默认关闭）
│   │   └── 划线和想法（默认关闭）
│   └── 账号管理
│       ├── 账号状态
│       ├── 立即续期 Cookie
│       └── 清除账号数据
└── 关于
```

## 文件结构

```
weread.koplugin/
├── _meta.lua              插件元数据
├── main.lua               入口、UI、业务逻辑
├── CLAUDE.md              开发规范
├── lib/
│   ├── client.lua          HTTP 客户端
│   ├── content.lua         内容解码、EPUB/HTML 生成
│   ├── cookie.lua          Cookie 解析
│   ├── crypto.lua          SHA-256、MD5
│   ├── download_dialog.lua 下载进度对话框
│   ├── i18n.lua            中文翻译
│   ├── qr_login.lua        扫码登录协议、状态机与凭证保存
│   ├── settings.lua        设置持久化
│   └── weread.lua          微信读书协议工具
├── scripts/
│   ├── fetch_weread_epub.py     EPUB 生成参考脚本
│   ├── verify_qr_login.py       扫码登录协议验证（不保存凭证）
│   └── verify_mp_articles.py    公众号 API 验证脚本
└── docs/
    ├── weread-api-reference.md      API 接口参考
    └── weread-content-research.md   内容解码研究
```
