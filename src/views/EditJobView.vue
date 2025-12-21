<script setup>
import { onMounted, reactive, ref } from "vue"
import { useRoute } from "vue-router"
import { useToast } from "vue-toastification"

import router from "@/router"

const route = useRoute()

const jobId = route.params.id

const form = reactive({
  type: "Полная занятость",
  title: "",
  description: "",
  salary: "",
  location: "",
  company: {
    name: "",
    description: "",
    contactEmail: "",
    contactPhone: "",
  },
})

const state = reactive({
  isLoading: true,
  error: null,
})

// Состояния ошибок для каждого поля
const errors = ref({
  title: "",
  description: "",
  location: "",
  companyName: "",
  companyDescription: "",
  contactEmail: "",
})

const toast = useToast()

// Функция загрузки вакансии для редактирования
const loadJobForEdit = async () => {
  state.isLoading = true
  state.error = null

  try {
    // Поиск записи в LocalStorage
    const storedData = localStorage.getItem("jobs_app_data")

    if (!storedData) {
      throw new Error("Нет данных в LocalStorage")
    }

    const data = JSON.parse(storedData)

    if (!data.jobs || !Array.isArray(data.jobs)) {
      throw new Error("Некорректный формат данных")
    }

    const foundJob = data.jobs.find((job) => job.id === jobId)

    if (!foundJob) {
      throw new Error("Вакансия не найдена")
    }

    // Проверяем, что это пользовательская вакансия
    if (!foundJob.isUserJob) {
      throw new Error("Демо-вакансии нельзя редактировать")
    }

    // Заполняем форму данными вакансии
    form.type = foundJob.type || "Полная занятость"
    form.title = foundJob.title || ""
    form.description = foundJob.description || ""
    form.salary = foundJob.salary || ""
    form.location = foundJob.location || ""
    form.company.name = foundJob.company?.name || ""
    form.company.description = foundJob.company?.description || ""
    form.company.contactEmail = foundJob.company?.contactEmail || ""
    form.company.contactPhone = foundJob.company?.contactPhone || ""
  } catch (error) {
    console.error("Ошибка загрузки вакансии:", error)
    state.error = error.message
    toast.error(`Не удалось загрузить вакансию: ${error.message}`)

    // Перенаправляем обратно, если вакансия не найдена
    if (
      error.message.includes("не найдена") ||
      error.message.includes("Демо")
    ) {
      setTimeout(() => router.push("/jobs"), 1500)
    }
  } finally {
    state.isLoading = false
  }
}

// Валидация формы
const validateForm = () => {
  let isValid = true

  // Очищаем предыдущие ошибки
  errors.value = {
    title: "",
    description: "",
    location: "",
    companyName: "",
    companyDescription: "",
    contactEmail: "",
  }

  // Проверка названия вакансии
  if (!form.title.trim()) {
    errors.value.title = "Обязательно для заполнения"
    isValid = false
  }

  // Проверка описания вакансии
  if (!form.description.trim()) {
    errors.value.description = "Обязательно для заполнения"
    isValid = false
  }

  // Проверка места работы
  if (!form.location.trim()) {
    errors.value.location = "Обязательно для заполнения"
    isValid = false
  }

  // Проверка названия компании
  if (!form.company.name.trim()) {
    errors.value.companyName = "Обязательно для заполнения"
    isValid = false
  }

  // Проверка описания компании
  if (!form.company.description.trim()) {
    errors.value.companyDescription = "Обязательно для заполнения"
    isValid = false
  }

  // Проверка email
  if (!form.company.contactEmail.trim()) {
    errors.value.contactEmail = "Обязательно для заполнения"
    isValid = false
  } else if (!isValidEmail(form.company.contactEmail)) {
    errors.value.contactEmail = "Пожалуйста, введите почту в правильном формате"
    isValid = false
  }

  return isValid
}

// Проверка email регулярным выражением
const isValidEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(email)
}

