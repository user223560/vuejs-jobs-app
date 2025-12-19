<script setup>
import { RouterLink } from "vue-router"
</script>

<template>
  <section class="text-center flex flex-col justify-center items-center h-96">
    <i class="bi bi-exclamation-triangle text-yellow-500 text-7xl mb-5"></i>
    <h1 class="text-6xl font-bold mb-4">Ошибка 404</h1>
    <p class="text-xl mb-5">Эта страница была удалена или перемещена</p>
    <RouterLink
      to="/"
      class="text-white bg-green-700 hover:bg-green-900 rounded-md px-3 py-2 mt-4"
    >
      Назад
    </RouterLink>
  </section>
</template>

<style scoped></style>
