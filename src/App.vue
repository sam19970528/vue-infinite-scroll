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
    <div ref="observerRef" />
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
const perPage = 30

const username = import.meta.env.VITE_GITHUB_USERNAME

const fetchRepos = async () => {
  try {
    const res = await axios.get(
      `https://api.github.com/users/${username}/repos`,
      {
        params: {
          page: page.value,
          per_page: perPage,
          sort: 'updated'
        }
      }
    )
    repos.value = res.data.map((item: Repo) => ({
      id: item.id,
      name: item.name,
      html_url: item.html_url,
      description: item.description
    }))
  } catch (err) {
    console.log(err)
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
