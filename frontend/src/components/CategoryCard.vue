<template>
  <router-link :to="`/courses?category=${category.category}`">
    <div
      class="flex flex-col h-full rounded-md overflow-hidden text-ink-gray-9 border hover:shadow-lg transition-shadow duration-200"
      style="min-height: 150px"
    >
      <div
        class="flex-grow flex items-center justify-center p-4 bg-gradient-to-br from-gray-50 to-gray-100"
      >
        <h2 class="text-xl font-semibold text-center text-gray-800">{{ category.category }}</h2>
      </div>
    </div>
  </router-link>
</template>

<script setup>
defineProps({
  category: {
    type: Object,
    required: true,
  },
});
</script>
