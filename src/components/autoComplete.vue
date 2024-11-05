<script setup>
  import { ref } from 'vue'
  import { watchThrottled } from '@vueuse/core'
  import { ofetch } from 'ofetch'

  const query = ref('')
  const loading = ref(false)
  const results = ref([])
  const error = ref(null)
  const perPage = 10

  watchThrottled(query, async () => {
    loading.value = true

    if (query.value.length < 3) return
  
    const { _data: data, status, error } = await ofetch.raw(`https://api.github.com/search/repositories?q=${query.value}&per_page=${perPage}`)

    if (status === 200) {
      results.value = data.items
    } else {
      error.value = error
    }
    loading.value = false
  }, { throttle: 1000 })

</script>
<template>
  <div class="flex flex-col gap-4">
    <h1 class="text-lg font-medium text-center">Easy autoComplete</h1>
    <label class="relative block">
      <span class="sr-only">Search</span>
      <span class="absolute inset-y-0 left-0 flex items-center pl-2">
        <svg class="h-5 w-5 fill-slate-300" viewBox="0 0 20 20"><!-- ... --></svg>
      </span>
      <input v-model="query" class="placeholder:italic placeholder:text-slate-400 block bg-white w-full border border-slate-300 rounded-md py-2 px-4 shadow-sm focus:outline-none focus:border-sky-500 focus:ring-sky-500 focus:ring-1 sm:text-sm text-gray-400 focus:text-gray-600" placeholder="Search for a repository" type="text" name="search"/>
    </label>
    <div class="flex flex-col">
      <div v-if="loading">Loading...</div>
      <div v-else-if="results.length === 0" class="text-sm">No results found</div>
      <div v-else class="grid grid-row divide-y gap-2 divide-gray-400">
        <div v-for="result in results" :key="result.id" class="flex flex-col py-2">
          <a :href="result.html_url" class="text-blue-500">{{ result.full_name }}</a>
          <p>{{ result.description }}</p>
        </div>
      </div>
    </div>
    <div v-if="error" class="bg-red-600 rounded-md p-2 border-2 border-red-900 text-white">
      {{ error }}
    </div>
  </div>
</template>