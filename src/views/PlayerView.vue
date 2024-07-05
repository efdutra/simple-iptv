<template>
  <div>
    <video ref="videoPlayer" class="video-js vjs-default-skin" controls preload="auto" width="640" height="264"></video>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import videojs from 'video.js'
import 'video.js/dist/video-js.css'

import { useRoute } from 'vue-router'

const route = useRoute()
const videoPlayer = ref<HTMLVideoElement | null>(null)

onMounted(() => {
  if (videoPlayer.value) {
    const player = videojs(videoPlayer.value, {
      controls: true,
      autoplay: true,
      preload: 'auto',
      techOrder: ['html5'],
      sources: [{
        src: route.query.url as string,
        type: 'application/x-mpegURL'
      }]
    })

    player.on('error', function() {
      console.error('Error playing video:', player.error())
    })
  }
})
</script>

<style scoped>
/* Add your styles here */
</style>
