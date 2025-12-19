<script setup>
import { RouterLink } from "vue-router"
</script>

<template>
  <section class="bg-green-50">
    <div class="container m-auto py-10 px-6">
      <div class="grid grid-cols-1 md:grid-cols-70/30 w-full gap-6">
        <main>
          <div
            class="bg-white p-6 rounded-lg shadow-md text-center md:text-left"
          >
            <div class="text-gray-500 mb-4">Полная занятость</div>
            <h1 class="text-3xl font-bold mb-4">Senior Front-End Developer</h1>
            <div
              class="text-gray-500 mb-4 flex align-middle justify-center md:justify-start"
            >
              <i
                class="fa-solid fa-location-dot text-lg text-orange-700 mr-2"
              ></i>
              <p class="text-orange-700">Москва</p>
            </div>
          </div>

          <div class="bg-white p-6 rounded-lg shadow-md mt-6">
            <h3 class="text-green-800 text-lg font-bold mb-6">
              Описание вакансии
            </h3>

            <p class="mb-4">
              Ищем талантливого разработчика в наш головной офис в Москве. У
              идеального для нас кандидата отличные знания HTML, CSS,
              JavaScript, а также богатый опыт работы с фреймворками, в
              частности - Vue и Angular.
            </p>

            <h3 class="text-green-800 text-lg font-bold mb-2">Зарплата</h3>

            <p class="mb-4">От 250 000 ₽</p>
          </div>
        </main>
        <aside>
          <div class="bg-white p-6 rounded-lg shadow-md">
            <h3 class="text-xl font-bold mb-6">О работодателе</h3>

            <h2 class="text-2xl">ООО Новотех Солюшнс</h2>

            <p class="my-2">
              Новотех - лидер на рынке компаний, специализирующихся на
              веб-разработке и цифровых решениях. Мы гордимся тем, что выпускаем
              на рынок высококачественные продукты и сервисы для наших клиентов.
            </p>

            <hr class="my-4" />

            <h3 class="text-xl">Электронная почта:</h3>

            <p class="my-2 bg-green-100 p-2 font-bold">
              contact@newteksolutions.com
            </p>

            <h3 class="text-xl">Телефон:</h3>

            <p class="my-2 bg-green-100 p-2 font-bold">555-555-5555</p>
          </div>
          <div class="bg-white p-6 rounded-lg shadow-md mt-6">
            <h3 class="text-xl font-bold mb-6">Управление вакансией</h3>
            <a
              href="add-job.html"
              class="bg-green-500 hover:bg-green-600 text-white text-center font-bold py-2 px-4 rounded-full w-full focus:outline-none focus:shadow-outline mt-4 block"
              >Редактировать</a
            >
            <button
              class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-full w-full focus:outline-none focus:shadow-outline mt-4 block"
            >
              Удалить
            </button>
          </div>
        </aside>
      </div>
    </div>
  </section>
</template>

<style scoped></style>
