<template>
  <n-card title="关于" :bordered="false" size="large" class="rounded-16px shadow-sm">
    <p class="leading-24px">
      本系统前端基于Soybean
      Admin，它使用了最新的前端技术栈，并提炼了典型的业务模型和页面，后端使用Python的Flask框架。实现了对于用户上传数据的存储、管理以及可视化。
    </p>
  </n-card>
</template>

<script setup lang="ts">
defineOptions({ name: 'ProjectIntroduction' });
</script>

<style scoped></style>
