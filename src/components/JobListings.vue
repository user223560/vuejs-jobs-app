<script setup>
import { reactive, defineProps, onMounted } from "vue"
import { RouterLink } from "vue-router"
import PulseLoader from "vue-spinner/src/PulseLoader.vue"

import JobListing from "./JobListing.vue"
import ConnectionError from "@/components/ConnectionError.vue"

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
  retryCount: 0,
})

const loadJobs = async () => {
  state.isLoading = true
  state.error = null

  try {
    // Загружаем данные из json-файла
    const response = await fetch(`${import.meta.env.BASE_URL}data/jobs.json`)

    if (!response.ok) {
      throw new Error(`Ошибка загрузки данных: ${response.status}`)
    }

    const jsonData = await response.json()

    if (!jsonData.jobs || !Array.isArray(jsonData.jobs)) {
      throw new Error("Некорректный формат данных в файле")
    }

    const demoJobs = jsonData.jobs

    // Загружаем пользовательские данные из LocalStorage
    const stored = localStorage.getItem("jobs_app_data")
    const storageData = stored ? JSON.parse(stored) : {}
    const userJobs = storageData.jobs || []

    // Объединяем данные
    state.jobs = [...demoJobs, ...userJobs]
    state.retryCount = 0 // Сбрасываем счетчик при успехе
  } catch (error) {
    console.error("Ошибка загрузки", error)
    state.error = error.message
    state.retryCount++

    state.jobs = []
  } finally {
    state.isLoading = false
  }
}

// Повторная попытка
const retryLoad = () => {
  loadJobs()
}

onMounted(() => {
  loadJobs()
})
</script>

<template>
  <section>
    <div class="container-xl lg:container m-auto">
      <h2 class="text-3xl font-bold text-green-500 my-6 text-center">
        Обзор вакансий
      </h2>

      <!-- Используем компонент ConnectionError для состояний загрузки и ошибки -->
      <ConnectionError
        v-if="state.isLoading || (state.error && state.jobs.length === 0)"
        :isLoading="state.isLoading"
        :error="state.error && state.jobs.length === 0"
        :retryCount="state.retryCount"
        :loadingText="state.isLoading ? 'Загрузка вакансий...' : ''"
        @retry="retryLoad"
      />

      <!-- Основной контент (только при наличии данных) -->
      <div v-else>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 pb-6">
          <JobListing
            v-for="job in state.jobs.slice(0, limit || state.jobs.length)"
            :key="job.id"
            :job="job"
          />
        </div>

        <!-- Сообщение если нет вакансий (без ошибки) -->
        <div
          v-if="state.jobs.length === 0 && !state.error"
          class="text-center py-10"
        >
          <div class="text-gray-500 text-4xl mb-4">📭</div>
          <h2 class="text-xl font-bold text-gray-600 mb-2">
            Вакансий пока нет
          </h2>
          <p class="text-gray-700 mb-4">Будьте первым, кто добавит вакансию!</p>
          <RouterLink
            to="/jobs/add"
            class="inline-block px-6 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600"
          >
            Добавить первую вакансию
          </RouterLink>
        </div>
      </div>
    </div>
  </section>

  <!-- Кнопка "Посмотреть все вакансии" -->
  <section
    v-if="
      showButton && !state.isLoading && !state.error && state.jobs.length > 0
    "
    class="m-auto max-w-lg my-10 px-6 transition-opacity duration-300"
  >
    <RouterLink
      to="/jobs"
      class="block bg-black text-white text-center py-4 px-6 rounded-xl hover:bg-gray-700"
    >
      Посмотреть все вакансии
    </RouterLink>
  </section>
</template>

<style scoped></style>
