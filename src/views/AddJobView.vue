<script setup>
import { reactive, ref, onMounted } from "vue"
import { useToast } from "vue-toastification"

import router from "@/router"
import ConnectionError from "@/components/ConnectionError.vue"

// Состояние для проверки соединения
const state = reactive({
  hasConnection: false,
  isLoading: true,
  retryCount: 0,
})

// Функция проверки соединения с сервером
const checkServerConnection = async () => {
  state.isLoading = true
  try {
    const response = await fetch("/data/jobs.json")
    if (!response.ok) {
      throw new Error(`Ошибка соединения: ${response.status}`)
    }
    state.hasConnection = true
    state.retryCount = 0
  } catch (error) {
    console.error("Ошибка соединения с сервером:", error)
    state.hasConnection = false
    state.retryCount++
  } finally {
    state.isLoading = false
  }
}

// Функция повторной попытки
const retryConnection = () => {
  checkServerConnection()
}

onMounted(() => {
  checkServerConnection()
})

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

// Функция сохранения вакансии в LocalStorage
const saveJobToLocalStorage = (job) => {
  try {
    // Получаем текущие данные из LocalStorage
    const storedData = localStorage.getItem("jobs_app_data")
    const data = storedData ? JSON.parse(storedData) : { jobs: [] }

    // Генерируем уникальный ID по timestamp
    const newId = Date.now().toString()

    // Создаем объект вакансии
    const newJob = {
      id: newId,
      type: job.type,
      title: job.title,
      description: job.description,
      salary: job.salary,
      location: job.location,
      company: {
        name: job.company.name,
        description: job.company.description,
        contactEmail: job.company.contactEmail,
        contactPhone: job.company.contactPhone,
      },
      isUserJob: true, // Помечаем как пользовательскую
      createdAt: new Date().toISOString(),
    }

    // Добавляем в массив
    data.jobs.push(newJob)

    // Сохраняем обратно в LocalStorage
    localStorage.setItem("jobs_app_data", JSON.stringify(data))

    console.log("Вакансия сохранена в LocalStorage с ID:", newId)
    return newId
  } catch (error) {
    console.error("Ошибка сохранения в LocalStorage:", error)
    throw error
  }
}

const handleSubmit = async () => {
  // Валидация формы
  if (!validateForm()) {
    toast.warning("Заполните обязательные поля")
    return
  }

  try {
    // Сохраняем в LocalStorage
    const jobId = saveJobToLocalStorage(form)

    // Показываем уведомление
    toast.success("Вакансия успешно опубликована!")

    // Перенаправляем на страницу созданной вакансии
    router.push(`/jobs/${jobId}`)
  } catch (error) {
    console.error("Ошибка добавления данных", error)
    toast.error("Произошла ошибка при публикации вакансии")
  }
}

// Сброс ошибки при вводе в поле
const clearError = (field) => {
  if (errors.value[field]) {
    errors.value[field] = ""
  }
}
</script>

<template>
  <section class="bg-green-50">
    <ConnectionError
      v-if="!state.hasConnection"
      :isLoading="state.isLoading"
      :error="!state.hasConnection && !state.isLoading"
      :retryCount="state.retryCount"
      :loadingText="state.isLoading ? 'Проверка соединения...' : ''"
      @retry="retryConnection"
    />

    <div v-else>
      <div class="container m-auto max-w-2xl py-24">
        <div
          class="bg-white px-6 py-8 mb-4 shadow-md rounded-md border m-4 md:m-0"
        >
          <form @submit.prevent="handleSubmit" novalidate>
            <h2 class="text-3xl text-center font-semibold mb-6">
              Добавление вакансии
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
                :class="{ 'border-red-500': errors.title }"
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
                class="border rounded w-full py-2 px-3 mb-2"
                :class="{ 'border-red-500': errors.title }"
                placeholder="напр. Middle Frontend Developer"
              />
              <p v-if="errors.title" class="text-red-500 text-sm">
                {{ errors.title }}
              </p>
            </div>
            <div class="mb-4">
              <label
                for="description"
                class="block text-gray-700 font-bold mb-2"
                >Описание вакансии *</label
              >
              <textarea
                v-model="form.description"
                @input="clearError('description')"
                id="description"
                name="description"
                class="border rounded w-full py-2 px-3"
                :class="{ 'border-red-500': errors.description }"
                rows="4"
                placeholder="Опишите требования, обязанности, ваши ожидания и т.д."
              ></textarea>
              <p v-if="errors.description" class="text-red-500 text-sm">
                {{ errors.description }}
              </p>
            </div>

            <div class="mb-4">
              <label for="type" class="block text-gray-700 font-bold mb-2"
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
                <option value="90 000 ₽ - 100 000 ₽">
                  90 000 ₽ - 100 000 ₽
                </option>
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
                class="border rounded w-full py-2 px-3 mb-2"
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
                class="border rounded w-full py-2 px-3"
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
                >Сфера деятельности *</label
              >
              <textarea
                v-model="form.company.description"
                @input="clearError('companyDescription')"
                id="company_description"
                name="company_description"
                class="border rounded w-full py-2 px-3"
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
                class="border rounded w-full py-2 px-3"
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

            <div>
              <button
                class="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-full w-full focus:outline-none focus:shadow-outline"
                type="submit"
              >
                Разместить вакансию
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped></style>
