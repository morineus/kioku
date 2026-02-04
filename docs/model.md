# 数据模型

## 文章

文章要进行多语言（i18n）支持，同时支持两种渲染风格（tech/life）。

### 文章数据总览

「文章」数据模型字段说明：

| 字段       | 说明                                      | 多语言支持         |
| ---------- | ----------------------------------------- | ------------------ |
| id         | 文章唯一标识符                            | 否                 |
| slug       | 文章 URL 别名                             | 否                 |
| title      | 文章标题                                  | 是                 |
| renderType | 渲染方式（tech/life）                     | 否                 |
| category   | 文章分类                                  | 是                 |
| series     | 文章系列/专栏（与分类不同的文章收录方式） | 是                 |
| tags       | 文章标签列表                              | 是                 |
| createdAt  | 创建时间                                  | 否                 |
| updatedAt  | 更新时间                                  | 否                 |
| viewCount  | 文章浏览次数                              | 否                 |
| coverImage | 文章封面图片URL                           | 否                 |
| summary    | 文章摘要                                  | 是（与文章一对一） |
| content    | 文章正文内容（markdown格式）              | 是（与文章一对一） |

### 数据库表结构

#### posts 相关表

**posts 表** - 文章主表，存储不随语言改变的硬信息

| 字段        | 类型         | 约束              | 说明                  |
| ----------- | ------------ | ----------------- | --------------------- |
| id          | SERIAL       | PRIMARY KEY       | 文章唯一标识符        |
| slug        | VARCHAR(255) | UNIQUE, NOT NULL  | URL 别名              |
| render_type | VARCHAR(20)  | DEFAULT 'tech'    | 渲染方式：tech / life |
| category_id | INT          | FOREIGN KEY       | 关联分类 ID           |
| series_id   | INT          | FOREIGN KEY, NULL | 关联系列 ID（可为空） |
| cover_image | TEXT         |                   | 封面图 URL            |
| view_count  | INT          | DEFAULT 0         | 浏览次数              |
| created_at  | TIMESTAMPTZ  | DEFAULT NOW()     | 创建时间              |
| updated_at  | TIMESTAMPTZ  | DEFAULT NOW()     | 更新时间              |

**索引**：

- `idx_posts_slug` ON slug
- `idx_posts_category_id` ON category_id
- `idx_posts_series_id` ON series_id

---

**post_contents 表** - 文章内容，存储随语言改变的文本

| 字段    | 类型         | 约束                  | 说明              |
| ------- | ------------ | --------------------- | ----------------- |
| id      | SERIAL       | PRIMARY KEY           | 主键              |
| post_id | INT          | FOREIGN KEY, NOT NULL | 关联 posts.id     |
| locale  | VARCHAR(10)  | NOT NULL              | 语言代码 zh/en/jp |
| title   | VARCHAR(255) | NOT NULL              | 文章标题          |
| summary | TEXT         |                       | 文章摘要          |
| content | TEXT         | NOT NULL              | Markdown 正文     |

**约束**：

- UNIQUE(post_id, locale) - 同文章同语言唯一
- FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE

**索引**：

- `idx_post_contents_post_locale` ON (post_id, locale)

---

#### categories 相关表

**categories 表** - 分类主表

| 字段      | 类型         | 约束              | 说明                  |
| --------- | ------------ | ----------------- | --------------------- |
| id        | SERIAL       | PRIMARY KEY       | 分类唯一标识符        |
| slug      | VARCHAR(255) | UNIQUE, NOT NULL  | 分类 URL 别名         |
| parent_id | INT          | FOREIGN KEY, NULL | 父级分类 ID（可为空） |

**索引**：

- `idx_categories_slug` ON slug
- `idx_categories_parent_id` ON parent_id

---

**category_translations 表** - 分类翻译

| 字段        | 类型         | 约束                  | 说明        |
| ----------- | ------------ | --------------------- | ----------- |
| id          | SERIAL       | PRIMARY KEY           | 主键        |
| category_id | INT          | FOREIGN KEY, NOT NULL | 关联分类 ID |
| locale      | VARCHAR(10)  | NOT NULL              | 语言代码    |
| name        | VARCHAR(100) | NOT NULL              | 分类名称    |

**约束**：

- UNIQUE(category_id, locale) - 同分类同语言唯一
- FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE

**索引**：

- `idx_category_translations_category_locale` ON (category_id, locale)

---

#### tags 相关表

**tags 表** - 标签主表

| 字段 | 类型         | 约束             | 说明           |
| ---- | ------------ | ---------------- | -------------- |
| id   | SERIAL       | PRIMARY KEY      | 标签唯一标识符 |
| slug | VARCHAR(255) | UNIQUE, NOT NULL | 标签 URL 别名  |

**索引**：

- `idx_tags_slug` ON slug

---

**tag_translations 表** - 标签翻译

| 字段   | 类型         | 约束                  | 说明        |
| ------ | ------------ | --------------------- | ----------- |
| id     | SERIAL       | PRIMARY KEY           | 主键        |
| tag_id | INT          | FOREIGN KEY, NOT NULL | 关联标签 ID |
| locale | VARCHAR(10)  | NOT NULL              | 语言代码    |
| name   | VARCHAR(100) | NOT NULL              | 标签名称    |

**约束**：

- UNIQUE(tag_id, locale) - 同标签同语言唯一
- FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE

**索引**：

- `idx_tag_translations_tag_locale` ON (tag_id, locale)

---

**post_tags 表** - 文章标签关联表（多对多）

| 字段    | 类型 | 约束                  | 说明        |
| ------- | ---- | --------------------- | ----------- |
| post_id | INT  | FOREIGN KEY, NOT NULL | 关联文章 ID |
| tag_id  | INT  | FOREIGN KEY, NOT NULL | 关联标签 ID |

**约束**：

- PRIMARY KEY (post_id, tag_id)
- FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE
- FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE

**索引**：

- `idx_post_tags_tag_id` ON tag_id

---

#### series 相关表

**series 表** - 系列主表

| 字段 | 类型         | 约束             | 说明           |
| ---- | ------------ | ---------------- | -------------- |
| id   | SERIAL       | PRIMARY KEY      | 系列唯一标识符 |
| slug | VARCHAR(255) | UNIQUE, NOT NULL | 系列 URL 别名  |

**索引**：

- `idx_series_slug` ON slug

---

**series_translations 表** - 系列翻译

| 字段        | 类型         | 约束                  | 说明        |
| ----------- | ------------ | --------------------- | ----------- |
| id          | SERIAL       | PRIMARY KEY           | 主键        |
| series_id   | INT          | FOREIGN KEY, NOT NULL | 关联系列 ID |
| locale      | VARCHAR(10)  | NOT NULL              | 语言代码    |
| name        | VARCHAR(255) | NOT NULL              | 系列名称    |
| description | TEXT         |                       | 系列描述    |

**约束**：

- UNIQUE(series_id, locale) - 同系列同语言唯一
- FOREIGN KEY (series_id) REFERENCES series(id) ON DELETE CASCADE

**索引**：

- `idx_series_translations_series_locale` ON (series_id, locale)
