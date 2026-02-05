<template>
  <main class="page">
    <header class="page__header">
      <p class="page__eyebrow">个人博客</p>
      <h1 class="page__title">文章</h1>
      <p class="page__subtitle">简约风 · 单列从上到下排列</p>
    </header>

    <section class="page__content">
      <div v-if="pending" class="state">加载中…</div>
      <div v-else-if="error" class="state state--error">
        加载失败，请稍后重试。
      </div>
      <div v-else class="list">
        <article v-for="post in posts" :key="post.id" class="list__item">
          <div class="list__meta">
            <span class="list__date">{{ formatDate(post.createdAt) }}</span>
            <span v-if="post.category" class="list__dot">·</span>
            <span v-if="post.category" class="list__category">{{
              post.category.name
            }}</span>
            <span v-if="post.series" class="list__dot">·</span>
            <span v-if="post.series" class="list__series">{{
              post.series.name
            }}</span>
          </div>
          <h2 class="list__title">{{ post.title }}</h2>
          <div v-if="post.tags?.length" class="list__tags">
            <span v-for="tag in post.tags" :key="tag.id" class="list__tag"
              >#{{ tag.name }}</span
            >
          </div>
        </article>

        <p v-if="!posts.length" class="state">暂无文章</p>
      </div>
    </section>
  </main>
</template>

<script setup lang="ts">
interface TagInfo {
  id: number;
  slug: string;
  name: string;
}

interface CategoryInfo {
  id: number;
  slug: string;
  name: string;
}

interface SeriesInfo {
  id: number;
  name: string;
  slug: string;
  description?: string | null;
}

interface PostListItem {
  id: number;
  slug: string;
  title: string;
  renderType: string;
  series: SeriesInfo | null;
  tags: TagInfo[] | null;
  category: CategoryInfo | null;
  locale: string;
  createdAt: string;
  updatedAt: string;
}

interface ApiResponse<T> {
  code: number;
  message: string;
  data: {
    total: number;
    list: T;
  };
}

const config = useRuntimeConfig();
const baseURL = config.public.apiBase;

const { data, pending, error } = await useFetch<ApiResponse<PostListItem[]>>(
  "/v1/posts",
  {
    baseURL,
    query: {
      locale: "zh",
      renderType: "tech",
    },
  },
);

const posts = computed(() => data.value?.data?.list ?? []);

const formatDate = (value: string) => {
  if (!value) return "";
  const date = new Date(value);
  if (Number.isNaN(date.getTime())) return value;
  return date.toLocaleDateString("zh-CN", {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  });
};
</script>

<style scoped>
.page {
  max-width: 760px;
  margin: 0 auto;
  padding: 72px 24px 96px;
}

.page__header {
  margin-bottom: 40px;
}

.page__eyebrow {
  font-size: 13px;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #8b8b8b;
  margin: 0 0 12px;
}

.page__title {
  font-size: 32px;
  font-weight: 600;
  margin: 0 0 12px;
}

.page__subtitle {
  margin: 0;
  color: #5a5a5a;
}

.page__content {
  display: flex;
  flex-direction: column;
}

.state {
  color: #6f6f6f;
  padding: 24px 0;
}

.state--error {
  color: #b3261e;
}

.list {
  display: flex;
  flex-direction: column;
  gap: 28px;
}

.list__item {
  padding-bottom: 24px;
  border-bottom: 1px solid #ececec;
}

.list__meta {
  font-size: 12px;
  color: #8a8a8a;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 6px;
  margin-bottom: 10px;
}

.list__dot {
  opacity: 0.6;
}

.list__title {
  font-size: 20px;
  font-weight: 600;
  margin: 0 0 10px;
  color: #1f1f1f;
}

.list__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.list__tag {
  font-size: 12px;
  color: #6a6a6a;
  background: #f1f1f1;
  padding: 4px 10px;
  border-radius: 999px;
}
</style>
