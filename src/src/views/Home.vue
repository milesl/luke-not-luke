<template>
  <div class="home">
    <FaceRecognition></FaceRecognition>
  </div>
</template>

<script>
// @ is an alias to /src
import FaceRecognition from '@/components/FaceRecognition.vue'

export default {
  name: 'home',
  components: {
    FaceRecognition
  }
}
</script>
