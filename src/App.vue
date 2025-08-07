<template>
  <div class="max-w-2xl mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">
      GitHub Repositories ({{ username }})
    </h1>

    <ul>
      <li
        v-for="repo in repos"
        :key="repo.id"
        class="p-3 mb-2 bg-white rounded shadow hover:bg-gray-50 transition"
      >
        <a
          :href="repo.html_url"
          target="_blank"
          class="text-blue-600 font-semibold"
        >
          {{ repo.name }}
        </a>
        <p class="text-gray-500 text-sm">
          {{ repo.description || 'No description' }}
        </p>
      </li>
    </ul>
    <div
      v-if="isLoading"
      class="space-y-2"
    >
      <div
        v-for="n in 3"
        :key="n"
        class="h-12 bg-gray-200 animate-pulse rounded"
      ></div>
    </div>
    <div ref="observerRef" />
    <div
      v-if="isEnd"
      class="text-center text-gray-500 mt-4"
    >
      已加載所有 Repositories
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// 定義需要的欄位
interface Repo {
  id: number
  name: string
  html_url: string
  description: string | null
}
const repos = ref<Repo[]>([])
const page = ref(1)
const perPage = ref(30)

const username = import.meta.env.VITE_GITHUB_USERNAME
const isEnd = ref(false)
const isLoading = ref(false)

const fetchRepos = async () => {
  if (isEnd.value || isLoading.value) return
  isLoading.value = true
  try {
    const res = await axios.get(
      `https://api.github.com/users/${username}/repos`,
      {
        params: {
          page: page.value,
          per_page: perPage.value,
          sort: 'updated'
        }
      }
    )
    const newData = res.data.map((item: Repo) => ({
      id: item.id,
      name: item.name,
      html_url: item.html_url,
      description: item.description
    }))
    repos.value.push(...newData)
    // 如果取回的資料不足數量 則代表是最後一頁
    if (res.data.length < perPage.value) {
      isEnd.value = true
      return
    }
    page.value++
    // 第一次取30筆後 之後10筆10筆取
    if (perPage.value === 30) {
      perPage.value = 10
    }
  } catch (err) {
    console.log(err)
  } finally {
    isLoading.value = false
  }
}

const observerRef = ref()
const myObserver = ref<IntersectionObserver | null>(null)
const options = {
  rootMargin: `0px 0px 50px 0px`,
  threshold: 0
}
onMounted(() => {
  myObserver.value = new IntersectionObserver(entries => {
    if (entries[0].isIntersecting) {
      console.log('isIntersecting')

      fetchRepos()
    }
  }, options)
  myObserver.value.observe(observerRef.value)
})
</script>
