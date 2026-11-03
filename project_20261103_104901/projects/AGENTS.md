# AGENTS.md

## 项目概览
红楼梦·MBTI 人格测试 —— 小红书虚拟商品，**单文件纯静态网页**（native-static 模板）。
全部功能（验证码解锁、28 题答题、计分匹配、结果展示、海报生成）都在根目录 `index.html` 一个文件内，无后端请求，本地双击即可运行。

## 技术栈
- 原生 HTML + CSS + JS，仅两个外部 CDN：Tailwind CSS v3（cdn.tailwindcss.com）、html2canvas@1.4.1（jsdelivr）
- 字体 CDN：Google Fonts（Ma Shan Zheng 书法体 / Noto Sans SC 正文），CN 环境用 fonts.googleapis.cn
- 9 张 Q 版人物插画以 webp base64 内嵌在 `CONFIG.avatars`（约 210KB）：8 张单人像 + 1 张 8 人合照 `group`（引导页主视觉，容器为 w-72 h-52 圆角卡片），保证单文件自包含且海报 canvas 不被跨域污染

## 构建与运行
- 开发预览：`npx serve -l ${DEPLOY_RUN_PORT}`（.coze 已配置 dev.run，主仓端口 5000）
- 无构建步骤，改完 index.html 刷新即可
- 验证：`node --check` 检查 JS；用 test_run 对 localhost:5000 做冒烟测试

## 代码结构（index.html）
| 区段 | 位置 | 说明 |
|------|------|------|
| Design Token | `<script>` tailwind.config | 朱砂红 primary #C0392B、石青蓝 secondary #3E6B73、月白 background #FAF5EE |
| 页面 DOM | body 内 5 个 `<section class="screen">` | unlock / intro / quiz / loading / result，JS 控制切换 |
| CONFIG 配置区 | 主 script 开头 | **卖家可改**：codesMD5（访问码MD5列表）、questions（28题）、results（16型解析）、avatars（插画base64） |
| MD5 实现 | CONFIG 上方 | 自包含 md5()，与 python hashlib 结果一致 |
| 应用逻辑 | CONFIG 之后 | 解锁校验（3次错锁10分钟/localStorage 7天）、答题状态机、计分（票数比较法）、人物匹配（字母重合度+并列判别 J/P→S/N→T/F→E/I）、海报渲染（html2canvas 750×1000 scale 2）、复制文案 |
| 隐藏海报 DOM | #poster（left:-9999px） | 3:4 小红书尺寸，截图节点 |

## 关键规则
- 题目结构：每轴 7 题（EI/SN/TF/JP 共 28 题），A 选项计左端字母（E/S/T/J），B 计右端（I/N/F/P），奇数题数保证无平局
- 人物池 8 位：林黛玉INFP、贾宝玉ENFP、薛宝钗INFJ、王熙凤ESTJ、贾探春ENTJ、晴雯ISFP、刘姥姥ENTP；贾宝玉/史湘云共用 ENFP（结果页显示双重镜像）
- 16 型中 8 型为映射结果（exact=false），匹配算法与 results.charType 已全量自洽验证
- localStorage 键：`hlm_unlock_exp`（解锁7天）、`hlm_lock_until`（锁定10分钟）、`hlm_fail_cnt`（连错计数）
- 横屏由纯 CSS `#landscape-mask` 遮罩提示「请竖屏使用哦」

## 设计规范
见 `DESIGN.md`（新中式国风：朱砂红+石青蓝+月白，Ma Shan Zheng 仅用于标题/印章，正文 Noto Sans SC，卡片软阴影，按钮朱砂色波纹，页面切换淡入上浮）。
原型稿：`.cozeproj/prototype/mobile/`（unlock/intro/quiz/result 4 页）。