// Ф-ция обновления вакансии в LocalStorage
const updateJobInLocalStorage = () => {
  try {
    // Вытаскиваем текущие данные из LocalStorage
    const storedData = localStorage.getItem("jobs_app_data")

    if (!storedData) {
      throw new Error("Нет данных в LocalStorage")
    }

    const data = JSON.parse(storedData)

    if (!data.jobs || !Array.isArray(data.jobs)) {
      throw new Error("Некорректный формат данных")
    }

    // Ищем индекс вакансии для обновления
    const jobIndex = data.jobs.findIndex((job) => job.id === jobId)

    if (jobIndex === -1) {
      throw new Error("Вакансия не найдена")
    }

    // Обновляем вакансию (сохраняем оригинальные поля и добавляем обновленные)
    data.jobs[jobIndex] = {
      ...data.jobs[jobIndex], // Сохраняем оригинальные данные (id, isUserJob, createdAt)
      type: form.type,
      title: form.title,
      description: form.description,
      salary: form.salary,
      location: form.location,
      company: {
        name: form.company.name,
        description: form.company.description,
        contactEmail: form.company.contactEmail,
        contactPhone: form.company.contactPhone,
      },
      updatedAt: new Date().toISOString(), // Добавляем время обновления
    }

    // Сохраняем обратно в LocalStorage
    localStorage.setItem("jobs_app_data", JSON.stringify(data))

    console.log("Вакансия обновлена в LocalStorage с ID:", jobId)
    return jobId
  } catch (error) {
    console.error("Ошибка обновления вакансии:", error)
    throw error
  }
}

const handleSubmit = async () => {
  // Валидация формы
  if (!validateForm()) {
    toast.error("Заполните обязательные поля")
    return
  }

  try {
    // Обновляем вакансию в LocalStorage
    updateJobInLocalStorage()

    // Показываем уведомление
    toast.success("Вакансия успешно обновлена!")

    // Перенаправляем на страницу вакансии
    router.push(`/jobs/${jobId}`)
  } catch (error) {
    console.error("Ошибка обновления данных", error)
    toast.error("Произошла ошибка при обновлении вакансии")
  }
}

// Сброс ошибки при вводе в поле
const clearError = (field) => {
  if (errors.value[field]) {
    errors.value[field] = ""
  }
}

onMounted(() => {
  loadJobForEdit()
})
</script>

