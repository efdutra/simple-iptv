<template>
  <div>
    <h1>IPTV Playlist</h1>
    <input v-model="m3uUrl" placeholder="Enter m3u URL" />
    <button @click="fetchM3U">Load Playlist</button>
    <div v-if="loading" class="loader"></div>
    <div v-if="channels.length">
      <ul>
        <li v-for="channel in channels" :key="channel.url">
          <img :src="channel.logo" alt="channel logo" v-if="channel.logo"/>
          <p>{{ channel.name }}</p>
          <button @click="playChannel(channel.url)">Play</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

interface Channel {
  name: string
  logo?: string
  url: string
}

const m3uUrl = ref('')
const channels = ref<Channel[]>([])
const loading = ref(false)
const router = useRouter()

const fetchM3U = async () => {
  if (!m3uUrl.value) return
  loading.value = true
  channels.value = []
  try {
    const response = await axios.get(m3uUrl.value, { responseType: 'text' })
    parseM3U(response.data)
  } catch (error) {
    console.error("Error loading M3U:", error)
  } finally {
    loading.value = false
  }
}

const parseM3U = (data: string) => {
  const lines = data.split('\n')
  let channel: Partial<Channel> = {}
  lines.forEach((line) => {
    if (line.startsWith('#EXTINF')) {
      const info = line.match(/#EXTINF:.*,(.*)/)
      const name = info ? info[1] : ''
      const logoMatch = line.match(/tvg-logo="(.*?)"/)
      const logo = logoMatch ? logoMatch[1] : undefined
      channel = { name, logo }
    } else if (line && !line.startsWith('#')) {
      channel.url = line
      channels.value.push(channel as Channel)
      channel = {}
    }
  })
}

const playChannel = (url: string) => {
  router.push({ name: 'Player', query: { url } })
}
</script>

<style scoped>
.loader {
  border: 16px solid #f3f3f3;
  border-radius: 50%;
  border-top: 16px solid blue;
  width: 120px;
  height: 120px;
  animation: spin 2s linear infinite;
  margin: 20px auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
