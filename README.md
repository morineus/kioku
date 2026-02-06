# Kioku

> @[mulberyn](https://github.com/mulberyn) 是一个前端萌新，这是他一边学一边尝试开发的项目。

## 技术栈

- React + NextJS + TypeScript

## 特性

- 支持 i18n 国际化
- 技术文章/生活文章双布局

## URL 设计

- `/` 首页（英文首页 `/en`）
- `/[lang]/post/[type]`，文章列表页，`type` 可选，值为 `tech` 或 `life`（列表页可采取不同展示风格，`lang` 默认为 `zh`，英文版 `/en/post/[type]`）
- `/[lang]/post/[type]/[slug]`，文章详情页
