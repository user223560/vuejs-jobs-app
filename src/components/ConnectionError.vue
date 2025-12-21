<script setup>
import { defineProps, defineEmits } from "vue"
import PulseLoader from "vue-spinner/src/PulseLoader.vue"

defineProps({
  isLoading: {
    type: Boolean,
    default: false,
  },
  error: {
    type: Boolean,
    default: false,
  },
  retryCount: {
    type: Number,
    default: 0,
  },
  loadingText: {
    type: String,
    default: "Проверка соединения...",
  },
})

const emit = defineEmits(["retry"])
</script>

<template>
  <div class="container m-auto max-w-2xl py-24">
    <!-- Состояние загрузки -->
    <div v-if="isLoading" class="text-center text-gray-500 py-10">
      <PulseLoader />
      <p class="mt-2">{{ loadingText }}</p>
    </div>

    <!-- Состояние ошибки -->
    <div v-else-if="error" class="text-center py-10 px-4">
      <div
        class="bg-white px-6 py-8 mb-4 shadow-md rounded-md border m-4 md:m-0"
      >
        <i class="bi bi-exclamation-triangle text-red-500 text-4xl mb-4"></i>
        <h2 class="text-xl font-bold text-red-600 mb-2">
          Отсутствует соединение с сервером
        </h2>
        <p class="text-gray-700 mb-4">
          Пожалуйста, попробуйте подключиться позже
        </p>

        <button
          v-if="retryCount <= 3"
          @click="emit('retry')"
          class="mt-4 px-6 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600 transition-colors"
        >
          Попробовать снова
        </button>
        <div v-else class="mt-6">
          <p class="text-gray-600 mb-4">Попробуйте вернуться позже</p>
          <button
            @click="emit('retry')"
            class="inline-block px-6 py-2 bg-gray-500 text-white rounded-lg hover:bg-gray-600 transition-colors mr-4"
          >
            Обновить страницу
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
