<script setup>
import { RouterLink } from "vue-router"
</script>

<template>
  <section>
    <div class="container m-auto py-6 px-6">
      <RouterLink
        to="/jobs"
        class="text-green-500 hover:text-green-600 flex items-center"
      >
        <i class="bi bi-arrow-left-circle mr-3"></i> Назад к списку вакансий
      </RouterLink>
    </div>
  </section>
</template>

<style scoped></style>
