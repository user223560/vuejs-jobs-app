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
  error: null,
})

// Ф-ция загрузки данных
const loadJobs = async () => {
  state.isLoading = true
  state.error = null

  try {
    const response = await fetch("/data/jobs.json")

    if (!response.ok) {
      throw new Error(`Ошибка HTTP: ${response.status}`)
    }

    const data = await response.json()

    // Проверка на правильный формат структуры данных
    if (!data.jobs || !Array.isArray(data.jobs)) {
      throw new Error("Некорректный формат данных в JSON файле")
    }

    state.jobs = data.jobs
  } catch (error) {
    console.error("Ошибка загрузки данных", error)
    state.error = error.message
  } finally {
    state.isLoading = false
  }
}

// Повторная попытка загрузки данных
const retryLoad = () => {
  loadJobs()
}

// Загрузка данных при монтировании
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
      <!-- Показываем спиннер во время подгрузки вакансий -->
      <div v-if="state.isLoading" class="text-center text-gray-500 py-6">
        <PulseLoader />
        <p class="mt-2">Загрузка вакансий...</p>
      </div>

      <!-- Состояние ошибки -->
      <div v-else-if="state.error" class="text-center text-red-500 py-6">
        <p class="font-semibold">Не удалось загрузить вакансии</p>
        <p class="text-sm mt-1">{{ state.error }}</p>
        <button
          @click="retryLoad"
          class="mt-4 px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600"
        >
          Попробовать снова
        </button>
      </div>

      <!-- Успешная загрузка -->
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
