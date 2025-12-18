<script setup>
import Card from "./Card.vue"
</script>
<template>
  <section class="py-4">
    <div class="container-xl lg:container m-auto">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 p-4 rounded-lg">
        <Card>
          <h2 class="text-2xl font-bold">Для соискателей</h2>
          <p class="mt-2 mb-4">Легкий и удобный поиск вакансий</p>
          <a
            href="jobs.html"
            class="inline-block bg-black text-white rounded-lg px-4 py-2 hover:bg-gray-700"
          >
            Найти вакансию
          </a>
        </Card>
        <Card bg="bg-green-100">
          <h2 class="text-2xl font-bold">Для работодателей</h2>
          <p class="mt-2 mb-4">
            Разместите свою вакансию для подбора лучшего специалиста
          </p>
          <a
            href="add-job.html"
            class="inline-block bg-green-500 text-white rounded-lg px-4 py-2 hover:bg-green-600"
          >
            Разместить вакансию
          </a>
        </Card>
      </div>
    </div>
  </section>
</template>
<style scoped></style>
