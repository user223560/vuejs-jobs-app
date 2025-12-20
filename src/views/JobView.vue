<script setup>
import { reactive, onMounted, watch, computed } from "vue"
import { RouterLink, useRoute, useRouter } from "vue-router"
import { useToast } from "vue-toastification"

import PulseLoader from "vue-spinner/src/PulseLoader.vue"
import BackButton from "@/components/BackButton.vue"

const route = useRoute()
const jobId = route.params.id

const router = useRouter()

const toast = useToast()

const state = reactive({
  job: {},
  isLoading: true,
})

// Computed свойство для проверки типа вакансии
const isUserJob = computed(() => {
  return state.job.isUserJob === true
})

const deleteJob = async () => {
  try {
    const confirmDelete = window.confirm(
      "Вы уверены, что хотите удалить эту вакансию?"
    )

    if (!confirmDelete) {
      return
    }

    // Получение данных из LocalStorage
    const storedData = localStorage.getItem("jobs_app_data")

    if (!storedData) {
      throw new Error("Нет данных в LocalStorage")
    }

    const data = JSON.parse(storedData)

    // Проверка правильности структуры данных (что такой массив существует)
    if (!data.jobs || !Array.isArray(data.jobs)) {
      throw new Error("Некорректный формат данных")
    }

    // Фильтрация массива, убирается вакансия с нужным ID
    const initialLength = data.jobs.length
    data.jobs = data.jobs.filter((job) => job.id !== jobId)

    // Проверка, если вакансия уже удалена
    if (data.jobs.length === initialLength) {
      throw new Error("Вакансия не найдена")
    }

    // Сохранение обновленных данных
    localStorage.setItem("jobs_app_data", JSON.stringify(data))

    toast.success("Вакансия успешно удалена!")
    router.push("/jobs")
  } catch (error) {
    console.error("Ошибка удаления вакансии", error)

    if (error.message === "Вакансия не найдена") {
      toast.error("Вакансия уже была удалена")
      router.push("/jobs")
    } else {
      toast.error("Не удалось удалить вакансию")
    }
  }
}

// Функция загрузки вакансии
const loadJob = async (id) => {
  state.isLoading = true

  try {
    // Сначала ищем в LocalStorage
    const storedData = localStorage.getItem("jobs_app_data")
    let foundJob = null

    if (storedData) {
      const data = JSON.parse(storedData)
      foundJob = data.jobs?.find((job) => job.id === id)
    }

    // Если не нашли, ищем в JSON файле
    if (!foundJob) {
      const response = await fetch("/data/jobs.json")
      const data = await response.json()
      foundJob = data.jobs?.find((job) => job.id === id)
    }

    state.job = foundJob || {}
  } catch (error) {
    console.error("Ошибка загрузки", error)
    toast.error("Не удалось загрузить данные о вакансии")
  } finally {
    state.isLoading = false
  }
}

// Первоначальная загрузка
onMounted(() => {
  loadJob(jobId)
})

// Ф-ция watch для отслеживания изменения ID (плавная навигация без перезагрузки страницы)
watch(
  () => route.params.id,
  (newId) => {
    if (newId) {
      loadJob(newId)
    }
  }
)
</script>

<template>
  <BackButton />
  <section v-if="!state.isLoading" class="bg-green-50">
    <div class="container m-auto py-10 px-6">
      <div class="grid grid-cols-1 md:grid-cols-70/30 w-full gap-6">
        <main>
          <div
            class="bg-white p-6 rounded-lg shadow-md text-center md:text-left"
          >
            <div class="text-gray-500 mb-4">{{ state.job.type }}</div>
            <h1 class="text-3xl font-bold mb-4">{{ state.job.title }}</h1>
            <div
              class="text-gray-500 mb-4 flex align-middle justify-center md:justify-start"
            >
              <i class="bi bi-geo-alt text-xl text-orange-700 mr-2"></i>
              <p class="text-orange-700">{{ state.job.location }}</p>
            </div>
          </div>

          <div class="bg-white p-6 rounded-lg shadow-md mt-6 break-words">
            <h3 class="text-green-800 text-lg font-bold mb-6">
              Описание вакансии
            </h3>

            <p class="mb-4">
              {{ state.job.description }}
            </p>

            <h3 class="text-green-800 text-lg font-bold mb-2">Зарплата</h3>

            <p class="mb-4">{{ state.job.salary || "Не указана" }}</p>
          </div>
        </main>
        <aside>
          <div
            class="bg-white p-6 rounded-lg shadow-md [word-break:break-word]"
          >
            <h3 class="text-xl font-bold mb-6">О работодателе</h3>

            <h2 class="text-2xl">{{ state.job.company?.name }}</h2>

            <p class="my-2">
              {{ state.job.company?.description }}
            </p>

            <hr class="my-4" />

            <h3 class="text-xl">Электронная почта:</h3>

            <p class="my-2 bg-green-100 p-2 font-bold">
              {{ state.job.company?.contactEmail }}
            </p>

            <h3 class="text-xl">Телефон:</h3>

            <p class="my-2 bg-green-100 p-2 font-bold">
              {{ state.job.company?.contactPhone || "Не указан" }}
            </p>
          </div>
          <div v-if="isUserJob" class="bg-white p-6 rounded-lg shadow-md mt-6">
            <h3 class="text-xl text-center font-bold mb-6">
              Управление вакансией
            </h3>
            <RouterLink
              :to="`/jobs/edit/${state.job.id}`"
              class="bg-green-500 hover:bg-green-600 text-white text-center font-bold py-2 px-4 rounded-full w-full focus:outline-none focus:shadow-outline mt-4 block"
            >
              Редактировать
            </RouterLink>
            <button
              @click="deleteJob"
              class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-full w-full focus:outline-none focus:shadow-outline mt-4 block"
            >
              Удалить
            </button>
          </div>
        </aside>
      </div>
    </div>
  </section>
  <div v-else class="text-center text-gray-500 py-6">
    <PulseLoader />
  </div>
</template>

<style scoped></style>
