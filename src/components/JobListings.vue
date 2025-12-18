<script setup>
import { ref } from "vue"
import jobData from "@/jobs.json"

const jobs = ref(jobData.jobs)
</script>

<template>
  <section>
    <div class="container-xl lg:container m-auto">
      <h2 class="text-3xl font-bold text-green-500 mb-6 text-center">
        Поиск вакансий
      </h2>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div v-for="job in jobs" :key="job.id">
          {{ job.title }}
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped></style>
