<script setup lang="ts">
await navigateTo("/posts", { replace: true });
</script>

<template>
  <main class="page">
    <p class="text">正在跳转到文章列表…</p>
  </main>
</template>

<style scoped>
.page {
  min-height: 100vh;
  display: grid;
  place-items: center;
  color: #6f6f6f;
}
</style>
