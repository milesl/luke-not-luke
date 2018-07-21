<template>
  <div>
    <section class="section">
      <div class="container">
        <h1 class="title">Luke-Not-Luke</h1>
        <h2 class="subtitle">
          Face FaceRecognition with face-api.js to identify Luke or Not Luke.
        </h2>
      </div>
    </section>
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
