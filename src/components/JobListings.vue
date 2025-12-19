<script setup>
import { ref, defineProps } from "vue"
import jobData from "@/jobs.json"
import JobListing from "./JobListing.vue"

const jobs = ref(jobData.jobs)

defineProps({
  limit: Number,
  showButton: {
    type: Boolean,
    default: false,
  },
})
</script>

<template>
  <section>
    <div class="container-xl lg:container m-auto">
      <h2 class="text-3xl font-bold text-green-500 mb-6 text-center">
        Поиск вакансий
      </h2>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <JobListing
          v-for="job in jobs.slice(0, limit || jobs.length)"
          :key="job.id"
          :job="job"
        />
      </div>
    </div>
  </section>
  <section class="m-auto max-w-lg my-10 px-6" v-if="showButton">
    <a
      href="/jobs"
      class="block bg-black text-white text-center py-4 px-6 rounded-xl hover:bg-gray-700"
    >
      Посмотреть все вакансии
    </a>
  </section>
</template>

<style scoped></style>
