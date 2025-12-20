<script setup>
import { reactive } from "vue"

import router from "@/router"

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
  try {
    // Валидация обязательных полей
    if (!form.title.trim()) {
      alert("Введите название вакансии")
      return
    }

    if (!form.company.contactEmail) {
      alert("Введите email для связи")
      return
    }

    // Сохраняем в LocalStorage
    const jobId = saveJobToLocalStorage(form)

    // Показываем уведомление
    alert("Вакансия успешно добавлена!")

    // Перенаправляем на страницу созданной вакансии
    router.push(`/jobs/${jobId}`)
  } catch (error) {
    console.error("Ошибка добавления данных", error)
    alert("Произошла ошибка при сохранении вакансии")
  }
}
</script>

<template>
  <section class="bg-green-50">
    <div class="container m-auto max-w-2xl py-24">
      <div
        class="bg-white px-6 py-8 mb-4 shadow-md rounded-md border m-4 md:m-0"
      >
        <form @submit.prevent="handleSubmit">
          <h2 class="text-3xl text-center font-semibold mb-6">
            Добавить вакансию
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
              required
            >
              <option value="Полная занятость">Полная занятость</option>
              <option value="Частичная занятость">Частичная занятость</option>
              <option value="Удаленная работа">Удаленная работа</option>
              <option value="Стажировка">Стажировка</option>
            </select>
          </div>

          <div class="mb-4">
            <label class="block text-gray-700 font-bold mb-2"
              >Наименование должности</label
            >
            <input
              v-model="form.title"
              type="text"
              id="name"
              name="name"
              class="border rounded w-full py-2 px-3 mb-2"
              placeholder="напр. Middle Frontend Developer"
              required
            />
          </div>
          <div class="mb-4">
            <label for="description" class="block text-gray-700 font-bold mb-2"
              >Описание</label
            >
            <textarea
              v-model="form.description"
              id="description"
              name="description"
              class="border rounded w-full py-2 px-3"
              rows="4"
              placeholder="Опишите требования, обязанности, ваши ожидания и т.д."
            ></textarea>
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
              required
            >
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
              Место работы
            </label>
            <input
              v-model="form.location"
              type="text"
              id="location"
              name="location"
              class="border rounded w-full py-2 px-3 mb-2"
              placeholder="Местонахождение компании"
              required
            />
          </div>

          <h3 class="text-2xl mb-5">Информация о работодателе</h3>

          <div class="mb-4">
            <label for="company" class="block text-gray-700 font-bold mb-2"
              >Название организации (ФИО ИП)</label
            >
            <input
              v-model="form.company.name"
              type="text"
              id="company"
              name="company"
              class="border rounded w-full py-2 px-3"
              placeholder="Название организации (ФИО ИП)"
            />
          </div>

          <div class="mb-4">
            <label
              for="company_description"
              class="block text-gray-700 font-bold mb-2"
              >Сфера деятельности</label
            >
            <textarea
              v-model="form.company.description"
              id="company_description"
              name="company_description"
              class="border rounded w-full py-2 px-3"
              rows="4"
              placeholder="Чем занимается ваша компания?"
            ></textarea>
          </div>

          <div class="mb-4">
            <label
              for="contact_email"
              class="block text-gray-700 font-bold mb-2"
              >Электронная почта</label
            >
            <input
              v-model="form.company.contactEmail"
              type="email"
              id="contact_email"
              name="contact_email"
              class="border rounded w-full py-2 px-3"
              placeholder="Электронная почта для соискателей"
              required
            />
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

          <div>
            <button
              class="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-full w-full focus:outline-none focus:shadow-outline"
              type="submit"
            >
              Опубликовать вакансию
            </button>
          </div>
        </form>
      </div>
    </div>
  </section>
</template>

<style scoped></style>