<template>
  <section class="bg-green-50">
    <div class="container m-auto max-w-2xl py-24">
      <div v-if="state.isLoading" class="text-center py-10">
        <div
          class="inline-block animate-spin rounded-full h-8 w-8 border-b-2 border-green-500"
        ></div>
        <p class="mt-2 text-gray-600">Загрузка данных вакансии...</p>
      </div>
      <div v-else-if="state.error" class="text-center py-10">
        <i class="bi bi-exclamation-triangle text-red-500 text-4xl mb-4"></i>
        <h2 class="text-xl font-bold text-red-600 mb-2">Ошибка</h2>
        <p class="text-gray-700 mb-4">{{ state.error }}</p>
        <button
          @click="router.push('/jobs')"
          class="inline-block bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600"
        >
          Вернуться к списку вакансий
        </button>
      </div>
      <div
        v-else
        class="bg-white px-6 py-8 mb-4 shadow-md rounded-md border m-4 md:m-0"
      >
        <form @submit.prevent="handleSubmit" novalidate>
          <h2 class="text-3xl text-center font-semibold mb-6">
            Редактирование вакансии
          </h2>
          <div class="mb-4">
            <label for="type" class="block text-gray-700 font-bold mb-2"
              >Тип вакансии</label
            >
            <select
              v-model="form.type"
              id="type"
              name="type"
              class="border rounded w-full py-2 px-3"
            >
              <option value="Полная занятость">Полная занятость</option>
              <option value="Частичная занятость">Частичная занятость</option>
              <option value="Удаленная работа">Удаленная работа</option>
              <option value="Стажировка">Стажировка</option>
            </select>
          </div>
          <div class="mb-4">
            <label class="block text-gray-700 font-bold mb-2"
              >Наименование должности *</label
            >
            <input
              v-model="form.title"
              @input="clearError('title')"
              type="text"
              id="name"
              name="name"
              class="border rounded w-full py-2 px-3 mb-1"
              :class="{ 'border-red-500': errors.title }"
              placeholder="напр. Middle Frontend Developer"
            />
            <p v-if="errors.title" class="text-red-500 text-sm">
              {{ errors.title }}
            </p>
          </div>
          <div class="mb-4">
            <label for="description" class="block text-gray-700 font-bold mb-2"
              >Описание вакансии *</label
            >
            <textarea
              v-model="form.description"
              @input="clearError('description')"
              id="description"
              name="description"
              class="border rounded w-full py-2 px-3 mb-1"
              :class="{ 'border-red-500': errors.description }"
              rows="4"
              placeholder="Опишите требования, обязанности, ваши ожидания и т.д."
            ></textarea>
            <p v-if="errors.description" class="text-red-500 text-sm">
              {{ errors.description }}
            </p>
          </div>
          <div class="mb-4">
            <label for="salary" class="block text-gray-700 font-bold mb-2"
              >Зарплата</label
            >
            <select
              v-model="form.salary"
              id="salary"
              name="salary"
              class="border rounded w-full py-2 px-3"
            >
              <option value="">Не указана</option>
              <option value="До 50 000 ₽">До 50 000 ₽</option>
              <option value="50 000 ₽ - 60 000 ₽">50 000 ₽ - 60 000 ₽</option>
              <option value="60 000 ₽ - 70 000 ₽">60 000 ₽ - 70 000 ₽</option>
              <option value="70 000 ₽ - 80 000 ₽">70 000 ₽ - 80 000 ₽</option>
              <option value="80 000 ₽ - 90 000 ₽">80 000 ₽ - 90 000 ₽</option>
              <option value="90 000 ₽ - 100 000 ₽">90 000 ₽ - 100 000 ₽</option>
              <option value="110 000 ₽ - 120 000 ₽">
                110 000 ₽ - 120 000 ₽
              </option>
              <option value="125 000 ₽ - 150 000 ₽">
                125 000 ₽ - 150 000 ₽
              </option>
              <option value="150 000 ₽ - 175 000 ₽">
                150 000 ₽ - 175 000 ₽
              </option>
              <option value="180 000 ₽ - 200 000 ₽">
                180 000 ₽ - 200 000 ₽
              </option>
              <option value="От 200 000 ₽">От 200 000 ₽</option>
            </select>
          </div>
          <div class="mb-4">
            <label class="block text-gray-700 font-bold mb-2">
              Место работы *
            </label>
            <input
              v-model="form.location"
              @input="clearError('location')"
              type="text"
              id="location"
              name="location"
              class="border rounded w-full py-2 px-3 mb-1"
              :class="{ 'border-red-500': errors.location }"
              placeholder="Местонахождение компании"
            />
            <p v-if="errors.location" class="text-red-500 text-sm">
              {{ errors.location }}
            </p>
          </div>
          <h3 class="text-2xl mb-5">Информация о работодателе</h3>
          <div class="mb-4">
            <label for="company" class="block text-gray-700 font-bold mb-2"
              >Название организации (ФИО ИП) *</label
            >
            <input
              v-model="form.company.name"
              @input="clearError('companyName')"
              type="text"
              id="company"
              name="company"
              class="border rounded w-full py-2 px-3 mb-1"
              :class="{ 'border-red-500': errors.companyName }"
              placeholder="напр. ПАО «Сбербанк России» или ИП Иванов Иван И."
            />
            <p v-if="errors.companyName" class="text-red-500 text-sm">
              {{ errors.companyName }}
            </p>
          </div>
          <div class="mb-4">
            <label
              for="company_description"
              class="block text-gray-700 font-bold mb-2"
              >Сфера деятельности организации *</label
            >
            <textarea
              v-model="form.company.description"
              @input="clearError('companyDescription')"
              id="company_description"
              name="company_description"
              class="border rounded w-full py-2 px-3 mb-1"
              :class="{ 'border-red-500': errors.companyDescription }"
              rows="4"
              placeholder="Чем занимается ваша компания?"
            ></textarea>
            <p v-if="errors.companyDescription" class="text-red-500 text-sm">
              {{ errors.companyDescription }}
            </p>
          </div>
          <div class="mb-4">
            <label
              for="contact_email"
              class="block text-gray-700 font-bold mb-2"
              >Электронная почта *</label
            >
            <input
              v-model="form.company.contactEmail"
              @input="clearError('contactEmail')"
              type="email"
              id="contact_email"
              name="contact_email"
              class="border rounded w-full py-2 px-3 mb-1"
              :class="{ 'border-red-500': errors.contactEmail }"
              placeholder="Электронная почта для соискателей"
            />
            <p v-if="errors.contactEmail" class="text-red-500 text-sm">
              {{ errors.contactEmail }}
            </p>
          </div>
          <div class="mb-4">
            <label
              for="contact_phone"
              class="block text-gray-700 font-bold mb-2"
              >Телефон</label
            >
            <input
              v-model="form.company.contactPhone"
              type="tel"
              id="contact_phone"
              name="contact_phone"
              class="border rounded w-full py-2 px-3"
              placeholder="Телефон для соискателей"
            />
          </div>
          <div class="text-sm text-gray-500 mb-4">
            <p>* — обязательные для заполнения поля</p>
          </div>
          <div class="flex gap-4">
            <button
              type="button"
              @click="router.push(`/jobs/${jobId}`)"
              class="flex-1 bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-4 rounded-full focus:outline-none focus:shadow-outline"
            >
              Отмена
            </button>
            <button
              type="submit"
              class="flex-1 bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-full focus:outline-none focus:shadow-outline"
            >
              Сохранить изменения
            </button>
          </div>
        </form>
      </div>
    </div>
  </section>
</template>

<style scoped></style>
