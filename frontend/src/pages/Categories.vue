<template>
  <header
    class="sticky flex items-center justify-between top-0 z-10 border-b bg-surface-white px-3 py-2.5 sm:px-5"
  >
    <Breadcrumbs :items="breadcrumbs" />
  </header>
  <div class="container max-w-7xl py-8">
    <div class="flex justify-end mb-4">
      <FormControl
        v-model="searchTerm"
        :placeholder="__('Search by Category')"
        type="text"
        class="w-full sm:w-64"
      />
    </div>
    <div class="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3">
      <CategoryCard
        v-for="category in filteredCategories"
        :key="category.name"
        :category="category"
      />
    </div>
  </div>
</template>

<script setup>
import { createListResource, Breadcrumbs, FormControl } from 'frappe-ui';
import CategoryCard from '@/components/CategoryCard.vue';
import { computed, ref } from 'vue';

const searchTerm = ref('');

const breadcrumbs = computed(() => [
  {
    label: 'Categories',
    route: { name: 'Categories' },
  },
]);

const categories = createListResource({
  doctype: 'LMS Category',
  fields: ['name', 'category'],
  auto: true,
});

const filteredCategories = computed(() => {
  if (!categories.data) {
    return [];
  }
  if (!searchTerm.value) {
    return categories.data;
  }
  return categories.data.filter(category =>
    category.category.toLowerCase().includes(searchTerm.value.toLowerCase())
  );
});
</script>
