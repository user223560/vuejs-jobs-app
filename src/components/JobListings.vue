<script setup>
import { reactive, defineProps, onMounted } from "vue"
import { RouterLink } from "vue-router"
import PulseLoader from "vue-spinner/src/PulseLoader.vue"

import JobListing from "./JobListing.vue"

defineProps({
  limit: Number,
  showButton: {
    type: Boolean,
    default: false,
  },
})

const state = reactive({
  jobs: [],
  isLoading: true,
})

const loadJobs = async () => {
  state.isLoading = true

  try {
    // Всегда показывать из файла и из локального хранилища
    const response = await fetch("/data/jobs.json")
    const jsonData = await response.json()
    const demoJobs = jsonData.jobs || []

    const stored = localStorage.getItem("jobs_app_data")
    const storageData = stored ? JSON.parse(stored) : {}
    const userJobs = storageData.jobs || []

    state.jobs = [...demoJobs, ...userJobs] // Сначала новые, потом старые
  } catch (error) {
    console.error("Ошибка загрузки", error)
  } finally {
    state.isLoading = false
  }
}

onMounted(() => {
  loadJobs()
})
</script>

<template>
  <section>
    <div class="container-xl lg:container m-auto">
      <h2 class="text-3xl font-bold text-green-500 mb-6 text-center">
        Поиск вакансий
      </h2>

      <div v-if="state.isLoading" class="text-center text-gray-500 py-6">
        <PulseLoader />
      </div>

      <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <JobListing
          v-for="job in state.jobs.slice(0, limit || state.jobs.length)"
          :key="job.id"
          :job="job"
        />
      </div>
    </div>
  </section>

  <section class="m-auto max-w-lg my-10 px-6" v-if="showButton">
    <RouterLink
      to="/jobs"
      class="block bg-black text-white text-center py-4 px-6 rounded-xl hover:bg-gray-700"
    >
      Посмотреть все вакансии
    </RouterLink>
  </section>
</template>

<style scoped></style>
